<template>
    <div style="background: white; border-radius: 5px; width:220px; border: solid #a1caf1 2px;">
        <div style="font-weight: bold"> &nbsp drag for priority</div>
        <draggable v-model="sort_list" item-key="index" style="padding-left:5px" @change="$emit('update', sort_list)">
            <template #item="{element}">
                <li :key="element.index">
                    <div style="display:inline-block" position="relative" >
                        {{ element.name }}
                        <MySwitch class="ban" v-model="element.on" unchecked-info="off" checked-info="on"
                            @click="$emit('update', sort_list)"/>
                        <MySwitch class="reverse" v-model="element.reverse" checked-info="re" unchecked-info="re"
                            @click="$emit('update', sort_list)" :disabled="!element.on"/>
                    </div>
                </li>
            </template>
        </draggable>
    </div>
</template>

<style scoped>
.ban{
    position: absolute;
    right: 55px;
}

.reverse{
    position: absolute;
    right:10px;
}

li{
    list-style: none;
}

li:hover{
    font-weight: bold;
}
</style>

<script>
import { sort_type, updateOrder } from '../util/sortForVisual'
import Draggable from 'vuedraggable';
import MySwitch from './MySwitch.vue';

export default{
    components: { MySwitch, Draggable },
    emits:["update"],
    data(){
        return{
            sort_list:sort_type.map((name, index) =>{
                return { name, index, on:true, reverse:false};
            })
        }
    }
}
</script>