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
    props: ['data', 'columns', 'selectedXColumn', 'selectedYColumn', 'selectedRepresentation'],
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
            if (!this.data || !this.columns) {
                return;
            }
            console.log("drawing table");

            const cellValues = this.columns.map(column => {
                // Using the column.name to find the value in each row.
                // If there's no data, it will generate an empty array.
                return this.data.map(row => row[column.name] || "");
            });

            this.graph = {
                data: [{
                    type: 'table',
                    header: {
                        values: this.columns.map(column => column.displayName),
                        align: "left",
                        fill_color: 'turquoise' // Please correct the typo in the color name
                    },
                    cells: {
                        values: cellValues,
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
            if (!this.data || !this.columns) {
                return;
            }
            console.log("drawing bar");

            const xColumn = this.selectedXColumn || this.columns.find(column => column.scale === 'categorical');
            const yColumn = this.selectedYColumn || this.columns.find(column => column.scale === 'numeric');

            if (!xColumn || !yColumn) {
                console.log('Appropriate columns not found for bar graph');
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
                    x: this.data.map(row => row[xColumn]),
                    y: this.data.map(row => row[yColumn]),
                    marker: {
                        color: this.data.map(row => color_dict[row[xColumn]] || 'grey')  // Default color 'grey' if not found in color_dict
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
            if (!this.data || !this.columns) {
                return;
            }
            console.log("drawing plot");
            const xColumn = this.selectedXColumn || this.columns.find(column => column.scale === 'categorical');
            const yColumn = this.selectedYColumn || this.columns.find(column => column.scale === 'numeric');

            if (!xColumn || !yColumn) {
                console.log('Appropriate columns not found for scatter plot');
                return;
            }

            this.graph = {
                data: [{
                    type: 'scatter',
                    mode: 'markers',
                    x: this.data.map(row => row[xColumn]),
                    y: this.data.map(row => row[yColumn]),
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
            console.log("drawing map");

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
        selectAndCreateRepresentation() {
            switch(this.selectedRepresentation) {
                    case 'table':
                        console.log("choosing table")
                        this.table();
                        break;
                    case 'bar':
                        console.log("choosing bar")
                        this.bar();
                        break;
                    case 'plot':
                        console.log("choosing plot")
                        this.plot();
                        break;
                    case 'map':
                        console.log("choosing map")
                        this.map();
                        break;
                    default:
                        console.log("error")
                        break;
                }
        }
    },
    created() {
        this.selectAndCreateRepresentation()
    },
    watch: {
        data: {
            immediate: true,
            handler(newData) {
                if (!newData) {
                    return;
                }
                console.log("new data")
                this.selectAndCreateRepresentation();
            }
        },
        columns: {
            immediate: true,
            handler(newColumns) {
                if (!newColumns) {
                    return;
                }
                console.log("new columns")
                this.selectAndCreateRepresentation();
            }
        },
        selectedRepresentation: {
            immediate: true,
            handler(new_representation) {
                if (!new_representation) {
                    return;
                }
                console.log("new representation")
                this.selectAndCreateRepresentation();
            }
        },
        selectedXColumn: {
            immediate: true,
            handler(new_column) {
                if (!new_column) {
                    return;
                }
                console.log("new X column")
                this.selectAndCreateRepresentation();
            }
        },
        selectedYColumn: {
            immediate: true,
            handler(new_column) {
                if (!new_column) {
                    return;
                }
                console.log("new Y column")
                this.selectAndCreateRepresentation();
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