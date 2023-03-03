<template>
    <div class="encode">
        <div class="fold-allow" id="encode-allow" @click="makeEncodeFold"></div>
        <div style="display:inline-block;font-weight: bold " @click="makeEncodeFold"> encode </div>
        <input type="button" id="reset" value="reset" @click="resetEncoding">
        <input type="button" id="update" value="update" @click="startUpdateVisualCode">
        
        <div class="rect" v-show="encode_fold">
            <div style="font-weight: bold; ">rectangle</div>
            width:
            <select class="rect-width" @change="setRectWidth">
                <option v-for="item in encode_options">{{item}}</option>
            </select>
            <br>
            height:
            <select class="rect-height" @change="setRectHeight">
                <option v-for="item in encode_options">{{item}}</option>
            </select>
            <br>
            position:
            <select class="rect-pos" @change="setRectPos">
                <option v-for="item in encode_options">{{item}}</option>
            </select>
            <br>
            color:
            <select class="rect-color" @change="setRectColor">
                <option v-for="item in encode_options">{{item}}</option>
            </select>
        </div>

        <div class="line" v-show="encode_fold">
            <div style="font-weight: bold">horizontal line</div>
            width:
            <select class="line1-width" @change="setLine1Width">
                <option v-for="item in encode_options">{{item}}</option>
            </select>
            <br>
            y-position:
            <select class="line1-height" @change="setLine1Pos">
                <option v-for="item in encode_options">{{item}}</option>
            </select>
            <br>
            stroke-width:
            <select class="line1-stroke" @change="setLine1StrokeWidth">
                <option v-for="item in encode_options">{{item}}</option>
            </select>
            <br>
            stroke-dash:
            <select class="line1-strokeDash" @change="setLine1StrokeDash">
                <option v-for="item in encode_options">{{item}}</option>
            </select>
            <br>
            color:
            <select class="line1-color" @change="setLine1Color">
                <option v-for="item in encode_options">{{item}}</option>
            </select>
        </div>

        <div class="line" v-show="encode_fold">
            <div style="font-weight: bold">vertical line</div>
            height:
            <select class="line2-height" @change="setLine2Height">
                <option v-for="item in encode_options">{{item}}</option>
            </select>
            <br>
            stroke-width:
            <select class="line2-stroke" @change="setLine2Stroke">
                <option v-for="item in encode_options">{{item}}</option>
            </select>
            <br>
            color:
            <select class="line2-color" @change="setLine2Color">
                <option v-for="item in encode_options">{{item}}</option>
            </select>

        </div>
    </div>

    <!-- <div class="sort">
        <div class="fold-allow" id="sort-allow"  @click="makeSortFold"></div>
        <div style="display:inline-block;font-weight: bold"  @click="makeSortFold"> sort rule &nbsp &nbsp </div>
        <MySwitch v-model="on_line_order" checked-info="on-line" unchecked-info="off-line"/>
        
        <div style="background: white; border-radius: 5px; width: 100%;">
            <div v-show="sort_fold" style="font-weight: bold"> &nbsp drag for priority</div>
            <draggable v-model="sort_list" v-show="sort_fold" item-key="index" style="padding-left:5px" 
              @change="updateSortRule">
                <template #item="{element}">
                    <li :key="element.index">
                        <div style="display:inline-block" position="relative" >
                            {{ element.name }}
                            <MySwitch class="ban" v-model="element.ban" checked-info="ban" 
                              @click="updateSortRule"/>
                            <MySwitch class="reverse" v-model="element.reverse" checked-info="re" 
                              @click="updateSortRule" :disabled="element.ban"/>
                        </div>
                    </li>
                </template>
            </draggable>
        </div>
    </div> -->

    <div class="param">
        <div style="font-weight: bold"> &nbsp &nbsp parameters </div>
        <div style="background: white; padding-left: 6px;">
            rect-opacity: 
            <input type="range" class="opacity" id="rect" min="0" max="1"
                step="0.01" v-model="rect_opacity" @input="changeRectOpacity">
            <br>
            line-opacity: 
            <input type="range" class="opacity" id="line" min="0" max="1"
                step="0.01" v-model="line_opacity" @input="changeLineOpacity">

            <br>
            global time:
            <!--input type="checkbox" class="globalx" @change="changeGlobalX" checked-->
            <MySwitch @click="changeGlobalX"/>
            <br>
            global value:
            <!--input type="checkbox" class="globaly" @change="changeGlobalY" checked-->
            <MySwitch @click="changeGlobalY"/>
        </div>
    </div>
    
