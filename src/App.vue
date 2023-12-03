<script setup lang="ts">
import CChatWidget from './components/chatwidget/CChatWidget.vue'
</script>

<template>
  <div class="w-screen h-screen bg-purple-50 relative overflow-hidden">
    <CChatWidget class="absolute bottom-10 right-10" />
  </div>
</template>

<style scoped>
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
