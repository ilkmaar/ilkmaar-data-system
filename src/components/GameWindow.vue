<template>
  <div class="game-window">
    <div class="game-background"></div>
    <div class="game-content">
      <div>
        <WorldInteractor :sheets="sheets" @query-response="queryResponse" />
        <CreatureInteractor :sheets="sheets" @query-response="queryResponse" />
      </div>
      <DataSheet v-if="dataSheetVisible" :sheet="sheet" @select="sheetClicked"/>
      <QuickView v-if="quickViewVisible" :sheet="sheet" @trashSheet="trashSheet" @saveSheetandCloseQuickView="saveSheetandCloseQuickView" @openSheetInDataVisualizer="openSheetInDataVisualizer" />
    </div>
  </div>
</template>

<script>
import WorldInteractor from './WorldInteractor.vue';
import CreatureInteractor from './CreatureInteractor.vue';
import DataSheet from './DataSheet.vue';
import QuickView from './QuickView.vue';

export default {
  name: 'GameWindow',
  components: {
    DataSheet,
    QuickView,
    CreatureInteractor,
    WorldInteractor
  },
  props: {
    sheets: {
      type: Array,
      required: false,
    },
  },
  data() {
    return {
      dataSheetVisible: false,
      quickViewVisible: false,
      sheet: null
    };
  },
  methods: {
    sheetClicked(sheet) {
      this.sheet = sheet
      this.showQuickView();
    },

    queryResponse(sheet) {
      this.sheet = sheet;
      this.dataSheetVisible = true;
      console.log("gamewindow received sheet: ", sheet)
    },

    trashSheet(sheet) {
      if(sheet){
        this.hideQuickView();
        this.dataSheetVisible = false;
        this.sheet = null;
      }
    },

    saveSheetandCloseQuickView(sheet) {
      this.$emit('addSheetToInventory', sheet);
      this.trashSheet(sheet)
    },

    openSheetInDataVisualizer(sheet) {
      this.$emit('openSheetInDataVisualizer', sheet);
      this.trashSheet(sheet);
    },

    showQuickView() {
      this.quickViewVisible = true;
    },

    hideQuickView() {
      this.quickViewVisible = false;
    },
  },
};
</script>

<style scoped>
.game-window {
  position: absolute;
  top: 0px;
  width: calc(100%);
  height: calc(100%);
  background-color: white;
}
.game-background {
  position: absolute;
  top: 0;
  bottom: 0;
  left: 0;
  right: 0;
  background: url('../../public/background-image.png') no-repeat center center fixed; 
  opacity: 0.75;
  -webkit-background-size: cover;
  -moz-background-size: cover;
  -o-background-size: cover;
  background-size: cover;
}

.game-content {
  position: relative;
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
}
</style>