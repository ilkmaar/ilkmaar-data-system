<template>
  <div class="world-interactor">
    <div>
      <p class="world-interactor-title">World Interactor</p>
    </div>

    <select class="area-select" v-model="selectedArea">
      <option disabled value="">Select an Area</option>
      <option v-for="area in areas" :key="area.area" :value="area">{{ area.area }}</option>
    </select>

    <select class="area-source-select" v-model="selectedSource" v-if="selectedArea">
      <option disabled value="">Select a Source</option>
      <option v-for="source in selectedArea.dataSources" :key="source.source" :value="source">{{ source.source }}</option>
    </select>

    <button @click="generateSheet">Query</button>
  </div>
</template>

<script>
import axios from 'axios';
import { v4 as uuidv4 } from 'uuid';

export default {
  name: 'WorldInteractor',
  props: {
    player: {
      type: String,
      required: false,
    }
  },
  data() {
    return {
      areas: [],
      selectedArea: null,
      selectedSource: null,
      responseSheet: null
    };
  },
  methods: {
    getWorldAreaImage(selectedArea) {
      if(selectedArea) {
        return("creature.jpeg");        
      } else {
        return("creature.jpeg");
      }
    },
    generateUniqueId() {
      return uuidv4();
    },
    generateSheet() {
      if(this.selectedArea.area !== "Misc") {
            this.query();
            return;
        }
        var baseUrl = "http://localhost:8000"        
        var endpoint = this.selectedSource.endpoint;
        var metadataEndpoint = endpoint + "/metadata";
        var baseQuery = endpoint.startsWith("/players") ? `?playerName=${this.player}` : "";
        var query = baseUrl + endpoint + baseQuery;
        var playerEndpoints = ["/foraging", "/gifting", "/crafting", "/trash"]

        var sendPlayerName = playerEndpoints.includes(endpoint)

        var newSheet = {};
        axios.get(baseUrl + metadataEndpoint)
        .then(response => {            
            newSheet.id = this.generateUniqueId();
            newSheet.title = `${sendPlayerName ? this.player + "'s " : ""}${this.selectedSource.source}`;
            
            let typeName = endpoint.split('/')[1];
            typeName = typeName.charAt(0).toUpperCase() + typeName.slice(1);
            newSheet.type = typeName + " Data";

            newSheet.endpoint = query;
            newSheet.icon = "icon.png";
            newSheet.metadata = response.data;
            this.$emit('query-response', newSheet);
        });
    }
  },
  created() {
      axios.get('http://localhost:8000/meta/areas').then(response => {
        this.areas = response.data;
        this.selectedArea = this.areas[8]
        this.selectedSource = this.selectedArea.dataSources[0]
      });
  },
  watch: {
    selectedArea(newArea) {
      this.selectedSource = newArea.dataSources[0];
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