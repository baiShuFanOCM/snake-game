<template>
  <div class="app">
    <h1>贪吃🌰的🐍</h1>
    <SnakeGame />
  </div>
</template>

<script setup lang="ts">
import SnakeGame from './components/snake_game.vue'
</script>

<style>
.app {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 20px;
  font-family: 'Arial Rounded MT Bold', Arial, sans-serif;
}

h1 {
  color: #4a5568;
  margin-bottom: 20px;
}
</style>
