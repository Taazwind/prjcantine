<template>
    <div>
      <h2>Menu du {{ selectedDay }}</h2>
      <div v-for="(dishes, category) in customMenu[selectedDay]" :key="category">
        <h3>{{ category }}</h3>
        <div class="dish-container" v-if="dishes.length > 0">
          <DishBox
            v-for="dish in dishes"
            :key="dish.title"
            :dish="dish"
            @vote-dish="voteDish"
            @delete-dish="deleteDish(category, dish)"
          />
        </div>
        <p v-else>Aucun plat disponible</p>
      </div>
    </div>
  </template>
  
  <script>
  import DishBox from "./DishBox.vue";
  
  export default {
    components: {
      DishBox,
    },
    props: {
      selectedDay: {
        type: String,
        required: true,
      },
      customMenu: {
        type: Object,
        required: true,
      },
    },
    methods: {
      voteDish(dish) {
        dish.votes++;
      },
      deleteDish(category, dish) {
        this.$emit("delete-dish", category, dish);
      },
    },
  };
  </script>
  
  <style scoped>
  .dish-container {
    display: flex;
    flex-wrap: wrap;
    gap: 20px;
    justify-content: center;
    margin: 20px 0;
  }
  </style>