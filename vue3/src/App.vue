<template>
  <HomeView />
</template>

<script setup lang="ts">
import HomeView from './components/HomeView.vue';
</script>

<style>
html {
  background-color: #000;
  color: #fff;
  font-size: 16px;
}

html,
body,
#app,
.container {
  height: 100%;
  width: 100%;
}

body {
  margin: 0;
  font-family: Tahoma, sans-serif;
}
</style>
