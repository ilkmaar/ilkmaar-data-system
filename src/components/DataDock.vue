<template>
    <div class="data-dock">
      <DataSheet 
        v-for="sheet in sheets" 
        :key="sheet.id" 
        :sheet="sheet"
        @select="selectSheet"
      />
    </div>
  </template>
  
  <script>
  import DataSheet from './DataSheet.vue';
  
  export default {
    name: 'DataDock',
    components: {
      DataSheet,
    },
    props: {
      sheets: {
        type: Array,
        required: true,
      },
    },
    methods: {
      selectSheet(sheet) {
        this.$emit('select', sheet);
      },
    },
  };
  </script>
  
<style scoped>

.data-dock {
  display: flex;
  align-items: center;
  overflow-x: auto;
  background: lightblue;
  color: #333;
  padding: 0 15px;
  box-sizing: border-box;
  flex-shrink: 0;
  height: 70px; /* Set a fixed height for the data dock */
}

</style>