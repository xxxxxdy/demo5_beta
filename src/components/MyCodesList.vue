<template>
    <div style="background:white; border-radius: 5px; margin-bottom: 5px;"
        @mouseenter="rollingForDetail" @mouseleave="rollingCancel">
        &nbsp &nbsp <b>attribute</b> &nbsp
        <select style="padding-left:3px" :class="'seletor'+index" @change="changeDataValue">
            <option v-for="item in data_value_name">{{item}}</option>  
        </select>
        <br>
        <div :style="style_shape" @mouseenter="showAdvanceWindows"></div>
        <div :style="style_color">&nbsp &nbsp persistent &nbsp {{per_val}}</div>
        
        <div class="range" style="position: relative; padding-left: 18px; padding-bottom: 30px;" >
            <input type="range" class="persistent" min="0" :max="coarse_max"
                :step="coarse_step" v-model="per_val" @input="changePersistent">
        </div>
        <input type="button" class="kill" value="X" @click="$emit('delete', index)">
        
        <input type="button" class="reset" value="reset" @click="resetOperation"  :style="style_reset">
        <input type="button" class="filter" value="filter" @click="filterCodeCell">
        <input type="button" class="align" value="primary" @click="alignAsBase">
        <input type="button" class="reorder" value="reorder" @click="reorderVisualCode" @dblclick="showOrderView">   
    </div>

    <div class="sort-div" v-show="sort_hover" style="cursor: pointer;">
        <SortWindows @update="updateSortList"/>
    </div>
    
    <TimeMover ref="timeaxis" :target="index" @time-axis="cutTimeRange"/>

    <div class="visual-code-canvas" :id="'canvas'+index" @scroll="recordScroll" >
        <div class="on-canvas" style="display:inline-block; margin-left: 2px; position: relative;" @mousedown="startSelectFunction">
            <div :class="'code-list'+index" v-for="item in codes_message" >
                <div :style="'height:'+onecode_height+'px'">
                <VisualCode ref="subcode" :target="index" :mapping="item.idx" :draw_message="item.value" 
                    :line_opacity="line_opacity" :rect_opacity="rect_opacity" :show="item.show"
                    :code_name="item.name" :show_name="show_name" :filter="item.filter"/></div>
            </div>
            <div :id="'box'+index" :style="style_box"></div>
        </div>
        <div class="my-scroll-bar" @mousedown="moveFloat">
            <div class="my-scroll-bar-item" v-for="item in codes_message" :key="item" 
            :style="{backgroundColor:getColorByMsg(item), height:myScorllBarItemHeight}"></div>
        </div>
        <div class="my-scroll-bar-float" :style="style_float" @mousedown="clickFloat"></div>
        
    </div> 
    
</template>

<style scoped>
.my-scroll-bar{
    display: inline-block;
    position: absolute;
    right: 0%;
    height: 78%;
    /* margin-left: 4px; */
}

.my-scroll-bar-float{
    width: 15px;
    right: 0%;
    opacity: 0.3;
    position: absolute;
    border: 1px black solid;
    border-radius: 5px;
    background-color: whitesmoke;
    z-index: 50;
}

.my-scroll-bar-float:hover{
    opacity: 0.7;
}

.my-scroll-bar-item{
    width: 16px;
}

.visual-code-canvas{
    /* position: relative;  */
    width: 100%;
    height: 78%;
    /* border-radius: 5px; */
    overflow-y: auto;
    background: white;
}
.kill{
    position: absolute;
    right: 5px;
    top: 5px;
    color: red;
}
input[type=range]{
    position: absolute;
    left: 12%;
    right: 8%
}

.align, .filter, .reset,.reorder{
    margin-left: 12px;
    margin-bottom: 2px;
}

.sort-div{
    position: absolute;
    padding: 2px;
    top: 70px;
    left: 255px;
    background-color: white;
    z-index: 100;
}

</style>

<script>
import { data_lists, visualcode_object, interactive_list, calculateInitPersistent, pers_list, filter_list} from '../util/codeList';
import { color_for_highlight, PERS_COLOR, pers_index, getRandomColor } from '../util/colorMapping';
import { re_sort_order, sort_order, updateOrder, getReSortOrder } from '../util/sortForVisual';
import { CODE_PADDING, CODE_WIDTH, user_parameters } from '../util/parameters';
import { getMarksForDraw, CodeToString } from '../util/drawDataManager'
import VisualCode from './VisualCode.vue';
import bus from "../util/eventBus";
import { data_value_line, data_type_name, data_field, data_value } from '../util/dataManager';
import TimeMover from './TimeMover.vue';
import SortWindows from './SortWindows.vue';

