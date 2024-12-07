<script setup>
import { ref } from "vue";
import { defineEmits } from "vue";

const emit = defineEmits(["add"]);



const title = ref("");
let numberMeal = 0;
const content = ref("");

const handleAdd = () => {
  if (title.value === "" || content.value === "") {
    return;
  }

  const newTask = {
    title: title.value,
    content: content.value,
  };

  emit("add", newTask);

  numberMeal += 1;

  title.value = "";
  content.value = "";
};
</script>

<template>
  <div>
    <input type="text" v-model="title" placeholder="Nom du plat" />
    <select id="choice" name="choice" v-model="content">
      <option value="entree">Entrée</option>
      <option value="plat">Plat</option>
      <option value="dessert">Dessert</option>
    </select>
    <button @click.prevent="handleAdd">Ajouter le plat au menu</button>
    <p>Nombre de plat disponible : {{ numberMeal }}</p>
  </div>
</template>

<style scoped></style>
