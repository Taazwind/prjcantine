<template>
    <form @submit.prevent="submitForm">
      <input v-model="newDish.title" placeholder="Nom du plat" />
      <input v-model="newDish.description" placeholder="Description" />
      <select v-model="newDish.category">
        <option>Entrée</option>
        <option>Plat</option>
        <option>Dessert</option>
      </select>
      <button type="submit">Ajouter</button>
    </form>
  </template>
  
  <script>
  export default {
    data() {
      return {
        newDish: {
          title: "",
          description: "",
          category: "Entrée",
        },
      };
    },
    methods: {
      submitForm() {
        this.$emit("add-dish", { ...this.newDish });
        this.newDish.title = "";
        this.newDish.description = "";
        this.newDish.category = "Entrée";
      },
    },
  };
  </script>
  
  <style scoped>
  form {
    display: flex;
    justify-content: center;
    margin-bottom: 20px;
  }
  
  input,
  select {
    margin-right: 10px;
    padding: 5px;
    border: 1px solid #ced4da;
    border-radius: 3px;
  }
  
  button {
    background-color: #28a745;
    color: white;
    border: none;
    padding: 5px 10px;
    cursor: pointer;
    border-radius: 3px;
  }
  
  button:hover {
    background-color: #218838;
  }
  </style>