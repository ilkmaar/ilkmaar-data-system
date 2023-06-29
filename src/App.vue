<template>
  <div id="app">
    <div class="top-bar">
      <DataDock :sheets="pinnedSheets" @select="selectSheet" />
      <div class="divider"></div>
      <DataDock :sheets="dockedSheets" @select="selectSheet" />
    </div>
    <div class="game-container">
      <GameWindow :sheets="sheets" @openSheetInDataVisualizer="openSheetInDataVisualizer" @addSheetToInventory="addSheetToInventory" />
      <DataVisualizer v-if="sheetSelectedToVisualize" :sheet="sheetSelectedToVisualize" @saveSheet="saveSheet" @deselect="deselectSheet" @togglePinSheet="togglePinSheet"/>
    </div>
    <DataInventory :sheets="inventorySheets" :inventoryOpen="inventoryOpen" @select="selectSheet" @toggle-inventory="toggleInventory"/>
  </div>
</template>

<script>
import DataDock from './components/DataDock.vue';
import GameWindow from './components/GameWindow.vue'
import DataInventory from './components/DataInventory.vue';
import DataVisualizer from './components/DataVisualizer.vue';
import axios from 'axios';

export default {
  name: 'App',
  components: {
    GameWindow,
    DataDock,
    DataInventory,
    DataVisualizer
  },
  data() {
    return {
      sheets: [],
      inventorySheets: [],
      pinnedSheets: [],
      dockedSheets: [],
      sheetSelectedToVisualize: null,
      inventoryOpen: false,
    };
  },
  methods: {
    selectSheet(sheet) {
      this.updateDock(sheet);

      // for now automatically select to visualize
      this.sheetSelectedToVisualize = sheet;
    },

    deselectSheet() {
      this.sheetSelectedToVisualize = null;
    },

    saveSheet(selectedRepresentation, selectedXColumn, selectedYColumn) {
        this.sheetSelectedToVisualize.metadata.selectedRepresentation = selectedRepresentation;
        this.sheetSelectedToVisualize.metadata.selectedXColumn = selectedXColumn;
        this.sheetSelectedToVisualize.metadata.selectedYColumn = selectedYColumn;

        this.addSheetToInventory(this.sheetSelectedToVisualize)
    },

    togglePinSheet(sheet_to_toggle) {
      const index = this.pinnedSheets.findIndex(sheet => sheet.id === sheet_to_toggle.id);
      if (index > -1) {
        this.pinnedSheets.splice(index, 1);
      } else {
        this.pinnedSheets.push(sheet_to_toggle);
      }
    },

    addSheetToInventory(sheet_to_add) {
      // first, remove the selected sheet if it's already in inventorySheets
      const index = this.inventorySheets.findIndex(sheet => sheet.id === sheet_to_add.id);
      if (index > -1) {
        this.inventorySheets.splice(index, 1);
      }

      // then, add the selected sheet to the top of dockedSheets
      this.inventorySheets.unshift(sheet_to_add);
      this.updateDock(sheet_to_add);
    },

    openSheetInDataVisualizer(sheet) {
      this.updateDock(sheet)
      this.sheetSelectedToVisualize = sheet;
    },

    updateDock(sheet_to_add) {
      // first, remove the selected sheet if it's already in dockedSheets
      const index = this.dockedSheets.findIndex(sheet => sheet.id === sheet_to_add.id);
      if (index > -1) {
        this.dockedSheets.splice(index, 1);
      }

      // then, add the selected sheet to the top of dockedSheets
      this.dockedSheets.unshift(sheet_to_add);

      // finally, if the length of dockedSheets is greater than 5, remove the last element
      if (this.dockedSheets.length > 5) {
        this.dockedSheets.pop();
      }
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
      this.inventorySheets = this.sheets.filter(sheet => data.inventorySheets.includes(sheet.id));
    });
  },
};
</script>

<style>
#app {
  height: calc(100vh - 16px);
  display: flex;
  flex-direction: column;
  align-items: stretch;
  background: #f5f5f5;
}

.top-bar {
  height: 70px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-shrink: 0;
  background: lightblue;
  color: #333;
  box-sizing: border-box;
}

.divider {
  border-left: 1px solid #999;
  height: 100%;
}

.game-container {
  position: relative;
  flex: 1; /* This allows the game-container to take up the remaining space */
  overflow: hidden; /* This prevents the DataVisualizer from spilling out of the container */
}

</style>