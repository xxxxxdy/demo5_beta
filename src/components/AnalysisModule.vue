<template>
    <div style="border: solid #a1caf1; border-radius: 5px;">
        <div style="background: #a1caf1;">
            <input id="treeinput" class="treeinput" name="window" type="radio" checked>
            <label for="treeinput" @click="openTreeWindow">code tree</label>
            <!-- <input id="selectinput" class="selectinput" name="window" type="radio">
            <label for="selectinput" @click="openSelectWindow">advanced</label> -->
        </div>
    
        <div class="treewindow" :style="style_vis1">
            <div style="font-weight:bold; padding-left: 25px;padding-top: 18px;font-size: 20px;">{{name}}</div>
            <div style="padding-left:25px">
                <canvas id="analysis" :width="(canvas_x+2*padding)*3" 
                :height="(canvas_y+padding+text)" :style="style_var"></canvas>
            </div>
        </div>
        <!-- <div class="selectwindow" :style="style_vis2">
            <div style="font-weight:bold; padding-left: 25px;padding-top: 18px;font-size: 20px;">{{name}}<br>
                <VisualCode :draw_message="draw_value" :show="true"/>
            </div>
            <canvas style="position:absolute; right: 80px; top: 10px; width: 360px; height: 300px;"
              width="1080" height="900" id="cluster" ></canvas>
            <fieldset style="margin:10px; width: 40%; position: relative;">
                <div style="font-weight:bold;">similarity</div>
                <MySwitch class="locked" v-model="islocked" checked-info="locked"/>
                <input type="text" v-model="sim_value" @change="showSimilarity"/>
                <input type="range" v-model="sim_value" max="1" min="0" step="0.001" @input="showSimilarity"/>
            </fieldset>
            <fieldset style="margin:10px; width:40%; position: relative;">
                <div style="font-weight:bold;">clustering</div>
                <div style="margin-left: 20px; color:red">{{hint}}</div>
                <li> epsilon 
                    <br>
                    <input style="margin-left:20px" type="text" v-model="eps" @change="codeClustering"/>
                    <input type="range" v-model="eps" max="1" min="0" step="0.001" @change="codeClustering"/>
                </li>
                <li>minPoints
                    <br>
                    <input style="margin-left:20px" type="text" v-model="minpts" @change="codeClustering"/>
                    <input type="range" v-model="minpts" max="100" min="0" step="1" @change="codeClustering"/>
                </li>
                <button style="position:absolute; right: 100px; top: 10px; border: 1px solid black;"
                  @mousedown="changeHint" @click="updateDistance">update distance</button>
            </fieldset>
        </div> -->
    </div>
</template>

<style scoped>
label{
    margin-left: 18px;
    font-weight:bold;
    font-size:18px;
}

.treeinput:checked +label, .selectinput:checked+label{
    background: #a1caf1;
    opacity: 100%;
}
.treeinput +label, .selectinput+label{
    background: #b0e2ff; 
    opacity: 50%;  
}

input[type=radio]{
	visibility: hidden;
}

input[type=text]{
    width:80px;
}

input[type=range]{
    width: 200px;
    margin-left: 20px;
}

.locked{
    position: absolute;
    right: 150px;
    top: 5px;
}

button:active{
    background-color: #b0e2ff;
}

</style>

<script>
import { pers_list, visualcode_object } from '../util/codeList';
// import { pers_list, visualcode_object, getCodePoints, code_points } from '../util/codeList';
import { data_field, data_type_name, data_value_line, getDataTime } from '../util/dataManager';
import bus from "../util/eventBus";
// import { data_dis, dbscan, cluster_res } from '../util/similarityCalculate';
// import { CLUSTER_COLOR_SET } from '../util/colorMapping'
import MySwitch from './MySwitch.vue';
import VisualCode from './VisualCode.vue';

