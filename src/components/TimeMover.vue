<template>
    <div class="time-axis" style="position: relative; background-color: white;">
        <canvas :id="'time_'+target" :style="style_var" :width="canvas_x" :height="canvas_y"></canvas>
        <div :id="'touter_'+target" class="outer" :style="style_box"  @mousedown="startDrag()">
        </div>
    </div>
</template>

<style scoped>
.outer{
    border-left:4px solid white;
    border-right: 4px solid white;
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
            style_var:{
                "width":SHOW_WIDTH*1.1+"px",
                "height": SHOW_HEIGHT+"px"
            },
            canvas_x: CODE_WIDTH + 2*CODE_PADDING,
            canvas_y: CODE_HEIGHT + 2*CODE_PADDING,
            style_box: {position:"absolute", left: "4px", top: "0px", 
                width: SHOW_WIDTH*1.1+"px", height: "100%"},
            box_para:{le: 0, ri: 0, step: 0},
            dragleft: false,
            move_offset: 0,
        
            nums: data_field.x_distri,
            maxs: data_field.max_distri
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
            let x = Math.round(e.offsetX*2/1.1/this.box_para.step);
            // console.log(this.dragleft, x, this.box_para.le, this.box_para.ri)
            if(this.dragleft) {
                this.box_para.le = x<this.box_para.ri-1? x: this.box_para.ri-1;
                this.box_para.le = this.box_para.le > 0? this.box_para.le : 0;
            }
            
            else {
                this.box_para.ri = x>this.box_para.le+1? x: this.box_para.le+1;
                this.box_para.ri = this.box_para.ri < this.nums.length ? this.box_para.ri : this.nums.length;
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

        startDrag(e){
            e = e|| window.event;
            let x = Math.round(e.offsetX*2/this.box_para.step);
            this.dragleft = x-this.box_para.le<this.box_para.ri-x ? true: false;
            // console.log(this.dragleft, x, this.box_para.le, this.box_para.ri)
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
                this.context.strokeRect(start, top, width, height);
                this.context.fillRect(start, top, width, height);
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