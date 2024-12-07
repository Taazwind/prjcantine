<template>
    <div class="dish-card">
      <h3>{{ dish.title }}</h3>
      <p>{{ dish.description }}</p>
      <p>Votes : {{ dish.votes }}</p>
      <button @click="vote">Voter</button>
    </div>
  </template>
  
  <script>
  export default {
    props: ['dish'],
    methods: {
      vote() {
        this.$emit('vote');
      }
    }
  };
  </script>
  
  <style scoped>
  .dish-card {
    border: 1px solid #ccc;
    padding: 10px;
    margin: 5px;
    border-radius: 5px;
  }
  button {
    background-color: #007bff;
    color: white;
    border: none;
    padding: 5px 10px;
    cursor: pointer;
    border-radius: 3px;
  }
  button:hover {
    background-color: #0056b3;
  }
  </style>
  