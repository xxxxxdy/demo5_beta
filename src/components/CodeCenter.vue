<template>
    <div class="center">
        <div style="font-weight:bold; padding-left: 18px">visual code
            <input type="button" class="name" value="name" @click="showName">
        </div>
        <!-- <input type="button" class="clear" value="clear" @click="clearCanvas"> -->
        
        <!--input type="button" class="export" value="export" @click="exportToCsv"-->
        <!-- <input type="button" class="add" style="font-size:18px" value="+" @click="addVisualCodeList"> -->
        <svg fill="none" stroke="currentColor" stroke-width="1.5" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg" aria-hidden="true" class="add" @click="addVisualCodeList">
            <path stroke-linecap="round" stroke-linejoin="round" d="M12 4.5v15m7.5-7.5h-15"></path>
        </svg>
        <div class="pers-bar">
            <div class="one-bar" v-for="item in visual_list" :key="item" :id="'bar_'+item">
                <MyCodesList  ref="code_window" @delete="deleteVisualCodeList" :index="item" :show_name="show_name"/>
            </div>
        </div>
    </div>
</template>

<style scoped>
.center{
    width: 100%;
    height: 100%;
    border: solid #a1caf1 2px; 
    border-radius: 5px; 
    background: #a1caf1; 
    position: relative;
}
.pers-bar{
    position: relative;
    display: flex;
    width: 100%;
    height: 100%;
}

.one-bar{
    position: relative;
    width: 100%;
    height: 100%;
    margin-left: 2px;
    margin-right: 2px;
    transform: all 0.3s
}

.clear, .name, .export{
    margin-left: 12px;
    margin-bottom: 3px;
}

.add{
    position: absolute;
    right: 5px;
    top: 0px;
    width: 25px;
}

</style>

<script>
import MyCodesList from './MyCodesList.vue';
import bus from '../util/eventBus';
import { interactive_list } from '../util/codeList';
import { color_for_highlight } from '../util/colorMapping';

export default{
    emits:['change'],
    components:{ MyCodesList },
    data(){
        return{
            show_name: false,
            visual_list: [0],
            latest_idx: 0
        }
    },

    methods:{
        showName(){
            this.show_name = !this.show_name;
        },

        exportToCsv(){

        },

        addVisualCodeList(){
            if(this.visual_list.length >= 3){
                alert('The visual codes list exceed its upper limit')
                return
            }
            this.visual_list.push(++this.latest_idx)
            this.$emit('change', this.visual_list.length)
        },

        deleteVisualCodeList(index){
            if(this.visual_list.length<=1){
                alert('This is the last visual code')
                return
            }
      
            for(let i=0; i<this.visual_list.length; i++){
                if(Number(index) === this.visual_list[i]){
                    this.visual_list.splice(i, 1)
                    bus.all.get("updateDataset").splice(i+2, 1)
                    bus.all.get("updateVisualCode").splice(i, 1)
                    // bus.all.get("updateDataOrder").splice(i, 1)
                    bus.all.get("changeRectOpacity").splice(i, 1)
                    bus.all.get("changeLineOpacity").splice(i, 1)
                    bus.all.get("clickForLine").splice(i+1, 1)
                    // bus.all.get("showSimilarityCode").splice(i, 1)
                    // bus.all.get("calculateDistance").splice(i, 1)
                    bus.all.get("alignOrder").splice(i, 1)
                    bus.all.get("synchronizeScroll").splice(i, 1)
                    break
                }
            }
            this.$emit('change', this.visual_list.length)
        },
 
    },
    created(){
        bus.on("clearCanvas", msg=>{
            for(let i=0; i<this.visual_list.length; i++)
                this.$refs.code_window[i].clearHighlight()
            for(let key in color_for_highlight)
                delete color_for_highlight[key]
            interactive_list.splice(0, interactive_list.length)
            bus.emit("clickForLine", -1)
        })
    },

}
</script>