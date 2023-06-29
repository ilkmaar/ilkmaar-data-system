<template>
    <div class="creature-interactor">
      <div>
        <p class="creature-interactor-title">Creature Interactor</p>
      </div>

      <img v-if="selectedCreature" :src="getCreatureImage(selectedCreature)" alt="creature" />
  
      <select class="creature-select" v-model="selectedCreature">
        <option disabled value="">Select a Creature</option>
        <option v-for="creature in creatures" :key="creature.creature_name" :value="creature.creature_name">{{ creature.creature_name }}</option>
      </select>
  
      <select class="creature-question-select" v-model="selectedQuestion">
        <option disabled value="">Select a Question</option>
        <option v-for="question in questions" :key="question.id" :value="question.id">{{ question.text }}</option>
      </select>
  
      <button @click="query">Query</button>
    </div>
</template>

<script>
  import axios from 'axios';

  export default {
    name: 'CreatureInteractor',
    props: {
        sheets: {
            type: Array,
            required: false,
        },
    },
    data() {
      return {
        creatures: [],
        questions: [
            {
            "id": "1",
            "text": "Health?"
            },
            {
            "id": "2",
            "text": "Locations?"
            }
        ],
        selectedCreature: "Aurora",
        selectedQuestion: "1",
        responseSheet: null
      };
    },
    methods: {
      getCreatureImage(creature) {
        // here you should return the URL of the image based on the creature name
        if(creature) {
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

        // Emit the random value
        this.$emit('query-response', randomSheet);
      }
    },
    created() {
      axios.get('http://localhost:8000/creatures').then(response => {
        this.creatures = response.data;
        console.log(this.creatures)
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