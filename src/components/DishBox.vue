<template>
  <div class="dish-box">
    <h4>{{ dish.title }}</h4>
    <p>{{ dish.description }}</p>
    <p>Votes : {{ dish.votes }}</p>
    <button @click="vote">Voter</button>
    <button @click="$emit('delete-dish')" class="btn del-btn">Supprimer</button>
  </div>
</template>

<script>
export default {
  props: {
    dish: {
      type: Object,
      required: true,
    },
  },
  methods: {
    vote() {
      this.$emit("vote-dish", this.dish);
    },
  },
};
</script>

<style scoped>
.dish-box {
  background-color: white;
  border: 1px solid #dee2e6;
  border-radius: 5px;
  padding: 15px;
  width: 250px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  display: flex;
  flex-direction: column;
  align-items: center;
}

.dish-box h4 {
  margin: 0;
  color: #495057;
}

.dish-box p {
  margin: 5px 0;
  color: #6c757d;
}

.dish-box button {
  margin-top: 10px;
  background-color: #007bff;
  color: white;
  border: none;
  padding: 5px 10px;
  border-radius: 3px;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

.dish-box button:hover {
  background-color: #0056b3;
}

.btn.del-btn {
  background-color: #dc3545;
  color: white;
  border: none;
  padding: 5px 10px;
  border-radius: 3px;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

.btn.del-btn:hover {
  background-color: #c82333;
}
</style>