</template>

<style scoped>
.encode, .param, .sort{
    position: relative;
    border: solid #a1caf1 2px;
    border-radius: 5px;
    margin-top: 8px;
    background: #a1caf1;
}

.fold-allow{
    position: relative;
    width: 20px;
    height: 20px;
    display: inline-block;
    transition: all 0.3s;
}

.fold-allow:before{
    content: '';
    width: 6px;
    border: 1px solid rgb(80, 79, 79);
    position: absolute;
    transform: rotateZ(45deg);
    top: 14px;
    left: 2px;
}
.fold-allow:after{
    content: '';
    width: 6px;
    border: 1px solid rgb(80, 79, 79);
    position: absolute;
    transform: rotateZ(-45deg);
    top: 14px;
    left: 8px;
}

/* .ban{
    position: absolute;
    right: 2px;
}

.reverse{
    position: absolute;
    right: 54px;
} */

#reset{
    position: absolute;
    right: 70px;
    top: 4px;
    color: red;
}

#update{
    position: absolute;
    right: 6px;
    top: 4px;
}

.rect, .line{
    position: relative;
    margin-top: 5px;
    padding-left: 10px;
    background: white;
    border-radius: 5px;
}

select{
    position: absolute;
    width: 45%;
    right: 1px;
}

#pers_val{
    color: red;
    display: inline;
}

.opacity{
    position: absolute;
    right: 5px;
    width: 50%;
}

</style>

<script>
import { mapping_relationship, user_parameters } from '../util/parameters'
import bus from '../util/eventBus'
import Draggable from 'vuedraggable'
import MySwitch from './MySwitch.vue'
import { sort_type, updateOrder } from '../util/sortForVisual'
import { initAllTheHillsData, visualcode_object } from '../util/codeList'

