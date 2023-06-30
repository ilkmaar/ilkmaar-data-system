<template>
  <div id="app">
    <div class="top-bar">
      <select class="player-selector" v-model="selectedPlayerName">
        <option v-for="player in players" :key="player.name" :value="player.name">{{ player.name }}</option>
      </select>
      <DataDock :sheets="pinnedSheets" @select="selectSheet" />
      <div class="divider"></div>
      <DataDock :sheets="dockedSheets" @select="selectSheet" />
    </div>
    <div class="game-container">
      <GameWindow :player="selectedPlayerName" @openSheetInDataVisualizer="openSheetInDataVisualizer" @addSheetToInventory="addSheetToInventory" />
      <DataVisualizer v-if="sheetSelectedToVisualize" :sheet="sheetSelectedToVisualize" @saveSheet="saveSheet" @craftSheet="craftSheet" @deselect="deselectSheet" @togglePinSheet="togglePinSheet"/>
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
import { v4 as uuidv4 } from 'uuid';

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
      players: [], // Array to store the list of players
      selectedPlayerName: null, // New property for storing the selected player
      sheetSelectedToVisualize: null,
      sheetToSquish: null,
      inventoryOpen: false,
      crafting: false
    };
  },
  methods: {
    selectSheet(sheet) {
      if(this.crafting) {
          this.sheetToSquish = sheet;
          this.squishSheets(this.sheetSelectedToVisualize, sheet)
      } else {
        this.updateDock(sheet);
        this.sheetSelectedToVisualize = sheet;
      }
    },

    generateUniqueId() {
          return uuidv4();
    },

    deselectSheet() {
        this.sheetSelectedToVisualize = null;
        this.crafting = false;
    },

    craftSheet() {
        this.crafting = true;
    },

    squishSheets(sheet1, sheet2) {
        console.log("squishing sheet1: ", sheet1)
        console.log("squishing sheet2: ", sheet2)

        let baseUrl = 'http://localhost:8000'
        let ep1 = sheet1.endpoint.split(baseUrl)[1];
        let ep2 = sheet2.endpoint.split(baseUrl)[1];

        console.log("endpoint1: ", ep1)
        console.log("endpoint2: ", ep2)

        let endpoint = baseUrl + '/squish' + ep1 + ep2;
        console.log("endpoint: ", endpoint)

        var newSheet = {};
        axios.get(endpoint + '/metadata')
          .then(response => {
              newSheet.id = this.generateUniqueId();
              newSheet.title = "Squished Sheet";
              newSheet.type = "Squished Data";
              newSheet.endpoint = endpoint;
              newSheet.icon = "icon.png";
              newSheet.metadata = response.data;

              this.sheetToSquish = null;
              this.crafting = false;
              console.log("newSheet: ", newSheet)

              this.addSheetToInventory(newSheet)
              this.sheetSelectedToVisualize = newSheet;
          });
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
      this.saveSheets();
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
      this.saveSheets();
    },

    openSheetInDataVisualizer(sheet) {
      this.updateDock(sheet)
      this.sheetSelectedToVisualize = sheet;
    },

    updateDock(sheet_to_add) {
      // first, remove the selected sheet if it's already in dockedSheets
      const index = this.dockedSheets.findIndex(sheet => sheet.title === sheet_to_add.title);
      if (index > -1) {
        this.dockedSheets.splice(index, 1);
      }

      // then, add the selected sheet to the top of dockedSheets
      this.dockedSheets.unshift(sheet_to_add);

      // finally, if the length of dockedSheets is greater than 5, remove the last element
      if (this.dockedSheets.length > 5) {
        this.dockedSheets.pop();
      }
      this.saveSheets();
    },

    toggleInventory() {
      this.inventoryOpen = !this.inventoryOpen;
    },

    saveSheets() {
      localStorage.setItem('sheets', JSON.stringify(this.players));
    },
  },
  created() {
    const saveState = false;
    const localData = localStorage.getItem('sheets');
    if (saveState && localData) {
      this.players = JSON.parse(localData);
      if(this.players.length > 0) {
        this.selectedPlayerName = this.players[7].name;
      }
    } else {
      axios.get('/sheets.json').then(response => {
        this.players = response.data;
        if(this.players.length > 0) {
          this.selectedPlayerName = this.players[7].name;
        }
        // save to local storage for future loads
        localStorage.setItem('sheets', JSON.stringify(this.players));
      });
    }
  },
  computed: {
      selectedPlayer() {
        const player = this.players.find(p => p.name === this.selectedPlayerName);
        return player ? player : null;
      },
      pinnedSheets() {
        return this.selectedPlayer ? this.selectedPlayer.pinnedSheets : [];
      },
      dockedSheets() {
        return this.selectedPlayer ? this.selectedPlayer.dockSheets : [];
      },
      inventorySheets() {
        return this.selectedPlayer ? this.selectedPlayer.inventorySheets : [];
      }
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

.player-selector {
  margin-left:15px;
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