<template>
  <div class="data-inventory" :style="inventoryStyle">
    <button class="close-button" @click="toggleInventory">Open/Close Data Inventory</button>
    <div class="inventory-tabs"  v-show="inventoryOpen">
      <h3 
        v-for="(sectionSheets, section) in sections" 
        :key="section" 
        @click="activeSection = section"
        :class="{ 'active-tab': activeSection === section }"
      >
        {{ section }}
      </h3>
    </div>
    <div class="inventory-section" v-if="activeSection"  v-show="inventoryOpen">
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
    inventoryOpen: {
      type: Boolean,
      required: true,
    },
    sheets: {
      type: Array,
      default: () => [],
    },
  },
  data() {
    return {
      activeSection: "Info",
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
    inventoryStyle() {
      if (this.inventoryOpen) {
        return {
          height: '20%',  // adjust this to your needs
          overflow: 'auto',  // to enable scroll if the content exceeds the container
        }
      } else {
        return {
          height: '50px',
        }
      }
    }
  },
  methods: {
    selectSheet(sheet) {
      this.$emit('select', sheet);
    },
    toggleInventory() {
      this.$emit('toggle-inventory');
    }
  },
};
</script>

<style scoped>
.data-inventory {
  position: relative;
  bottom: 0px;
  width: 100%;
  background: lightblue;
  z-index: 200;
  transition: height 0.3s ease-in-out; /* animate the height change */
}

.close-button {
  position: absolute;
  right: 15px;
  top: 10px;
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