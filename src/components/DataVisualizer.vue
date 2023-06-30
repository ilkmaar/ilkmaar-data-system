<template>
    <div class="visualizer" v-if="sheet">
      <div class="panel-container">
        <div class="title-bar">
          <button class="pin-button" @click="togglePinSheet">UN/PIN</button>
          <div class="title-container">
            <h2 class="title">{{ sheet.title }}</h2>
          </div>
          <button class="close-button" @click="deselectSheet">CLOSE</button>
        </div>

        <div class="tools-bar">
          <button class="save-button" @click="saveSheet">SAVE</button>
        </div>

        <div class="representation-selectors">
          <button 
            v-for="representation in representations" 
            :key="representation.id"
            :class="{'selected': representation.id === selectedRepresentation}"
            @click="selectRepresentation(representation)"
            :disabled="!isRepresentationAvailable(representation)"
          >
            {{ representation.name }}
          </button>
        </div>
  
        <div class="axis-selectors" v-if="selectedRepresentation === 'plot' || selectedRepresentation === 'bar'">
          <select v-model="selectedXColumn">
            <option disabled value="">Select X column</option>
            <option v-for="column in columns" :key="column.name" :value="column.name">{{ column.displayName }}</option>
          </select>
          <span>vs.</span>
          <select v-model="selectedYColumn">
            <option disabled value="">Select Y column</option>
            <option v-for="column in columns" :key="column.name" :value="column.name">{{ column.displayName }}</option>
          </select>
        </div>

        <div class="data-view-container" v-if="data">
            <DataView :data="data" :columns="columns" :selectedRepresentation="selectedRepresentation" :selectedXColumn="selectedXColumn" :selectedYColumn="selectedYColumn"/>
        </div>

      </div>
    </div>
    <div v-else>
      <p>Please select a data sheet to visualize.</p>
    </div>
</template>

<script>
//  import fakeData from './fakeData.js';
  import DataView from './DataView.vue';
  import axios from 'axios';
  
  export default {
    props: {
      sheet: {
        type: Object,
        required: true,
      },
    },
    components: {
      DataView
    },
    data() {
        return {
            data: null,
            graph: null,
            representations: [
                { id: 'table', name: 'Table' },
                { id: 'bar', name: 'Bar Chart' },
                { id: 'plot', name: 'Plot' },
                { id: 'map', name: 'Map' },
            ],
            metadata: {},
            columns: [],
            selectedRepresentation: '',
            selectedXColumn: '',
            selectedYColumn: '',
        };
  },
  methods: {
    togglePinSheet() {
      this.$emit('togglePinSheet', this.sheet);
    },
    deselectSheet() {
      this.$emit('deselect', this.sheet);
    },
    selectRepresentation(representation) {
      this.selectedRepresentation = representation.id;
    },
    isRepresentationAvailable(representation) {
      if (representation.id === 'map') {
        return this.columns && this.columns.some(column => column.name === 'location_x' || column.name === 'location_y');
      }
      return true;
    },
    saveSheet() {
        this.$emit('saveSheet', this.selectedRepresentation, this.selectedXColumn, this.selectedYColumn);
    },
    fetchData() {
      if (this.sheet) {
          axios.get(this.sheet.endpoint)
            .then(response => {
                this.data = response.data; // use the returned metadata
          });
      }
    }
  },
  watch: {
    sheet: {
      immediate: true,
      handler(newSheet) {
        if (newSheet) {
            this.metadata = newSheet.metadata;
            this.columns = newSheet.metadata.columns;
            this.selectedRepresentation = newSheet.metadata.selectedRepresentation || newSheet.metadata.defaultView;
            this.selectedXColumn = newSheet.metadata.selectedXColumn || newSheet.metadata.default_column_x;
            this.selectedYColumn = newSheet.metadata.selectedYColumn || newSheet.metadata.default_column_y;
            this.fetchData();
        } else {
            this.metadata = {};
            this.columns = [];
            this.selectedRepresentation = "";
            this.selectedXColumn = "";
            this.selectedYColumn = "";
        }
      },
    }
  }
};
</script>

<style scoped>
.visualizer {
  position: absolute;
  top: 0;
  justify-content: center;
  z-index: 100; 
  background: rgba(0,0,0,0.5); 
  width: 100%; 
  height: 100%;
  display: flex; 
  align-items: center; 
  flex-direction: column;
}

.panel-container {
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
  height: 100%;
  padding: 10px;
  border-radius: 10px;
  background: rgba(200,200,256,.9);
  width: 70%;
}

.title-bar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  height: 50px;
}

.tools-bar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  height: 50px;
}

.title-container {
  text-align: center;
  flex-grow: 1;
}

.title {
  margin: 0;
  text-align: center;
}

.pin-button {
  background: none;
  border: none;
  font-size: 1.5em;
  cursor: pointer;
}
.close-button {
  background: none;
  border: none;
  font-size: 1.5em;
  cursor: pointer;
}
.save-button {
  background: none;
  border: none;
  font-size: 1.5em;
  cursor: pointer;
}

.representation-selectors {
  position: relative;
  display: flex;
  width: 100%;
  justify-content: center;
}

.representation-selectors button {
  margin: 10px;
  padding: 5px;
  border: none;
  border-radius: 5px;
  background-color: #ddd;
  cursor: pointer;
}

.representation-selectors button.selected {
  background-color: #aaa;
}

/*
.representation-selectors,
.axis-selectors {
  display: flex;
  justify-content: center;
  width: 100%;
  margin-bottom: 10px;
}

.data-view-container {
  flex-grow: 1;
  display: flex;
}

.data-view-container DataView {
  height: 100%;
}

/*


.panel-container {
  width: 70%;
  height: 95%;
  background: rgba(200,200,256,.9);
  padding: 10px; 
  border-radius: 10px;
  display: flex; 
  flex-direction: column;
  align-items: center; 
  justify-content: center;
}

.title-bar {
  position: relative;
  top: 0px;
  width: 100%;
  display: flex;
  justify-content: space-between;
  align-items: center;
  height: 50px;
}


.axis-selectors {
  position: relative;
  display: flex;
  width: 100%;
  padding-top: 15px;
  justify-content: center;
  justify-content: center;
  background-color: white;
}

.axis-selectors select {
  margin: 0 10px;
}

*/

</style>