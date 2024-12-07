<script setup>
import TaskItem from "@/components/CantineItem.vue";
import { defineProps } from "vue";

defineProps({
  list: { type: Array, required: true },
});
</script>

<template>
  <div class="container">
    <!-- <h2>Entrée</h2> -->
    <div v-for="(item, index) in list" :key="index">
      <TaskItem :task="item" />
    </div>
  </div>
</template>

<style scoped>
/* .container {
  display: flex;
  justify-content: center;
  align-items: center;
} */
</style>
