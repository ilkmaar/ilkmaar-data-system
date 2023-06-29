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

        <div class="representation-selectors">
          <button 
            v-for="representation in representations" 
            :key="representation.id" 
            :class="{'selected': representation.id === selectedRepresentation.id}"
            @click="selectRepresentation(representation)"
            :disabled="!isRepresentationAvailable(representation)"
          >
            {{ representation.name }}
          </button>
        </div>
  
        <div class="axis-selectors" v-if="selectedRepresentation.id === 'plot' || selectedRepresentation.id === 'bar'">
          <select v-model="selectedXColumn">
            <option disabled value="">Select X column</option>
            <option v-for="column in sheet.metadata.columns" :key="column.name" :value="column.name">{{ column.displayName }}</option>
          </select>
          <span>vs.</span>
          <select v-model="selectedYColumn">
            <option disabled value="">Select Y column</option>
            <option v-for="column in sheet.metadata.columns" :key="column.name" :value="column.name">{{ column.displayName }}</option>
          </select>
        </div>

        <div class="data-view-container" v-if="data">
            <DataView :data="data" :metadata="metadata" :selectedRepresentation="selectedRepresentation" :selectedXColumn="selectedXColumn" :selectedYColumn="selectedYColumn"/>
        </div>

      </div>
    </div>
    <div v-else>
      <p>Please select a data sheet to visualize.</p>
    </div>
</template>
  
<script>
  import fakeData from './fakeData.js';
  import DataView from './DataView.vue';
//  import axios from 'axios';
  
  export default {
    props: ['sheet'],
    components: {
      DataView
    },  
    data() {
        return {
            data: null,
            metadata: null,
            graph: null,
            representations: [
                { id: 'table', name: 'Table' },
                { id: 'bar', name: 'Bar Chart' },
                { id: 'plot', name: 'Plot' },
                { id: 'map', name: 'Map' },
            ],
            selectedRepresentation: { id: 'table', name: 'Table' },
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
      this.selectedRepresentation = representation;
      this.metadata.representation = representation.id;
    },
    isRepresentationAvailable(representation) {
      if (representation.id === 'map') {
        return this.sheet && this.sheet.metadata.columns && this.sheet.metadata.columns.some(column => column.type === 'location');
      }
      return true;
    },
    fetchData() {
      if (this.sheet) {
          this.data = fakeData[this.sheet.id];
          this.metadata = this.sheet.metadata;
          this.selectedXColumn = this.sheet.metadata.default_column_x;
          this.selectedYColumn = this.sheet.metadata.default_column_y;
      }
      // axios.get('localhost:8000/creatures').then(response => {
      //   this.creatures = response;
      // });
    }
  },
  watch: {
      sheet: {
        immediate: true,
        handler(newSheet) {
          if (newSheet) {
            this.fetchData();
          }
        },
      },
      selectedXColumn: function (x_column) {
        console.log("selected x column changed 1")
        this.metadata.selectedXColumn = x_column;
      },
      selectedYColumn: function (y_column) {
        this.metadata.selectedYColumn = y_column;
      },
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