export default{
    components:{ Draggable, MySwitch },
    data(){
        return{
            encode_options:['none','width','top_time',
                'max','min','persistent','area'],
            encode_fold: true,
            // sort_fold: true,
            global_x: true,
            global_y: true,
            rect_opacity: 0.8,
            line_opacity: 1,
            // on_line_order: false,
            // sort_list:sort_type.map((name, index) =>{
            //     return { name, index, ban:false, reverse:false};
            // })
        }
    },

    methods:{
        // updateSortRule(){
        //     bus.emit('updateDataOrder', this.sort_list)
        // },

        /// fold toorbar
        makeEncodeFold(){
            this.encode_fold = !this.encode_fold;
            if(this.encode_fold){
                document.querySelector('#encode-allow').style.transform = "rotateZ(0deg)";
            }
            else{
                document.querySelector('#encode-allow').style.transform = "rotateZ(-90deg)";
            }
        },

        // makeSortFold(){
        //     this.sort_fold = !this.sort_fold;
        //     if(this.sort_fold){
        //         document.querySelector('#sort-allow').style.transform = "rotateZ(0deg)";
        //     }
        //     else{
        //         document.querySelector('#sort-allow').style.transform = "rotateZ(-90deg)";
        //     }
        // },

        resetEncoding(flag = true){
            this.rect_width.value = this.encode_options[6]
            mapping_relationship["rect"]["width"] = this.encode_options[6]
            this.rect_height.value = this.encode_options[0]
            mapping_relationship["rect"]["height"] = this.encode_options[0]
            this.rect_pos.value = this.encode_options[2]
            mapping_relationship["rect"]["x"] = this.encode_options[2]
            this.rect_color.value = this.encode_options[0]
            mapping_relationship["rect"]["fill"] = this.encode_options[0]
            this.line1_width.value = this.encode_options[1]
            mapping_relationship["hline"]["width"] = this.encode_options[1]
            this.line1_y.value = this.line1_y.options[0].value;
            mapping_relationship["hline"]["y"] = this.encode_options[0]
            this.line1_strokeWidth.value = this.encode_options[0]
            mapping_relationship["hline"]["strokeWidth"] = this.encode_options[0]
            this.line1_strokeDash.value = this.encode_options[0]
            mapping_relationship["hline"]["strokeDash"] = this.encode_options[0]
            this.line1_color.value = this.encode_options[0]
            mapping_relationship["hline"]["stroke"] = this.encode_options[0]
            this.line2_height.value = this.encode_options[0]
            mapping_relationship["vline"]["height"] = this.encode_options[0]
            this.line2_strokeWidth.value = this.encode_options[0]
            mapping_relationship["vline"]["strokeWidth"] = this.encode_options[0]
            this.line2_color.value = this.encode_options[0]
            mapping_relationship["vline"]["stroke"] = this.encode_options[0]

            if(flag) this.startUpdateVisualCode();
        },
        startUpdateVisualCode(){
            bus.emit("updateVisualCode", false);
            // console.log(mapping_relationship);
        },

        /// a series of assignment funtions
        setRectWidth(){
            mapping_relationship["rect"]["width"] = this.rect_width.value;
        },
        setRectHeight(){
            mapping_relationship["rect"]["height"] = this.rect_height.value;
        },
        setRectPos(){
            mapping_relationship["rect"]["x"] = this.rect_pos.value;
        },
        setRectColor(){
            mapping_relationship["rect"]["fill"] = this.rect_color.value;
        },
        setLine1Width(){
            mapping_relationship["hline"]["width"] = this.line1_width.value;
        },
        setLine1Pos(){
            mapping_relationship["hline"]["y"] = this.line1_y.value;
        },
        setLine1StrokeWidth(){
            mapping_relationship["hline"]["strokeWidth"] = this.line1_strokeWidth.value;
        },
        setLine1StrokeDash(){
            mapping_relationship["hline"]["strokeDash"] = this.line1_strokeDash.value;
        },
        setLine1Color(){
            mapping_relationship["hline"]["stroke"] = this.line1_color.value;
        },
        setLine2Height(){
            mapping_relationship["vline"]["height"] = this.line2_height.value;
        },
        setLine2Stroke(){
            mapping_relationship["vline"]["strokeWidth"] = this.line2_strokeWidth.value;
        },
        setLine2Color(){
            mapping_relationship["vline"]["stroke"] = this.line2_color.value;
        },

        changeRectOpacity(){
            user_parameters["rect_opacity"] = this.rect_opacity;
            bus.emit("changeRectOpacity", this.rect_opacity);
        },
        changeLineOpacity(){
            user_parameters["line_opacity"] = this.line_opacity;
            bus.emit("changeLineOpacity", this.line_opacity);
        },
        changeGlobalX(){
            user_parameters["global_time"] = !user_parameters["global_time"];
            for(let key in visualcode_object){
                for(let i=0, len=visualcode_object[key].length; i<len; i++)
                    visualcode_object[key][i].updateDataXDomain();
            }
            bus.emit("updateVisualCode", false);
        },
        changeGlobalY(){
            user_parameters["global_value"] = !user_parameters["global_value"];
            for(let key in visualcode_object){
                for(let i=0, len=visualcode_object[key].length; i<len; i++)
                    visualcode_object[key][i].updateDataYDomain(key);
            }
            bus.emit("updateVisualCode", false);
        }
    },

    created(){
        bus.on("updateDataset", msg=>{
            /// parameters will not reset while update dataset
            this.resetEncoding(false);
            this.sort_list = sort_type.map((name, index) =>{
                return { name, index, ban:false, reverse:false};
            })
            /// This function only call here to deal the origin data
            initAllTheHillsData()
            console.log("finish coding.");
            bus.emit("updateVisualCode", true);
        })
        initAllTheHillsData()
    },

    mounted(){
        this.rect_width = document.querySelector(".rect-width");
        this.rect_height = document.querySelector(".rect-height");
        this.rect_pos = document.querySelector(".rect-pos");
        this.rect_color = document.querySelector(".rect-color");
        this.line1_width = document.querySelector(".line1-width");
        this.line1_y = document.querySelector(".line1-height");
        this.line1_strokeWidth = document.querySelector(".line1-stroke");
        this.line1_strokeDash = document.querySelector(".line1-strokeDash");
        this.line1_color = document.querySelector(".line1-color");
        this.line2_height = document.querySelector(".line2-height");
        this.line2_strokeWidth = document.querySelector(".line2-stroke");
        this.line2_color = document.querySelector(".line2-color");

        this.resetEncoding(false);
    }
}
</script>