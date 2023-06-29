<template>
  <div class="world-interactor">
    <div>
      <p class="world-interactor-title">World Interactor</p>
    </div>

    <select class="area-select" v-model="selectedArea">
      <option disabled value="">Select an Area</option>
      <option v-for="area in areas" :key="area.name" :value="area.name">{{ area.name }}</option>
    </select>

    <select class="area-source-select" v-model="selectedSource">
      <option disabled value="">Select a Source</option>
      <option v-for="source in sources" :key="source.id" :value="source.id">{{ source.text }}</option>
    </select>

    <button @click="query">Query</button>
  </div>
</template>

<script>
export default {
  name: 'WorldInteractor',
  props: {
    sheets: {
      type: Array,
      required: false,
    },
  },
  data() {
    return {
      areas: [
          {"name": "Main Island"}, 
          {"name": "Potions Clinic"},
          {"name": "Shadow Island"}
      ],
      sources: [
          {
          "id": "1",
          "text": "Log?"
          },
          {
          "id": "2",
          "text": "Watcher?"
          }
      ],
      selectedArea: "Main Island",
      selectedSource: "1",
      responseSheet: null
    };
  },
  methods: {
    getWorldAreaImage(selectedArea) {
      // here you should return the URL of the image based on the creature name
      if(selectedArea) {
          return("creature.jpeg");        
      }
      else {
          return("creature.jpeg");
      }
    },
    query() {
        // Get a random index from the array
        const randomIndex = Math.floor(Math.random() * this.sheets.length);

        // Get the random value from the array
        const randomSheet = this.sheets[randomIndex];

        console.log(randomSheet)
        // Emit the random value
        this.$emit('query-response', randomSheet);
    }
  },
  emits: ['query-response']
};
</script>

<style>

.world-interactor {
  position: relative;
  display: flex;
  left: 150px;
  width: 15%;
  padding: 10px;
  margin-top: 30px;
  justify-content: center;
  flex-direction: column;
  background: white;
  box-shadow: 0 0 0 3px black
}

.world-interactor-title {
  text-align: center;
  font-size: 12;
  font-weight: bold;
  margin-top: 0px;
  margin-bottom: 10px;
}

.area-select {
  margin: 10px;
}

.area-source-select {
  margin: 10px;
}

.question-button {
position: absolute;
bottom: 20px;
right: 20px;
background-color: #444;
color: #fff;
border: none;
border-radius: 5px;
padding: 10px 20px;
font-size: 16px;
cursor: pointer;
}

.creature-image {
position: absolute;
bottom: 20px;
left: 20px;
width: 100px;
height: auto;
}
</style>