<template>
    <div class="time-axis" style="position: relative; background-color: white;">
        <svg t="1677927383035" class="resize_box_bar left_resize_bar" viewBox="0 0 1024 1024" version="1.1" xmlns="http://www.w3.org/2000/svg" p-id="7260" :style="style_left_resize_bar" @mousedown="startDrag($event, 2)">
            <path d="M511.34975 963.671206c-247.926057 0-448.889579-200.963522-448.889579-448.817329 0-247.926057 200.963522-448.817329 448.889579-448.817329 247.853807 0 448.817329 200.891272 448.817328 448.817329-0.036125 247.853807-200.999647 448.817329-448.817328 448.817329z m0-841.568617c-216.966909 0-392.751288 175.892754-392.751288 392.751288 0 216.894659 175.784379 392.751288 392.751288 392.751288 216.894659 0 392.679038-175.892754 392.679037-392.751288-0.036125-216.894659-175.820504-392.751288-392.679037-392.751288z m127.196218 632.151838c-10.945883 11.018133-28.755521 11.018133-39.665279 0l-214.835533-214.763282c-6.68313-6.75538-8.814506-15.931137-7.225005-24.637268-1.589501-8.706131 0.541875-17.990263 7.225005-24.637268l214.835533-214.835532a28.061921 28.061921 0 0 1 39.665279 0c10.945883 10.945883 10.945883 28.755521 0 39.665279l-199.807522 199.807521 199.807522 199.735271c10.945883 11.018133 10.945883 28.719396 0 39.665279z" fill="#d4237a" p-id="7261"></path>
        </svg>
        <svg class="resize_box_bar right_resize_bar" t="1677929332360" viewBox="0 0 1024 1024" version="1.1" xmlns="http://www.w3.org/2000/svg" p-id="7888" :style="style_right_resize_bar" @mousedown="startDrag($event, 3)">
            <path d="M512 964.465956c-247.853807 0-448.817329-200.963522-448.817329-448.817328 0-247.926057 200.963522-448.889579 448.817329-448.889579 247.926057 0 448.817329 200.963522 448.817329 448.889579 0 247.889932-200.855147 448.817329-448.817329 448.817328z m0-841.568616c-216.894659 0-392.751288 175.784379-392.751288 392.751288 0 216.894659 175.892754 392.679038 392.751288 392.679037 216.894659 0 392.751288-175.784379 392.751288-392.679037 0.036125-216.966909-175.856629-392.751288-392.751288-392.751288z m127.232343 417.424681l-214.763282 214.763282a28.061921 28.061921 0 0 1-39.665279 0 28.061921 28.061921 0 0 1 0-39.665279l199.735271-199.735271-199.735271-199.807522c-10.945883-10.945883-10.945883-28.755521 0-39.665279a28.061921 28.061921 0 0 1 39.665279 0l214.763282 214.763283c6.75538 6.75538 8.814506 16.003387 7.297256 24.745643 1.517251 8.597756-0.541875 17.845763-7.297256 24.601143z" fill="#d4237a" p-id="7889"></path>
        </svg>
        <canvas :id="'time_'+target" :style="style_var" :width="canvas_x" :height="canvas_y"></canvas>
        <div :id="'touter_'+target" class="outer" :style="style_box"  @mousedown="startDrag($event, 0)">
        </div>
    </div>
</template>

<style scoped>

.resize_box_bar{
    position:absolute;
    top: 50%;
    transform:translate(0,-50%);
    width: 12px;
    height: 12;
    z-index: 98;
}
</style>

<script>
import { data_field } from '../util/dataManager';
import { CODE_PADDING, CODE_HEIGHT, CODE_WIDTH, SHOW_WIDTH, SHOW_HEIGHT} from '../util/parameters'

