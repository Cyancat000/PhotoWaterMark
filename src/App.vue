<script setup lang="ts">
import CanvasEditor from './components/CanvasEditor.vue';
</script>

<template>
  <div>
    <CanvasEditor></CanvasEditor>
  </div>
</template>

<style scoped>
body{
  background-color: #FFFFFF;
}
.logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
  transition: filter 300ms;
}
.logo:hover {
  filter: drop-shadow(0 0 2em #646cffaa);
}
.logo.vue:hover {
  filter: drop-shadow(0 0 2em #42b883aa);
}
</style>
