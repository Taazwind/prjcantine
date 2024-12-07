<template>
    <div>
      <h2>Menu de la Semaine</h2>
      <div v-for="(dishes, category) in menu" :key="category">
        <h3>{{ category }}</h3>
        <div v-if="dishes.length > 0">
          <DishCard v-for="(dish, index) in dishes" :key="index" :dish="dish" @vote="voteForDish(category, index)" />
        </div>
        <p v-else>Aucun plat disponible</p>
      </div>
    </div>
  </template>
  
  <script>
  import DishCard from './DishCard.vue';
  
  export default {
    components: { DishCard },
    props: ['menu'],
    methods: {
      voteForDish(category, index) {
        this.$emit('updateVotes', category, index);
      }
    }
  };
  </script>
  
  <style scoped>
  h2 {
    text-align: center;
    color: #333;
  }
  </style>
  