export default{
    props:{
        index:{type:[String, Number], default: 0},
        show_name:{type:[Boolean], default:false}
    },
    components: { VisualCode, TimeMover, SortWindows },
    emits:['delete'],
    data() {
        return {
            sort_hover: false,
            mySrollBarTop:'0px',
            data_value_name:[],
            codes_message:[],
            codes_message_beta: [],
            is_beta: false,
            line_opacity: user_parameters["line_opacity"],
            rect_opacity: user_parameters["rect_opacity"],
            /// these two parameters should be change if the canvas change
            onecode_height: 60, 
            once_render_num: 15,
            first_on_screen: 0,
            coarse_max: 1.2,
            coarse_step:0.05,
            fine_step:0.005,
            per_val: 0,
            base_sort: false,
            sort_index: -1,
            style_float:{top:"0px", height:"0px"},
            float_min:0, // start position
            float_height:0, // end position - start position
            float_num:0, 
            style_color:{color: "red", display:"inline-block"},
            style_shape:{display:"inline-block", width:"12px", height:"12px","margin-left": "12px"},
            style_reset:{color: "grey"},
            filter_hover: false,
            sort_msg: null,
            style_box: {background: "aqua", position:"absolute", zIndex: 100, 
                left: "0px", top: "0px", width: "0px", height: "0px", opacity: 0.5},
            /// le: x axis of box relative to main
            /// rex: ... of div.visual-code-canvas ...
            /// canx: ... of canvas._0_0 ...
            is_box_move: false,
            box_para:{le: 0, tp: 0, ri: 0, bt: 0, rex: 0, rey: 0, rewi: 0, canx:0, canid: -1},
            is_mousemove: false,
            start_select_id: -1,
            upper_id: -1,
            lower_id: -1,
            past_id: -1,
        };
    },
    computed:{
        myScorllBarItemHeight(){
            return (100 / this.codes_message.length) + '' + '%'
        }
    },
    methods: {
        getColorByMsg(code_msg){
            if(code_msg["filter"].length > 0) return "red"
            let i = (1-code_msg["value"].hline.length/this.max_hill_num) * 255;
            return "rgb("+i+","+i+",255)";
            // let i = code_msg["value"].hline.length/this.max_hill_num * 255;
            // return "rgb("+i+",0,"+(255-i)+")"
        },

        updateSortList(msg){
            this.sort_msg = msg;
            this.reorderVisualCode();
        },

        closeOrderView(e){   
            e = e || window.event;
            let tar = e.target;
            let flag = false;
            while(tar.nodeName !== "MAIN"){
                if(tar.className === "sort-div"){
                    flag = true;
                    break;
                }
                tar =  tar.offsetParent;
            }
            if(flag) return;

            this.sort_hover = false;
            document.removeEventListener("mouseup", this.closeOrderView);
        },
        showOrderView(){
            this.sort_hover = true;
            let closeFunction = this.closeOrderView;
            document.addEventListener("mouseup", closeFunction);
        },        

        /// Interactive data (for codes)
        selectFunction(e){
            this.is_mousemove = true;
            e = e || window.event;
            let tar = e.target;
            if(tar.tagName !== "CANVAS" && tar.className !== "codename") return;
            let str = tar.id;
            let canvasid = Number(str.split('_')[1]);
            if(canvasid !== this.index) return;
            let codeid = Number(str.split('_')[2]);

            if(this.start_select_id === -1) {
                this.start_select_id = re_sort_order[this.sort_index][codeid]; 
                this.upper_id = this.start_select_id;
                this.lower_id = this.start_select_id;
                this.past_id = this.start_select_id;
             
                if(interactive_list.indexOf(codeid) === -1){
                    color_for_highlight[codeid] = getRandomColor();
                    interactive_list.push(codeid);
                    bus.emit("clickForLine", codeid);
                    bus.emit("analysisCode", [codeid, this.index, this.codes_message[re_sort_order[this.sort_index][codeid]]['value']]);
                }
                return;
            }
            
            let nows = re_sort_order[this.sort_index][codeid];
            if(nows === this.past_id) return;
            this.past_id = nows;

            if(nows > this.upper_id) {
                for(let i=this.upper_id+1; i<=nows; i++){
                    let id = sort_order[this.sort_index][i];
                    if(interactive_list.indexOf(id) === -1){
                        color_for_highlight[id] = getRandomColor();
                        interactive_list.push(id);
                        bus.emit("clickForLine", id);
                    }
                }
                this.upper_id = nows;
            } 
            else if(nows < this.lower_id) {
                for(let i=nows; i<this.lower_id; i++){
                    let id = sort_order[this.sort_index][i];
                    if(interactive_list.indexOf(id) === -1){
                        color_for_highlight[id] = getRandomColor();
                        interactive_list.push(id);
                        bus.emit("clickForLine", id);
                    }
                }
                this.lower_id = nows;
            }
            else if(nows < this.start_select_id){
                for(let i=this.lower_id; i<nows; i++){
                    let id = sort_order[this.sort_index][i];
                    let idx = interactive_list.indexOf(id);
                    if(idx !== -1){
                        delete(color_for_highlight[id]);
                        interactive_list.splice(idx, 1);
                        bus.emit("clickForLine", id);
                    }
                }
                this.lower_id = nows;
            }
            else if(nows > this.start_select_id){
                for(let i=nows+1; i<=this.upper_id; i++){
                    let id = sort_order[this.sort_index][i];
                    let idx = interactive_list.indexOf(id);
                    if(idx !== -1){
                        delete(color_for_highlight[id]);
                        interactive_list.splice(idx, 1);
                        bus.emit("clickForLine", id);
                    }
                }
                this.upper_id = nows;
            }
            else{
                for(let i=this.lower_id; i<=this.upper_id; i++){
                    let id = sort_order[this.sort_index][i];
                    if(id === codeid){
                        if(interactive_list.indexOf(id) === -1){
                            color_for_highlight[id] = getRandomColor();
                            this.addInteractive(id);
                            bus.emit("clickForLine", id);
                        }
                    }
                    else{
                        let idx = interactive_list.indexOf(id);
                        if(idx !== -1){
                            delete(color_for_highlight[id]);
                            interactive_list.splice(idx, 1);
                            bus.emit("clickForLine", id);
                        }
                    }
                }
                this.upper_id = this.start_select_id;
                this.lower_id = this.start_select_id;
            }
            bus.emit("analysisCode", [codeid, this.index, this.codes_message[nows]['value']]);
        },

        /// Interactive data (for one code)
        addInteractive(msg){
            interactive_list.push(msg);
            bus.emit("clickForLine", msg);
            bus.emit("analysisCode", [msg, this.index, this.codes_message[re_sort_order[this.sort_index][msg]]['value']]);
        },
        removeInteractive(msg, idx=0){
            interactive_list.splice(idx, 1);

            bus.emit("clickForLine", msg);
            if(interactive_list.length > 0){
                let idx = interactive_list[interactive_list.length-1]
                bus.emit("analysisCode", [idx, this.index, this.codes_message[re_sort_order[this.sort_index][idx]]['value']]);
            }  
            else bus.emit("analysisCode", [-1, -1, {}]);
        },
        closeSelectFunction(e){
            if(!this.is_mousemove){
                e = e || window.event;
                let tar = e.target;
                if(tar.tagName !== "CANVAS" && tar.className !== "codename") return;
                let str = tar.id;
                let codeid = Number(str.split('_')[2]);

                let idx = interactive_list.indexOf(codeid);
                if(idx === -1){
                    color_for_highlight[codeid] = getRandomColor();
                    this.addInteractive(codeid);
                }
                else{
                    delete(color_for_highlight[codeid]);
                    this.removeInteractive(codeid, idx);
                }
            }
            let moveFunction = this.selectFunction;
            let upFunction = this.closeSelectFunction;
            document.removeEventListener("mousemove", moveFunction);
            document.removeEventListener("mouseup", upFunction);
        },
        startSelectFunction(e){
            if(this.is_box_move) return;
            this.is_mousemove = false;
            this.start_select_id = -1;
            let moveFunction = this.selectFunction;
            let upFunction = this.closeSelectFunction;
            document.addEventListener("mousemove", moveFunction);
            document.addEventListener("mouseup", upFunction);
        },
        
        reorderVisualCode(){
            updateOrder(this.index, this.data_type, this.sort_msg);
            this.sort_index = this.index;
            let tmp = this.index;
            this.codes_message.sort(function(a, b){
                return re_sort_order[tmp][a.idx] - re_sort_order[tmp][b.idx]
            })
            for(let i =this.first_on_screen, j=0; i<this.codes_message.length && j<this.once_render_num; i++, j++){
                this.codes_message[i].show = true;
            }

            if(this.base_sort === true) 
                bus.emit("alignOrder", [this.sort_index, this.scroll_unit.scrollTop])
        },

        alignAsBase(){
            this.base_sort = true;
            bus.emit("alignOrder", [this.index, this.scroll_unit.scrollTop])
        },

        resetOperation(){
            if(this.style_reset["color"] === "grey") return;
            if(this.is_beta) {
                for(let i=0, j=this.codes_message.length; i<j; i++)
                    this.codes_message[i] = JSON.parse(JSON.stringify(this.codes_message_beta[i]));
                this.is_beta = false;
            }
            this.$refs.timeaxis.resetData();

            // updateOrder(this.index, this.data_type, this.sort_msg);
            if(this.sort_index !== this.index){
                this.sort_index = this.index;
                let tmp = this.sort_index;
                this.codes_message.sort(function(a, b){
                    return re_sort_order[tmp][a.idx] - re_sort_order[tmp][b.idx]
                })
            }
            
            for(let i=0; i<this.codes_message.length; i++){
                let len = this.codes_message[i].filter.length;
                if(len>0){
                    this.codes_message[i].filter.splice(0, len);
                    this.$refs.subcode[i].drawImage()
                }
            }
            filter_list[this.index].splice(0, filter_list[this.index].length);
            
            this.style_reset["color"] = "grey";
        },

        cutTimeRange(msg){
            let new_x = msg[0]+CODE_PADDING, new_x2 = msg[1]+CODE_PADDING;
            if(!this.is_beta) {
                this.codes_message_beta.splice(0, this.codes_message_beta.length);
                for(let i=0, j=this.codes_message.length; i<j; i++)
                    this.codes_message_beta.push(JSON.parse(JSON.stringify(this.codes_message[i]))); 
                this.is_beta = true;
            }
            if(this.is_beta){
                for(let i=0, j=this.codes_message.length; i<j; i++)
                    this.codes_message[i] = JSON.parse(JSON.stringify(this.codes_message_beta[i]));
            } 
            
            let k = CODE_WIDTH / (new_x2-new_x);
            let b = CODE_PADDING - k * new_x;
            // console.log(this.codes_message[0])
            for(let i=0, j=this.codes_message.length; i<j; i++){
                let m = this.codes_message[i].value.hline.length;        
                for(let h=0; h<m; h++){
                    if(this.codes_message[i].value.hline.x2 < 0) continue;
                    if(this.codes_message[i].value.hline.x < CODE_WIDTH) continue;
                    this.codes_message[i].value.hline[h].x = k*this.codes_message[i].value.hline[h].x + b;
                    this.codes_message[i].value.hline[h].x2 = k*this.codes_message[i].value.hline[h].x2 + b;
                    this.codes_message[i].value.rect[h].x = k*this.codes_message[i].value.rect[h].x + b;
                    this.codes_message[i].value.rect[h].width = k*this.codes_message[i].value.rect[h].width;
                    this.codes_message[i].value.vline[h].x = k*this.codes_message[i].value.vline[h].x + b;
                }
            }
            
            this.updateScrollForDraw(this.scroll_unit.scrollTop);

            // for(let i=0, j=this.codes_message.length; i<j; i++){
            //     this.$refs.subcode[i].drawImage();
            // }
            this.style_reset["color"] = "black";
        },

        /// Fine similarity part
        moveBox(e){
            let x = e.offsetX, y = e.offsetY;
            let tar = e.target;
            if(tar.nodeName === "BODY") return;
            if(tar.nodeName === "CANVAS"){
                let str = tar.id;
                this.box_para["canid"] = Number(str.split('_')[2]);
            }
            while(tar.nodeName !== "MAIN"){
                x += tar.offsetLeft, y += tar.offsetTop;
                tar =  tar.offsetParent;
            }
            this.box_para["ri"] = x;
            this.box_para["bt"] = y;

            if(x-this.box_para["rex"]>this.box_para["rewi"]) this.style_box["width"] = String(this.box_para["rewi"]-this.box_para["le"])+"px";
            else{
                let wid = x-this.box_para["le"];
                if(wid > 0){
                    this.style_box["width"] = String(wid)+"px";
                }
                else{
                    this.style_box["left"] = String(x-this.box_para["rex"])+"px";
                    this.style_box["width"] = String(-wid)+"px";
                }
            } 
            
            let heig = y-this.box_para["tp"];
            if(heig>0){
                this.style_box["height"] = String(heig)+"px";
            }
            else{
                this.style_box["top"] = String(y-this.box_para["rey"])+"px";
                this.style_box["height"] = String(-heig)+"px";
            }
        },
        realGetCodeRange(){
            this.style_box["left"]= "0px";
            this.style_box["top"]= "0px";
            this.style_box["width"]= "0px";
            this.style_box["height"]= "0px";
            
            filter_list[this.index].splice(0, filter_list[this.index].length);

            let new_x = this.box_para["le"]-this.box_para["canx"];
            let new_x2 = this.box_para["ri"]-this.box_para["canx"];
            if(new_x>new_x2){let tmp=new_x; new_x=new_x2; new_x2=tmp;}
            if(new_x<0) new_x=0;
            if(new_x2>CODE_WIDTH/2) new_x2=CODE_WIDTH/2;
            if(new_x === 0 && new_x2===CODE_WIDTH/2) return;
            // new_x *=2, new_x2*=2;
            /* According to parameter in CodeToString function, */
            /* the string of a character in string is 480/24 = 20 */
            
            new_x = Math.ceil(new_x/10); // new_x*2/20
            new_x2 = Math.ceil(new_x2/10);
            if(new_x2 === new_x) new_x2++;
            let sstr = this.codes_message[re_sort_order[this.sort_index][this.box_para.canid]]['str'].substring(new_x, new_x2);
            let slen = sstr.length * 20;
            // let strue = [], sfalse = [];
            for(let i=0; i<this.codes_message.length; i++){
                this.codes_message[i].filter.splice(0, this.codes_message[i].filter.length);
                let iidx = -1;
                let flag = false;
                while(true){
                    iidx = this.codes_message[i]['str'].indexOf(sstr, iidx+1);
                    if(iidx === -1) break;
                    flag = true;
                    
                    /// draw function
                    this.codes_message[i].filter.push([iidx*20, slen]);
                }
                if(flag) filter_list[this.index].push(sort_order[this.sort_index][i]);
                // if(flag) this.$refs.subcode[i].drawImage();
                // if(flag) strue.push(this.$refs.subcode[i].getDataIdx())
                // else sfalse.push(this.$refs.subcode[i].getDataIdx())
                this.$refs.subcode[i].drawImage();
            } 

            // this.updateScrollForDraw(this.scroll_unit.scrollTop);
            this.style_reset["color"] = "black";

            // if(this.base_sort === true) 
            //     bus.emit("alignOrder", [this.sort_index, this.scroll_unit.scrollTop])
        },
        getCodeRange(){
            let downFunction = this.changeBoxLeftTopForCell;
            let moveFunction = this.moveBox;
            let upFunction = this.getCodeRange;
            document.removeEventListener("mousedown", downFunction);
            document.removeEventListener("mousemove", moveFunction);
            document.removeEventListener("mouseup", upFunction);

            this.is_box_move = false;
            this.realGetCodeRange();
        },
        changeBoxLeftTopForCell(e){                                          
            let x = e.offsetX, y = e.offsetY;
            let tar = e.target;
            if(tar.nodeName === "BODY") return;
            if(tar.nodeName === "CANVAS"){
                let str = tar.id;
                this.box_para["canid"] = Number(str.split('_')[2]);
            }
            while(tar.nodeName !== "MAIN"){
                x += tar.offsetLeft, y += tar.offsetTop;
                tar =  tar.offsetParent;
            }
            this.box_para["le"] = x;
            this.box_para["tp"] = y;
            this.style_box["left"] = String(x-this.box_para["rex"])+"px";
            this.style_box["top"] = String(y-this.box_para["rey"])+"px";
            this.style_box["width"] = "0px";
            this.style_box["height"] = "0px";
            let moveFunction = this.moveBox;
            let upFunction = this.getCodeRange;
            document.addEventListener("mousemove", moveFunction);
            document.addEventListener("mouseup", upFunction);
        }, 
        filterCodeCell(){
            this.is_box_move = true;
            let downFunction = this.changeBoxLeftTopForCell;
            document.addEventListener("mousedown", downFunction);
        },

        /// This function is called while changing the filter
        updateDrawMessage(pers = 0, init = false){
            data_lists[this.index].splice(0, data_lists[this.index].length);
            for(let i=0, len=visualcode_object[this.data_type].length; i<len; i++){
                data_lists[this.index].push(visualcode_object[this.data_type][i].getDataForOneDraw(pers));
            }
            if(init) updateOrder(this.index, this.data_type);
        },

        /// This function is called while changing the mapping channel
        updateMappingChannel(){         
            this.codes_message.splice(0, this.codes_message.length);

            this.is_beta = false;
            this.codes_message_beta.splice(0, this.codes_message_beta.length);

            this.max_hill_num = -1;

            for(let i=0, len=data_lists[this.index].length; i<len; i++){
                let w1 = visualcode_object[this.data_type][sort_order[this.sort_index][i]].total_width;
                let w2 = visualcode_object[this.data_type][sort_order[this.sort_index][i]].start_width;
                let d_idx = sort_order[this.sort_index][i];
                let draw_value = { idx: d_idx, show: false, sim: 0, filter: [],
                    name: visualcode_object[data_value_line[0]][d_idx].name};
                draw_value['value'] = getMarksForDraw(data_lists[this.index][sort_order[this.sort_index][i]], w1, w2);
                let hill_num = draw_value['value'].hline.length;
                this.max_hill_num = hill_num>this.max_hill_num?hill_num:this.max_hill_num; 
                this.codes_message.push(draw_value);
            }
            for(let i=0; i<this.codes_message.length; i++)
                this.codes_message[i]["str"] = CodeToString(this.codes_message[i].value);
            
            if(typeof(this.scroll_unit) === "undefined")
                this.updateScrollForDraw();
            else this.updateScrollForDraw(this.scroll_unit.scrollTop);
            
            filter_list[this.index].splice(0, filter_list[this.index].length);
            /// For update circle in linechart in real time 
            if(interactive_list.length>0){
                bus.emit("clickForLine", -1);
                
                let idx = interactive_list[interactive_list.length-1]
                bus.emit("analysisCode", [idx, this.index, this.codes_message[re_sort_order[this.sort_index][idx]]['value']]);
            }
        },

        /// This function is called just when initializing for synchronizing interaction
        synchronizeHighlight(){
            for(let i=0; i<interactive_list.length; i++){
                this.$refs.subcode[re_sort_order[this.sort_index][interactive_list[i]]].highlightReceiver();
            }
        },

        /// change persistent
        changeFinePersistent(event){
            event = event || window.event;
            if(event.wheelDelta){
                if(event.wheelDelta > 0)
                    this.per_val += this.fine_step;
                else
                    this.per_val -= this.fine_step;
            }
            else if(event.detail){
                if(event.wheelDelta > 0)
                    this.per_val += this.fine_step;
                else
                    this.per_val -= this.fine_step;
            }
            if(this.per_val < 0) this.per_val = 0;
            this.per_val = parseFloat(this.per_val.toFixed(12));
            this.changePersistent()
        },
        rollingForDetail(){
            window.onmousewheel = document.onmousewheel = this.changeFinePersistent;
            document.addEventListener("DOMMouseScroll", this.changeFinePersistent, false)
        },
        rollingCancel(){
            window.onmousewheel = document.onmousewheel = null;
            document.removeEventListener("DOMMouseScroll", this.changeFinePersistent, false)
        },
        changePersistent(){
            this.per_val = Number(this.per_val)
            pers_list[this.index][1] = this.per_val
            
            this.updateDrawMessage(this.per_val)
            this.updateMappingChannel()
        },

        /// change value attribute
        changeDataValue(){
            for(let i=2, len=data_type_name.length; i<len; i++){
                if(this.type_selector.value === data_type_name[i]){
                    this.data_type = i;
                    break;
                }
            }
            /// or init any other persistent value
            this.per_val = 0;
            this.coarse_max = data_field["max_domain"][this.data_type] - data_field["min_domain"][this.data_type]
            this.coarse_step = this.coarse_max / 50
            this.fine_max = data_field["step"][this.data_type]
            this.fine_step = this.fine_max / 50

            pers_list[this.index][0] = this.data_type;
            pers_list[this.index][1] = this.per_val;

            this.updateDrawMessage(this.per_val)
            this.updateMappingChannel()

            this.style_reset.color = "grey";
        },

        moveFloat(e){
            if(this.float_include>=1) return;
            e = e || window.event;
            let wid = (this.float_height*this.float_include);
            let pos = e.offsetY + e.target.offsetTop - wid/2;
            if(pos<0) pos = 0;
            if(pos>this.float_height-wid) pos = this.float_height - wid;
            this.style_float.top = this.float_min+pos+"px";
            let scroll_float = pos/this.float_height*(this.codes_message.length*(this.onecode_height+2));
            this.scroll_unit.scrollTo(0, scroll_float);
            this.updateScrollForDraw(scroll_float, false);
        },
        clickMoveFilter(e){
            e = e || window.event;
            let tar = e.target;
            if(tar.tagName === "BODY" || tar.tagName==="HTML") return;
            if(tar.className === "one-bar") return;
            let flag = false;
            let move_y = e.offsetY;
            while(tar.nodeName !== "MAIN"){
                if(tar.className === "one-bar"){
                    if(tar.id !== "bar_"+this.index) break;
                    flag = true;
                    break;
                }
                if(tar.className === "on-canvas"){
                    move_y -= this.scroll_unit.scrollTop;
                }
                move_y += tar.offsetTop;
                tar = tar.offsetParent;
            }
            
            if(!flag) return;

            let pos = move_y-this.click_float_y - this.float_min;
            let wid = (this.float_height*this.float_include);
            if(pos<0) pos = 0;
            if(pos>this.float_height-wid) pos = this.float_height - wid;
            
            this.style_float.top = pos+"px";
            let scroll_float = pos/this.float_height*(this.codes_message.length*(this.onecode_height+2));
            this.scroll_unit.scrollTo(0, scroll_float);
            this.updateScrollForDraw(scroll_float, false);

        },
        upFliter(e){
            let moveFunction = this.clickMoveFilter;
            let upFunction = this.upFliter;
            document.removeEventListener("mousemove", moveFunction);
            document.removeEventListener("mouseup", upFunction);
        },
        clickFloat(e){
            if(this.float_include>=1) return;
            e = e || window.e;

            this.click_float_y = e.offsetY;
            let moveFunction = this.clickMoveFilter;
            let upFunction = this.upFliter;
            document.addEventListener("mousemove", moveFunction);
            document.addEventListener("mouseup", upFunction);
        },
        /// This function is used to record which codes should be render
        recordScroll(e){
            // this.mySrollBarTop = e.srcElement.scrollTop + 'px'
            let scroll_y = this.scroll_unit.scrollTop;
            if(this.sort_index !== this.index || this.base_sort === true)
                bus.emit("synchronizeScroll", [this.index, scroll_y])
            this.updateScrollForDraw(scroll_y)
        },

        updateScrollForDraw(scroll_y = 0, flag = true){
            /// This 2 is because the marign 2px in sub div
            this.first_on_screen = Math.floor(scroll_y / (this.onecode_height+2))
            let max_top = this.codes_message.length-this.once_render_num
            if( max_top > 0 && this.first_on_screen > max_top ) 
                this.first_on_screen = max_top;
            for(let i=0, j=0; i<this.codes_message.length && j<=this.once_render_num; i++){
                if(i>=this.first_on_screen){
                    this.codes_message[i].show = true;
                    j++;
                }
            }

            if(flag){
                let pos = scroll_y/(this.codes_message.length*(this.onecode_height+2))*this.float_height;
                this.style_float.top = this.float_min+pos+"px";
            }
        },

        /// This function is used to clear highlight and called in CodeCenter
        clearHighlight(){           
            for(let key in color_for_highlight){
                this.$refs.subcode[re_sort_order[this.sort_index][key]].highlightReceiver();
            }
        }
    },

    watch:{
        show_name(new_value, old_value){
            /// This number is the value of font size
            this.onecode_height += new_value ? 25: -25;
            this.updateScrollForDraw(this.scroll_unit.scrollTop);
        }
    },

    created() {
        bus.on("updateDataset", msg=>{
            this.data_value_name.splice(0, this.data_value_name.length)
            for(let i=0; i<data_value_line.length; i++)
                this.data_value_name.push(data_type_name[data_value_line[i]])
            this.data_type = data_value_line[0]
            this.coarse_max = data_field["max_domain"][this.data_type] - data_field["min_domain"][this.data_type]
            this.coarse_step = this.coarse_max / 50
            this.fine_step = data_field["step"][this.data_type] / 50
            this.$refs.timeaxis.updateData();

            pers_list[this.index][0] = this.data_type;
            pers_list[this.index][1] = this.per_val;

            this.float_include = this.float_num/this.codes_message.length;
            this.style_float.height = String(this.float_include>1?this.float_height: this.float_height*this.float_include)+"px";

            this.style_reset.color = "grey";
        })
        bus.on("updateVisualCode", msg=>{
            this.updateDrawMessage(this.per_val, msg);
            this.updateMappingChannel();
        })
        // bus.on("updateDataOrder", msg=>{
        //     this.sort_msg = msg;
        //     if(this.sort_index !== this.index) return;

        //     updateOrder(this.index, this.data_type, msg);
        //     let tmp = this.sort_index;
        //     this.codes_message.sort(function(a, b){
        //         return re_sort_order[tmp][a.idx] - re_sort_order[tmp][b.idx]
        //     })
        //     for(let i =this.first_on_screen, j=0; i<this.codes_message.length && j<this.once_render_num; i++, j++){
        //         this.codes_message[i].show = true;
        //     }

        //     if(this.base_sort === true) 
        //         bus.emit("alignOrder", [this.sort_index, this.scroll_unit.scrollTop])
        // })
        bus.on("changeRectOpacity", msg=>{  
            this.rect_opacity = msg
        })
        bus.on("changeLineOpacity", msg=>{  
            this.line_opacity = msg
        })

        /// including some code of calculating similarity, may modify later
        bus.on("clickForLine", msg=>{
            if(msg === -1) return;
            this.$refs.subcode[re_sort_order[this.sort_index][msg]].highlightReceiver();

            // let idx = re_sort_order[this.sort_index][msg]
            // for(let i=0; i<this.codes_message.length; i++){
            //     this.codes_message[i]['sim']=getSimilarity(
            //         this.codes_message[i]['str'], this.codes_message[idx]['str'])

            //     // if(i<12)  console.log(this.codes_message[i]['sim'])
            // }
            // console.log("-------------------------------")
        })

        bus.on("alignOrder", msg=>{
            if(msg[0]===-1) this.sort_index = this.index;
            else this.sort_index = msg[0];
            if(this.sort_index!== this.index) this.base_sort=false;
            let tmp = this.sort_index;
            this.codes_message.sort(function(a, b){
                return re_sort_order[tmp][a.idx] - re_sort_order[tmp][b.idx]
            })
            this.codes_message_beta.sort(function(a, b){
                return re_sort_order[tmp][a.idx] - re_sort_order[tmp][b.idx]
            })
            this.scroll_unit.scrollTo(0, msg[1]);
            this.updateScrollForDraw(this.scroll_unit.scrollTop);
            this.style_reset["color"] = "black";
        })

        bus.on("synchronizeScroll", msg=>{
            if(msg[0] === this.index) return;
            if(this.sort_index === this.index && this.base_sort === false) return;
            this.scroll_unit.scrollTo(0, msg[1]);
            this.updateScrollForDraw(msg[1]);
        })

        /// get value type list, default the first one
        for(let i=0; i<data_value_line.length; i++)
            this.data_value_name.push(data_type_name[data_value_line[i]])
        this.data_type = data_value_line[0]
        /// update persistent range
        this.coarse_max = data_field["max_domain"][this.data_type] - data_field["min_domain"][this.data_type]
        this.coarse_step = this.coarse_max / 50
        // this.fine_max = data_field["step"][this.data_type]
        // this.fine_step = this.fine_max / 50
        this.fine_step = data_field["step"][this.data_type] / 50
        
        if(this.show_name)   this.onecode_height += 25;
        this.sort_index = this.index;

        for(let i=0; i<pers_index.length; i++){
            if(pers_index[i]===-1){
                pers_index[i] = this.index;
                this.style_color["color"] = PERS_COLOR[i];
                this.style_shape["background"] = PERS_COLOR[i];
                switch(i){
                    case 0: this.style_shape["border-radius"] = "6px";break;
                    case 2: this.style_shape["transform"] = "rotate(45deg)"; break;
                    default: break;
                }
                break;
            }
        }
        data_lists[this.index] = []
        pers_list[this.index] = [this.data_type, this.per_val, this.style_color["color"] ]
        sort_order[this.index] = []
        re_sort_order[this.index] = []
        filter_list[this.index] = []
        this.updateDrawMessage(this.per_val, true);
        this.updateMappingChannel();
    },
    mounted(){
        this.type_selector = document.querySelector(".seletor"+this.index)
        this.scroll_unit = document.getElementById("canvas"+this.index)

        let node = this.scroll_unit, x=0, y=0;
        while(node.nodeName !== "MAIN"){
            x += node.offsetLeft, y += node.offsetTop;
            node = node.offsetParent;
        }

        this.float_height = this.scroll_unit.clientHeight;
        this.box_para["rex"] = x, this.box_para["rey"] = y;
        this.box_para["rewi"] = this.scroll_unit.offsetWidth;
        node = document.getElementById("_"+this.index+"_0");
        x = 0;
        while(node.nodeName !== "MAIN"){
            x += node.offsetLeft;
            node = node.offsetParent;
        }
        this.box_para["canx"] = x+CODE_PADDING/2;

        this.float_min = this.scroll_unit.offsetTop;
        this.float_height = this.scroll_unit.offsetHeight;
        this.float_num = Math.ceil(this.float_height/(this.onecode_height+2));

        this.style_float.top = this.float_min+"px";
        this.float_include = this.float_num/this.codes_message.length;
        this.style_float.height = String(this.float_include>1?this.float_height: this.float_height*this.float_include)+"px";

        this.synchronizeHighlight()
    },
    /// destroyed() at vue2.0
    unmounted(){
        for(let i=0; i<pers_index.length; i++){
            if(pers_index[i]===this.index){
                pers_index[i] = -1;
                break;
            }
        }

        if(this.base_sort === true) bus.emit("alignOrder", [-1, 0])

        delete data_lists[this.index]
        delete pers_list[this.index]
        delete sort_order[this.index]
        delete re_sort_order[this.index]
        delete filter_list[this.index]
        /// update linechart
        if(interactive_list.length > 0){
            bus.emit("clickForLine", -1)
            bus.emit("analysisCode", [interactive_list[interactive_list.length-1], -1, {}])
        }
    }
}
</script>