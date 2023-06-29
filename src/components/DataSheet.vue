<template>
  <div class="data-sheet" @click="selectSheet" @mouseover="mouseOver" @mouseleave="mouseLeave">
    <!-- Icon based on the default representation of the dataset -->
    <div class="icon-wrapper"
        :style="`box-shadow: 0 0 0 3px ${outerColor}`">
        <img :src="sheet.icon" alt="icon" />
    </div>
    <!-- Title of the dataset -->
    <h4 class="title" :title="sheet.title">{{ sheet.title }}</h4>
    <!-- Tooltip overlay -->
    <div v-if="showTooltip" class="tooltip">{{ sheet.title }}</div>
  </div>
</template>

<script>
export default {
  name: 'DataSheet',
  props: {
    sheet: {
      type: Object,
      required: true
    },
  },
  emits: ['select', 'mouseover', 'mouseleave'],  // Define the custom events here
  data() {
    return {
      showTooltip: false
    };
  },
  methods: {
    selectSheet() {
      this.$emit('select', this.sheet);
    },
    mouseOver() {
      this.$emit('mouseover');
    },
    mouseLeave() {
      this.$emit('mouseleave');
    },
    toggleTooltip(show) {
      this.showTooltip = show;
    },
  },
  computed: {
    outerColor() {
      switch(this.sheet.metadata.defaultView) {
        case 'table':
          return("black")
        case 'bar':
          return("red")
        case 'plot':
          return("green")
        case 'map':
          return("blue")
        default:
          return("black")
      }
    }
  }
};
</script>

<style scoped>
.data-sheet {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin: 0 0px;
  padding: 10px;
  color: #666;
  cursor: pointer;
  position: relative; /* Make it a positioning context for tooltip */
}

.icon-wrapper {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #eee;
}

.icon-wrapper img {
  max-width: 100%;
  max-height: 100%;
}

.title {
  margin-top: 0px;
  margin-block-start: 0px;
  margin-block-end: 0px;
  font-size: 12px;
  text-align: center;
  max-width: 60px; /* Adjust the maximum width as needed */
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  transition: all 0.3s;
}

.tooltip {
  position: absolute;
  top: 100%; /* Position it below the title */
  left: 50%;
  transform: translateX(-50%);
  background-color: rgba(0, 0, 0, 0.8);
  color: #fff;
  padding: 5px 10px;
  border-radius: 5px;
  font-size: 12px;
  white-space: nowrap;
  pointer-events: none; /* Prevent it from affecting mouse interactions */
  opacity: 0; /* Initially hidden */
  transition: opacity 0.3s;
}

.data-sheet:hover .tooltip {
  opacity: 1; /* Show tooltip on hover */
}
</style>