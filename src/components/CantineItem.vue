<script setup>
import { defineProps } from "vue";

defineProps({
  task: {
    type: { title: String, content: String },
    required: true,
  },
});
</script>

<template>
  <div class="item">
    <span>{{ task.title }}</span>
    <p>{{ task.content }}</p>
  </div>
</template>

<style scoped>
.item {
  padding: 10px 10px;
  border-radius: 15px;
  border: 1px solid black;
  width: fit-content;
  margin-top: 20px;
  margin-left: 10px;
  margin-right: 10px;
}

.item p {
  margin: 0;
  padding-top: 10px;
}
</style>
