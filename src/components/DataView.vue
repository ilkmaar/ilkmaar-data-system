<template>
    <div class="view-box">
      <div>
        <div class="vue-plotly-container" v-if="graph">
          <VuePlotly class="plotly-graph" :data="graph.data" :layout="graph.layout" />
        </div>
        <div v-else>
          Loading...
        </div>
      </div>
    </div>
</template>
  
<script>
  import { VuePlotly } from 'vue3-plotly';
  
  export default {
    props: ['data', 'metadata', 'selectedXColumn', 'selectedYColumn', 'selectedRepresentation'],
    components: {
      VuePlotly
    },
    data() {
      return {
        graph: null
      };
    },
    methods: {
        table() {
            if (!this.data) {
                return;
            }
            this.graph = {
                data: [{
                    type: 'table',
                    header: {
                        values: this.metadata.columns.map(column => column.displayName),
                        align: "left",
                        fill_color: 'turquiose'
                    },
                    cells: {
                        values: Object.keys(this.data[0]).map(key => this.data.map(row => row[key])),
                        align: "left",
                        fill_color: 'lavender'
                    }
                }],
                layout: {
                    autosize: true,
                    margin: {
                        l: 5,
                        r: 5,
                        b: 5,
                        t: 5,
                        pad: 5
                    }
                }
            };
        },

        bar() {
            if (!this.data) {
                return;
            }
            console.log("bar");

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
                    barmode: 'group',
                    margin: {
                        l: 5,
                        r: 5,
                        b: 5,
                        t: 5,
                        pad: 5
                    }
                }
            };
            },

        plot() {
            if (!this.data) {
                return;
            }
            console.log("plot");

            this.graph = {
                data: [{
                type: 'scatter',
                mode: 'markers',
                    x: this.data.map(row => row[this.selectedXColumn]),
                    y: this.data.map(row => row[this.selectedYColumn]),
                }],
                layout: {
                    autosize: true,
                    margin: {
                        l: 5,
                        r: 5,
                        b: 5,
                        t: 5,
                        pad: 5
                    }
                }
            };
        },

        map() {
            if (!this.data) {
                return;
            }
            console.log("map");

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
                    margin: {
                        l: 5,
                        r: 5,
                        b: 5,
                        t: 5,
                        pad: 5
                    },
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
    },
    created() {
        this.table();
    },
    watch: {
        data: {
            immediate: true,
            handler(newData) {
                // Call the correct method based on the representation
                console.log(newData);
                this.table()
            }
        },
        selectedRepresentation: {
            immediate: true,
            handler(new_representation) {
                if (!new_representation) {
                    return;
                }
                // Call the correct method based on the representation
                console.log("metadatachanged: ", new_representation);
                switch(new_representation.id) {
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
                        console.log("error")
                        break;
                }
            }
        },
        selectedXColumn: {
            immediate: true,
            handler(new_column) {
                console.log("selected x column changed 2")

                if (!new_column) {
                    return;
                }
                // Call the correct method based on the representation
                console.log("column changed: ", new_column);
                
                switch(this.selectedRepresentation.id) {
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
                        console.log("error")
                        break;
                }
            }
        },
        selectedYColumn: {
            immediate: true,
            handler(new_column) {
                // Call the correct method based on the representation
                if (!new_column) {
                    return;
                }
                console.log("metadatachanged: ", new_column);

                switch(this.selectedRepresentation.id) {
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
                        console.log("error")
                        break;
                }
            }
        }
    }
  };
</script>
  
<style scoped>
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
  max-height: 100%;
  max-width: 100%;
  padding: 0px;
  margin:0px;
}
</style>