<template>
    <div class="creature-interactor">
      <div>
        <p class="creature-interactor-title">Creature Interactor</p>
      </div>

      <img v-if="selectedCreatureName" :src="getCreatureImage(selectedCreatureName)" alt="creature" />
  
      <select class="creature-select" v-model="selectedCreatureName">
        <option disabled value="">Select a Creature</option>
        <option v-for="creature in creatures" :key="creature.creature_name" :value="creature.creature_name">{{ creature.creature_name }}</option>
      </select>
  
      <select class="creature-question-select" v-model="selectedQuestion">
        <option disabled value="">Select a Question</option>
        <option v-for="question in questions" :key="question.id" :value="question.text">{{ question.text }}</option>
      </select>
  
      <button @click="generateSheet">Query</button>
    </div>
</template>

<script>
  import axios from 'axios';
  import { v4 as uuidv4 } from 'uuid';

  export default {
    name: 'CreatureInteractor',
    props: {},
    data() {
      return {
        creatures: [],
        questions: [
            {
            "id": "1",
            "text": "Gifts"
            },
            {
            "id": "2",
            "text": "Interactions"
            }
        ],
        selectedCreatureName: "Aurora",
        selectedQuestion: "Gifts",
        responseSheet: null
      };
    },
    methods: {
      generateUniqueId() {
          return uuidv4();
      },
      getCreatureImage(creature) {
        // here you should return the URL of the image based on the creature name
        if(creature.creature_name) {
            return("creature.jpeg");        
        }
        else {
            return("creature.jpeg");
        }
      },
      generateSheet() {
        var baseUrl = "http://localhost:8000"
        var endpoint = '';
        var metadataEndpoint = '';

        switch(this.selectedQuestion) {
            case 'Gifts':
                endpoint = '/creatures/gifts';
                metadataEndpoint = '/creatures/gifts/metadata';
                break;
            case 'Interactions':
                endpoint = '/creatures/interactions';
                metadataEndpoint = '/creatures/interactions/metadata';
                break;
        }

        var baseQuery = "?creatureName=" + this.selectedCreatureName
        var query = baseUrl + endpoint + baseQuery;

        var newSheet = {};
        // Fetch metadata from the server
        axios.get(baseUrl + metadataEndpoint)
        .then(response => {
            // create sheet object
            newSheet.id = this.generateUniqueId(); // implement this function to generate a unique ID
            newSheet.title = `${this.selectedCreatureName}'s ${this.selectedQuestion}`;
            newSheet.type = "Creatures Data"; // or other type depending on the selected question
            newSheet.endpoint = query; 
            newSheet.icon = "icon.png";
            newSheet.metadata = response.data; // use the returned metadata

            // Emit the random value
            this.$emit('query-response', newSheet);
        });
      }
    },
    created() {
        axios.get('http://localhost:8000/creatures/list').then(response => {
          this.creatures = response.data;
        });
    },
    emits: ['query-response']
  };
</script>

<style>

.creature-interactor {
    position: relative;
    display: flex;
    left: 150px;
    margin-top: 30px;
    width: 15%;
    padding: 10px;
    justify-content: center;
    flex-direction: column;
    background: white;
    box-shadow: 0 0 0 3px black
}

.creature-interactor-title {
    text-align: center;
    font-size: 12;
    font-weight: bold;
    margin-top: 0px;
    margin-bottom: 10px;
}

.creature-select {
    margin: 10px;
}

.creature-question-select {
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