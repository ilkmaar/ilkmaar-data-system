<template>
  <div class="visualizer" v-if="sheet">
    <div class="title-bar">
      <h2 class="title">{{ sheet.title }}</h2>
      <button class="close-button" @click="deselectSheet">X</button>
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
        console.log("no data: ", data)
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
  display: flex;
  width: 65%;
  height: 80%;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  margin: 10px;
  border: 1px solid #ccc;
  border-radius: 10px;
  padding: 20px;
  background-color: #f9f9f9;
}

.title-bar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 10px;
}

.title {
  margin: 0;
  text-align: center;
}

.close-button {
  background: none;
  border: none;
  font-size: 1.5em;
  cursor: pointer;
}

.representation-selectors {
  display: flex;
  justify-content: center;
  margin-bottom: 10px;
}

.representation-selectors button {
  margin: 0 10px;
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
  display: flex;
  justify-content: center;
  margin-bottom: 10px;
}

.axis-selectors select {
  margin: 0 10px;
}

.view-box {
  width: 100%;
  height: 500px;
  position: relative; /* Make this a positioned parent */
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