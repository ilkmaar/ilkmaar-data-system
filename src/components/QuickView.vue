<template>
  <div class="quick-view" v-if="sheet">
    <div class="quick-view-buttons">
      <button @click="trashSheet">Trash</button>
      <button @click="saveSheetToInventory">Save</button>
      <button @click="openSheetInDataVisualizer">Explore</button>
    </div>
    <div class="data-view" v-if="data">
        <DataView :data="data" :columns="columns" :selectedXColumn="selectedXColumn" :selectedYColumn="selectedYColumn" :selectedRepresentation="selectedRepresentation"/>
    </div>
  </div>
</template>

<script>
//import data from './fakeData.js';
import DataView from './DataView.vue';
import axios from 'axios';

export default {
  name: 'QuickView',
  props: {
    sheet: {
      type: Object,
      required: true,
    }
  },
  components: {
    DataView
  },
  data() {
    return {
      data: null,
      columns: this.sheet.metadata.columns,
      selectedXColumn: this.sheet.metadata.selectedXColumn || this.sheet.metadata.default_column_x,
      selectedYColumn: this.sheet.metadata.selectedYColumn || this.sheet.metadata.default_column_y,
      selectedRepresentation: this.sheet.metadata.defaultView || this.sheet.metadata.defaultView
    };
  },
  methods: {
    trashSheet() {
      this.$emit('trashSheet', this.sheet);
    },
    saveSheetToInventory() {
      this.$emit('saveSheetandCloseQuickView', this.sheet);
    },
    openSheetInDataVisualizer() {
      this.$emit('openSheetInDataVisualizer', this.sheet);
    },
    fetchData() {
      if (this.sheet) {
          axios.get(this.sheet.sqlQuery)
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
          this.fetchData();
        } else {
          console.log("sheet");
        }
      },
    }
  }
};
</script>

<style scoped>

.quick-view {
  position: absolute;
  width: 250px;
  height: 200px;
  top: 250px;
  left: 250px; 
  background-color: aquamarine;
  padding: 5px;
  z-index: 300;
  display: flex;
  flex-direction: column;
}

.data-view {
  position: relative;
  padding: 3px;
  height: 100%;
  flex-grow: 1;
  overflow-y: auto; /* Enables vertical scrolling */
  max-width: 100%; /* Makes sure the DataView doesn't overflow its parent's width */
  max-height: calc(100%-15px); /* Makes sure the DataView doesn't overflow its parent's height */
}

</style>