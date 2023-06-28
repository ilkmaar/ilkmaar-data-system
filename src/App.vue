<template>
  <div id="app">
    <div class="top-bar">
      <DataDock :sheets="pinnedSheets" @select="selectSheet" />
      <div class="divider"></div>
      <DataDock :sheets="dockedSheets" @select="selectSheet" />
      <DataInventoryButton @toggle="toggleInventory" />
    </div>
    <DataVisualizer :sheet="selectedSheet" @deselect="deselectSheet"/>
    <DataInventory v-if="inventoryOpen" :sheets="sheets" @select="selectSheet"/>
  </div>
</template>

<script>
import DataDock from './components/DataDock.vue';
import DataInventory from './components/DataInventory.vue';
import DataVisualizer from './components/DataVisualizer.vue';
import DataInventoryButton from './components/DataInventoryButton.vue';
import axios from 'axios';

export default {
  name: 'App',
  components: {
    DataDock,
    DataInventory,
    DataVisualizer,
    DataInventoryButton,
  },
  data() {
    return {
      sheets: [],
      pinnedSheets: [],
      dockedSheets: [],
      selectedSheet: null,
      inventoryOpen: false,
    };
  },
  methods: {
    selectSheet(sheet) {
      this.selectedSheet = sheet;
    },
    deselectSheet(sheet) {
      this.selectedSheet = sheet;
    },
    toggleInventory() {
      this.inventoryOpen = !this.inventoryOpen;
    },
  },
  created() {
    axios.get('/sheets.json').then(response => {
      const data = response.data;
      this.sheets = data.sheets;
      this.pinnedSheets = this.sheets.filter(sheet => data.pinnedSheets.includes(sheet.id));
      this.dockedSheets = this.sheets.filter(sheet => data.dockSheets.includes(sheet.id));
    });
  },
};
</script>

<style>
#app {
  display: flex;
  flex-direction: column;
  align-items: stretch;
  background: #f5f5f5;
  min-height: 100vh;
  height: 100%;
}

.top-bar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-shrink: 0;
  height: 70px;
  background: lightblue;
  color: #333;
  padding: 0px;
  box-sizing: border-box;
}

.divider {
  border-left: 1px solid #999;
  height: 100%;
}
</style>