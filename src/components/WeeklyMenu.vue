<template>
  <div class="menu">
    <h2>{{ menuTitle }}</h2>
    <div v-for="(dishes, category) in menu" :key="category">
      <h3>{{ category }}</h3>
      <div v-if="dishes.length > 0">
        <DishCard v-for="dish in dishes" :key="dish.title" :dish="dish" />
      </div>
      <p v-else>Aucun plat disponible</p>
    </div>
  </div>
</template>

<script>
import DishCard from "./DishCard.vue";

export default {
  components: {
    DishCard
  },
  props: {
    menu: {
      type: Object,
      required: true
    },
    menuTitle: {
      type: String,
      default: "Menu"
    }
  }
};
</script>

<style scoped>
.menu {
  margin-top: 20px;
}
h2 {
  text-align: center;
}
</style>
