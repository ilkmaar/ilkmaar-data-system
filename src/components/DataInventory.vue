<template>
  <div class="data-inventory">
    <div class="inventory-tabs">
      <h3 
        v-for="(sectionSheets, section) in sections" 
        :key="section" 
        @click="activeSection = section"
        :class="{ 'active-tab': activeSection === section }"
      >
        {{ section }}
      </h3>
    </div>
    <div class="inventory-section" v-if="activeSection">
      <div class="inventory-section-content">
        <DataSheet v-for="sheet in sections[activeSection]" :key="sheet.id" :sheet="sheet" @select="selectSheet"/>
      </div>
    </div>
  </div>
</template>
  
<script>
import DataSheet from './DataSheet.vue';

export default {
  name: 'DataInventory',
  components: {
    DataSheet,
  },
  props: {
    sheets: {
      type: Array,
      default: () => [],
    },
  },
  data() {
    return {
      activeSection: null,
    };
  },
  created() {
    if (this.sheets.length > 0) {
      this.activeSection = this.sheets[0].type;
    }
  },
  computed: {
    sections() {
      return this.sheets.reduce((sections, sheet) => {
        if (!sections[sheet.type]) {
          sections[sheet.type] = [];
        }
        sections[sheet.type].push(sheet);
        return sections;
      }, {});
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
.data-inventory {
  width: 100%;
  height:25%;
  padding: 5px;
  background: #fff;
  border-radius: 5px;
  margin-top: 5px;
}

.inventory-tabs {
  display: flex;
  justify-content: flex-start;  /* Updated this line */
  border-bottom: 1px solid #ccc;
}

.inventory-tabs h3 {
  margin: 0;
  padding: 5px;
  cursor: pointer;
  background-color: #f0f0f0;
  border: 1px solid #ccc;
  border-bottom: none;
}

.inventory-tabs h3.active-tab {
  background-color: #fff;
  border-bottom: 1px solid #fff;
}

.inventory-section {
  padding-top: 20px;
}

.inventory-section-content {
  display: flex;
  overflow-x: auto;
}

.inventory-section-content > * {
  flex: 0 0 auto;
  width: 200px; /* or whatever fixed width you prefer */
  margin-right: 10px;
}

</style>