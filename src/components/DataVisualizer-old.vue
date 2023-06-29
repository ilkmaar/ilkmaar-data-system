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

        <div class="view-box">
          <div>
            <div v-if="graph">
              <VuePlotly class="plotly-graph" :data="graph.data" :layout="graph.layout" />
            </div>
            <div v-else>
              Loading...
            </div>
          </div>
        </div>
    </div>
  </div>
  <div v-else>
    <p>Please select a data sheet to visualize.</p>
  </div>
</template>

<script>
import { VuePlotly } from 'vue3-plotly';
import data from './fakeData.js';

export default {
  props: ['sheet'],
  components: {
    VuePlotly
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
      selectedRepresentation: null,
      selectedXColumn: '',
      selectedYColumn: '',
    };
  },
  methods: {
    togglePinSheet() {
      this.$emit('togglePinSheet');
    },
    deselectSheet() {
      this.$emit('deselect');
    },
    selectRepresentation(representation) {
      this.selectedRepresentation = representation;
      
      // Clear existing graph
      this.graph = null;
      
      switch(representation.id) {
        case 'table':
          this.table();
          break;
        case 'bar':
          this.bar();
          break;
        case 'plot':
          this.plot();
          break;
        case 'map':
          this.map();
          break;
        default:
          console.error(`Unknown representation: ${representation}`);
          break;
      }
    },
    isRepresentationAvailable(representation) {
      if (representation.id === 'map') {
        return this.sheet && this.sheet.columns && this.sheet.columns.some(column => column.type === 'location');
      }
      return true;
    },

    table() {
      if (!this.data) {
        return;
      }
      this.graph = {
        data: [{
          type: 'table',
          header: {
            values: this.sheet.metadata.columns.map(column => column.displayName),
            align: "left",
            fill_color: 'paleturquoise'
          },
          cells: {
            values: Object.keys(this.data[0]).map(key => this.data.map(row => row[key])),
            align: "left",
            fill_color: 'lavender'
          }
        }],
        layout: {
          autosize: true
        }
      };
    },

    bar() {
      if (!this.data) {
        return;
      }
      const color_dict = {
        'Shadow': 'darkblue',
        'Light': 'yellow',
        'Growth': 'green',
        'Stability': 'grey'
      };

      this.graph = {
        data: [{
          type: 'bar',
          x: this.data.map(row => row[this.selectedXColumn]),
          y: this.data.map(row => row[this.selectedYColumn]),
          marker: {
            color: this.data.map(row => color_dict[row[this.selectedXColumn]])
          }
        }],
        layout: {
          autosize: true,
          barmode: 'group'
        }
      };
    },

    plot() {
      if (!this.data) {
        return;
      }
      this.graph = {
        data: [{
          type: 'scatter',
          mode: 'markers',
          x: this.data.map(row => row[this.selectedXColumn]),
          y: this.data.map(row => row[this.selectedYColumn]),
        }],
        layout: {
          autosize: true
        }
      };
    },

    map() {
      if (!this.data) {
        return;
      }
      this.graph = {
        data: [{
          type: 'scatter',
          mode: 'markers',
          x: this.data.map(row => row[this.selectedXColumn]),
          y: this.data.map(row => row[this.selectedYColumn]),
          marker: { color: this.data.map(row => row['color_column']) }
        }],
        layout: {
          autosize: true,
          images: [{
            source: "https://i.imgur.com/MH1kzqg.png",
            xref: "x",
            yref: "y",
            x: -100,
            y: 100,
            sizex: 200,
            sizey: 200,
            sizing: "stretch",
            opacity: 0.5,
            layer: "below"
          }],
          xaxis: { range: [-100, 100], automargin: true },
          yaxis: { range: [-100, 100], automargin: true },
        }
      };
    },
    fetchData() {
      if (this.sheet) {
          this.data = data[this.sheet.id];
      }
      // try {
      //   // replace this URL with the actual endpoint
      //   const response = await fetch(`https://your-api-endpoint/${this.sheet.query}`);
        
      //   if (!response.ok) {
      //     throw new Error(`HTTP error! status: ${response.status}`);
      //   }
        
      //   this.data = await response.json();
      // } catch (error) {
      //   console.log('Fetch error: ', error);
      // }
      this.table();
    }
  },
  watch: {
      sheet: {
        immediate: true,
        handler(newSheet) {
          if (newSheet) {
            this.fetchData();

            // set default representation
            const defaultRepresentation = this.representations.find(representation => representation.id === newSheet.metadata.defaultView);
            this.selectRepresentation(defaultRepresentation || this.representations[0]);

            // set default columns
            this.selectedXColumn = newSheet.metadata.default_column_x || '';
            this.selectedYColumn = newSheet.metadata.default_column_y || '';
          }
        },
      },
      selectedXColumn: function () {
        this.selectRepresentation(this.selectedRepresentation)
      },
      selectedYColumn: function () {
        this.selectRepresentation(this.selectedRepresentation)
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
  height: 50px; /* or any other value */
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

.view-box {
  width: 100%;
  position: relative; /* Make this a positioned parent */
  flex-grow: 1;
}

.plotly-container {
  width: 100%; /* Set the container width to occupy full space */
  max-width: 100%; /* Add max-width to avoid exceeding the parent container */
  height: 100%; /* Set the container height to occupy full space */
}

.plotly-graph {
  position: absolute; /* Absolutely position the graph */
  top: 0; /* Align to the top */
  right: 0; /* Align to the right */
  bottom: 0; /* Align to the bottom */
  left: 0; /* Align to the left */
}
</style>