export default{
    data() {
        return {
            islocked: false,
            sim_value: 0,
            eps: 0,
            minpts: 1,
            data_points: [],
            name: " ",
            hint: " ",
            style_var: {
                "width": "870px",
                "height": "300px"
            },
            draw_value: {"hline":[], "vline":[], "rect":[]},
            style_vis1: { "display": "block", "position": "relative"},
            // style_vis2: { "display": "none", "position": "relative" },
            canvas_x: 810,
            canvas_y: 810,
            padding: 90,
            text: 20,
            list_idx: -1,
            analysis_idx: -1,
            max_persistent: 10
        };
    },
    methods: {
        // /// the function for changing the windows
        // openTreeWindow() {
        //     this.style_vis1["display"] = "block";
        //     this.style_vis2["display"] = "none";
        // },
        // openSelectWindow() {
        //     this.style_vis2["display"] = "block";
        //     this.style_vis1["display"] = "none";
        // },
        // /// the funtion for similarity
        // getSelectInfo( draw_data = null) {
        //     if(this.analysis_idx < 0 || this.list_idx < 0) this.draw_value= {"hline":[], "vline":[], "rect":[]};
        //     else this.draw_value = draw_data;
        //     this.cluster_max = getCodePoints(this.list_idx);
        //     this.drawClusterPoint();
        // },
        // drawClusterPoint(){
        //     this.c_context.fillStyle = "#ffffff";
        //     this.c_context.globalAlpha = 1;
        //     this.c_context.fillRect(0, 0, 1080, 900);
        //     if (this.analysis_idx < 0 || this.list_idx < 0)
        //         return;
        //     let draw_x, draw_y,unit_x = 1000/this.cluster_max[0], unit_y = 870/this.cluster_max[1];
        //     this.c_context.globalAlpha = 0.5;
        //     let flag = cluster_res.length === code_points.length;
            
        //     for(let i=0, j=code_points.length; i<j; i++){
        //         draw_x = code_points[i][0] * unit_x + 40;
        //         draw_y = code_points[i][1] * unit_y + 15;
        //         if(flag && cluster_res[i]>=0 && cluster_res[i]<11) 
        //             this.c_context.fillStyle = CLUSTER_COLOR_SET[cluster_res[i]];
        //         else  this.c_context.fillStyle = '#000000';
        //         this.c_context.beginPath();
        //         this.c_context.arc(draw_x, draw_y, 15, 0, 2 * Math.PI);
        //         this.c_context.closePath();
        //         this.c_context.fill();
        //     }
        // },
        // changeHint(){
        //     this.hint = "waiting for calculating distance ...";
        // },
        // codeClustering(){
        //     if(data_dis.length !== code_points.length){
        //         this.hint = "please update distance first!";
        //         return;
        //     }
        //     dbscan(data_dis, this.eps, this.minpts);
        //     this.drawClusterPoint();
        // },
        // showSimilarity(){
        //     if(this.analysis_idx < 0) return;
        //     bus.emit("showSimilarityCode", Number(this.sim_value));
        // },
        // updateDistance(){
        //     bus.emit("calculateDistance", this.list_idx);
        //     this.hint = " ";
        // },
        /// the function for drawing code tree
        getMaxMinX(data_type) {
            this.x_start = visualcode_object[data_type][this.analysis_idx].getStartTime();
            this.x_draw_width = this.canvas_x / (visualcode_object[data_type][this.analysis_idx].getEndTime() - this.x_start);
            // this.x_end = visualcode_object[this.data_type][this.analysis_idx].getEndTime()
            // this.x_width = this.x_end - this.x_start
        },
        drawAnalysisTree(data_type, pos = 0) {
            let x_offset = pos * (this.canvas_x + 2 * this.padding) + this.padding;
            // let threshold = (visualcode_object[this.data_type][this.analysis_idx].max_value- 
            //    visualcode_object[this.data_type][this.analysis_idx].min_value)/2
            this.getMaxMinX(data_type);
            /// this variable should not be in class but I am lazy to modify it
            this.y_draw_width = -1;
            let hills_list = visualcode_object[data_type][this.analysis_idx].hill_list;
            this.context.lineWidth = 4;
            for (let i = hills_list.length - 1; i >= 0; i--) {
                // if(hills_list[i].start_persistent > threshold) continue
                if (this.y_draw_width === -1) {
                    this.y_draw_width = this.canvas_y / hills_list[i].end_persistent;
                }
                let line_y = hills_list[i].start_persistent * this.y_draw_width, line_y2 = hills_list[i].end_persistent * this.y_draw_width;
                if (line_y2 - line_y < 40)
                    continue;
                let le_x = getDataTime(hills_list[i].left_idx), ri_x = getDataTime(hills_list[i].right_idx);
                let line_x = (le_x - this.x_start) * this.x_draw_width + x_offset, line_x2 = (ri_x - this.x_start) * this.x_draw_width + x_offset;
                /// draw h line
                let rect_width = (hills_list[i].value_sum / (ri_x - le_x) - hills_list[i].min_value) /
                    (hills_list[i].max_value - hills_list[i].min_value) * (line_x2 - line_x);
                let rect_x = line_x + (line_x2 - line_x - rect_width) *
                    (getDataTime(hills_list[i].top_idx) - le_x) / (ri_x - le_x);
                // console.log(line_x, line_x2, line_y, line_y2, rect_width, rect_x)
                // this.context.strokeStyle = 'rgba(0, 0, 0, 1)'
                // this.context.lineWidth = 4
                this.context.setLineDash([20, 20]);
                this.context.beginPath();
                this.context.moveTo(line_x, (line_y2 + line_y) / 2);
                this.context.lineTo(line_x2, (line_y2 + line_y) / 2);
                this.context.stroke();
                this.context.setLineDash([]);
                this.context.beginPath();
                this.context.moveTo(line_x2, line_y2);
                this.context.lineTo(line_x2, line_y);
                this.context.stroke();
                // this.context.fillStyle = 'rgba(0, 0, 0, 1)'
                this.context.fillRect(rect_x, line_y + 5, rect_width, line_y2 - line_y - 5);
            }
            this.context.font = "54px Arial";
            this.context.fillText(data_type_name[data_type], x_offset, this.canvas_y + this.padding);
        },
        drawPersistentLine(type, pers, color, pos) {
            let hills = visualcode_object[type][this.analysis_idx].hill_list;
            let total_pers = hills[hills.length - 1].end_persistent;
            let y_pos = this.canvas_y / total_pers * pers;
            if (y_pos < 2)
                y_pos = 2;
            if (y_pos > this.canvas_y)
                y_pos = this.canvas_y;
            let x_start = pos * (this.canvas_x + 2 * this.padding) + this.padding / 2, x_end = x_start + this.canvas_x + this.padding;
            this.context.beginPath();
            this.context.moveTo(x_start, y_pos);
            this.context.lineTo(x_end, y_pos);
            // this.context.strokeStyle = color
            this.context.stroke();
            // this.context.strokeStyle = 'rgba(0, 0, 0, 1)'
            let y_sign = y_pos;
            if (y_sign < 20)
                y_sign = 20;
            else if (y_sign > this.canvas_y - 20)
                y_sign = this.canvas_y - 20;
            switch (color) {
                case "red":
                    this.context.beginPath();
                    this.context.arc(x_end + 25, y_sign, 20, 0, 2 * Math.PI);
                    this.context.closePath();
                    this.context.fill();
                    break;
                case "green":
                    this.context.fillRect(x_end + 9, y_sign - 16, 32, 32);
                    this.context.closePath();
                    this.context.fill();
                    break;
                case "blue":
                    this.context.moveTo(x_end + 5, y_sign);
                    this.context.lineTo(x_end + 25, y_sign - 20);
                    this.context.lineTo(x_end + 45, y_sign);
                    this.context.lineTo(x_end + 25, y_sign + 20);
                    this.context.closePath();
                    this.context.fill();
                    break;
                default:
                    break;
            }
        },
        drawAnalysisTrees() {
            this.context.fillStyle = "#ffffff";
            this.context.fillRect(0, 0, (this.canvas_x + 2 * this.padding) * 3, this.canvas_y + 2 * this.padding);
            this.context.fillStyle = "rgb(0, 0, 0)";
            if (this.analysis_idx < 0)
                return;
            let tree_list = [];
            for (let key in pers_list) {
                if (tree_list.indexOf(pers_list[key][0]) === -1) {
                    this.drawAnalysisTree(pers_list[key][0], tree_list.length);
                    tree_list.push(pers_list[key][0]);
                }
                this.drawPersistentLine(pers_list[key][0], pers_list[key][1], pers_list[key][2], tree_list.indexOf(pers_list[key][0]));
            }
        }
    },
    created() {
        bus.on("analysisCode", msg => {
            this.analysis_idx = msg[0];
            if (msg[0] !== -1)
                this.name = visualcode_object[data_value_line[0]][msg[0]].name;
            else
                this.name = " ";
            
            this.list_idx = msg[1];
            this.drawAnalysisTrees();
            // this.getSelectInfo(msg[2]);
        });

        bus.on("updateDataset", msg=>{
            this.list_idx= -1;
            this.analysis_idx= -1;
            this.name = " ";
            this.drawAnalysisTrees();
            // this.getSelectInfo();
        })

    },
    mounted() {
        this.canvas = document.querySelector("#analysis");
        this.context = this.canvas.getContext("2d");
        // this.c_canvas = document.querySelector("#cluster");
        // this.c_context = this.c_canvas.getContext("2d");
    },
    components: { VisualCode, MySwitch }
}
</script>