export default{
    emits:['timeAxis'], 

    props:{
        target:{ type: [String, Number], default: 0 }
    },

    data(){
        return{
            style_x_num:SHOW_WIDTH*1.1,
            
            canvas_x: CODE_WIDTH + 2*CODE_PADDING,
            canvas_y: CODE_HEIGHT + 2*CODE_PADDING,
            
            box_para:{le: 0, ri: 0, step: 0},
            dragleft: false,
            move_offset: 0,
        
            nums: data_field.x_distri,
            maxs: data_field.max_distri,
            MouseDownPos: -1,
            leBeforeMove:0,
            riBeforeMove:0,
            changeBoxState:0,
            resize_bar_width:12
        }
    },
    computed:{
        style_var(){
            return {
                "width": this.style_x_num+"px",
                "height": SHOW_HEIGHT+"px"
            }
        },
        style_box(){
            return {position:"absolute", left: "4px", top: "0px", 
                width: this.style_x_num+"px", height: "100%"}
        },
        style_left_resize_bar(){
            return{
                left: (CODE_PADDING + (this.box_para.step * this.box_para.le)) * this._scale_x-this.resize_bar_width+3
            }
        },
        style_right_resize_bar(){
            return{
                left: (CODE_PADDING + (this.box_para.step * this.box_para.ri)) * this._scale_x
            }
        },
        _scale_x(){
            return this.style_x_num / this.canvas_x 
        }
    },
    methods:{

        resetData(){
            this.box_para.le = 0;
            this.box_para.ri = this.nums.length;
            this.drawAxis();
        },

        updateData(){
            this.nums = data_field.x_distri;
            this.maxs = data_field.max_distri;
            this.context.fillStyle = '#ffffff'
            this.context.fillRect(0, 0, this.canvas_x, this.canvas_y)
            this.resetData();
        }, 

        moveBoxSide(e){
            e = e|| window.event;
            let tar = e.target;
            let str = tar.id;
            if(str !=="touter_"+this.target){
                return;
            }
            let nowPos = this.getNowMousePos(e.offsetX)
            if(nowPos < 0){
                nowPos = 0
            }else if(nowPos > this.nums.length){
                nowPos = this.nums.length
            }
            let tmp = nowPos - this.MouseDownPos
            if(this.changeBoxState == 0){
                // move the box
                if(tmp < 0){
                    //left move
                    let new_le = this.leBeforeMove + tmp
                    new_le = new_le < 0 ? 0:new_le
                    this.box_para.ri = this.riBeforeMove + (new_le - this.leBeforeMove)
                    this.box_para.le = new_le
                }else if(tmp > 0){
                    //right move
                    let new_ri = this.riBeforeMove + tmp
                    new_ri = new_ri > this.nums.length ? this.nums.length:new_ri
                    this.box_para.le = this.leBeforeMove + (new_ri - this.riBeforeMove)
                    this.box_para.ri = new_ri
                }
            }else if(this.changeBoxState == 1){
                //set a new box
                //defalut: nowPos is on the right
                if(nowPos < this.MouseDownPos){
                    tmp = this.MouseDownPos
                    this.MouseDownPos = nowPos
                    nowPos = tmp
                }
                if(this.MouseDownPos >= this.nums.length){
                    this.MouseDownPos = this.nums.length-1
                }
                this.box_para.le = this.MouseDownPos
                this.box_para.ri = nowPos + 1
            }else if(this.changeBoxState == 2){
                //left resize
                if(nowPos >= this.box_para.ri){
                    nowPos = this.box_para.ri - 1
                }
                this.box_para.le = nowPos
            }else if(this.changeBoxState == 3){
                //right resize
                if(nowPos <= this.box_para.le){
                    nowPos = this.box_para.le + 1
                }
                this.box_para.ri = nowPos
            }
            this.drawAxis();
        },

        updateBoxSide(e){
            let moveFunction = this.moveBoxSide;
            let upFunction = this.updateBoxSide;
            document.removeEventListener("mousemove", moveFunction);
            document.removeEventListener("mouseup", upFunction);
            this.$emit('timeAxis', [this.box_para.le*this.box_para.step, this.box_para.ri*this.box_para.step]);
        },

        getNowMousePos(offsetX){
            return Math.round(offsetX/this._scale_x/this.box_para.step)
        },
        startDrag(e, flag){
            e = e|| window.event;
            this.MouseDownPos = this.getNowMousePos(e.offsetX)
            if(flag == 0){
                if(this.MouseDownPos < 0){
                    this.MouseDownPos = 0
                }else if(this.MouseDownPos > this.nums.length){
                    this.MouseDownPos = this.nums.length
                }
                if(this.MouseDownPos >= this.box_para.le && this.MouseDownPos < this.box_para.ri){
                    // move the box
                    this.riBeforeMove = this.box_para.ri
                    this.leBeforeMove = this.box_para.le
                    this.changeBoxState = 0
                }else{
                    // set a new box
                    this.changeBoxState = 1
                }
            }else{
                // change size
                this.changeBoxState = flag
            }
            //console.log(flag,this.changeBoxState,e.offsetX,this.MouseDownPos,this.box_para.le, this.box_para.ri)
            //console.log(e.offsetX,this.box_para.step,this.dragleft, x, this.box_para.le, this.box_para.ri)
            let moveFunction = this.moveBoxSide;
            let upFunction = this.updateBoxSide;
            document.addEventListener("mousemove", moveFunction);
            document.addEventListener("mouseup", upFunction);
        },
        initCanvas(){
            this.canvas = document.querySelector("#time_"+this.target)
            this.context = this.canvas.getContext('2d')
        },

        drawAxis(){     
            let start = CODE_PADDING;
            let bottom = CODE_HEIGHT;
            let width = this.box_para.step;
            for(let i=0, j=this.nums.length; i<j; i++){
                if(i>=this.box_para.le && i<this.box_para.ri)  this.context.fillStyle = 'aqua';
                else  this.context.fillStyle = 'grey';
                let height = bottom*this.nums[i]/this.maxs;
                let top = bottom - height + CODE_PADDING;
                this.context.fillRect(start, top, width, height);
                this.context.strokeRect(start, top, width, height);
                start += width;
            }
        }
    },

    mounted(){
        this.initCanvas();
        this.box_para.ri = this.nums.length;
        this.box_para.step = CODE_WIDTH/this.nums.length;
        this.drawAxis();
    }
}
</script>