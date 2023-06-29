<template>
  <div class="quick-view" v-if="sheet">
    <div class="quick-view-buttons">
      <button @click="trashSheet">Trash</button>
      <button @click="saveSheetToInventory">Save</button>
      <button @click="openSheetInDataVisualizer">Explore</button>
    </div>
    <div class="data-view" v-if="data">
        <DataView :data="data" :metadata="metadata" />
    </div>
  </div>
</template>

<script>
import data from './fakeData.js';
import DataView from './DataView.vue';

export default {
  name: 'QuickView',
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
      metadata: null
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
          this.data = data[this.sheet.id];
          this.metadata = this.sheet.metadata;
          this.metadata.representation = this.sheet.metadata.defaultView;
          this.metadata.representation = this.sheet.metadata.defaultView;
          this.metadata.representation = this.sheet.metadata.defaultView;
      }
    }
  },
  watch: {
      sheet: {
        immediate: true,
        handler(newSheet) {
          if (newSheet) {
            this.fetchData();

            // set default representation
            //const defaultRepresentation = this.representations.find(representation => representation.id === newSheet.metadata.defaultView);
            //this.selectRepresentation(defaultRepresentation || 'table');

            // set default columns
            //this.selectedXColumn = newSheet.metadata.default_column_x || '';
            //this.selectedYColumn = newSheet.metadata.default_column_y || '';
          }
        }
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