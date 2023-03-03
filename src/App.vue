<template>
  <main :style="grid_style">
    <div class="file-reader">
      <FileReader/>
    </div>
    <div class="linechart">
      <LineChart/>
    </div>
    <div class="toolbar">
      <ToolBar/>
    </div>
    <div class="analysis-module">
      <AnalysisModule/>
    </div>
    <!--It is necessary to put CodeCenter at the last-->
    <div class="codeslist">
      <CodeCenter @change="changeColumns"/>
    </div>
  </main>
</template>

<style scoped>
main {
  flex: 1;
  padding: 40px;
  color: black;
  display: grid;
  grid-template-columns: 220px var(--totalwidth) 920px;
  grid-template-rows: 50px 500px 450px;
  grid-gap: 6px;
  position: absolute; left: 50%;
  transform: translate(-50%, 0);
}

.file-reader{
  grid-column: 1;
  grid-row: 1;
}

.toolbar{
  grid-column: 1;
  grid-row: 2 / 4;
}

.linechart{
  grid-column: 3;
  grid-row: 1 / 3;
}

.codeslist{
  grid-column: 2;
  grid-row: 1 / 4;
}

.analysis-module{
  grid-column: 3;
  grid-row: 3;
}

</style>

<script>
import FileReader from './components/FileReader.vue'
import LineChart from './components/LineChart.vue';
import ToolBar from './components/ToolBar.vue';
import AnalysisModule from './components/AnalysisModule.vue';
import CodeCenter from './components/CodeCenter.vue';

export default{
    data() {
        return {
          grid_style:{'--totalwidth': '290px'}
        };
    },
    methods: {
      changeColumns(msg){
        this.grid_style['--totalwidth'] = 290 * msg +'px'
      }
    },
    mounted() {
    },
    components: { FileReader, LineChart, ToolBar, AnalysisModule, CodeCenter}
}
</script>