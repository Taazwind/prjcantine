<template>
    <div class="day-buttons">
      <button v-for="day in days" :key="day" @click="$emit('select-day', day)">
        {{ day }}
      </button>
    </div>
  </template>
  
  <script>
  export default {
    props: {
      days: {
        type: Array,
        required: true,
      },
    },
  };
  </script>
  
  <style scoped>
  .day-buttons {
    display: flex;
    justify-content: center;
    margin-bottom: 20px;
    gap: 10px;
  }
  
  .day-buttons button {
    padding: 10px 15px;
    background-color: #007bff;
    color: white;
    border: none;
    border-radius: 5px;
    cursor: pointer;
  }
  
  .day-buttons button:hover {
    background-color: #0056b3;
  }
  </style>