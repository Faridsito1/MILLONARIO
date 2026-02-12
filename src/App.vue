<!-- src/App.vue -->
<template>
  <q-layout view="hHh lpR fFf">
    <q-page-container>
      <GameScreen />
    </q-page-container>
  </q-layout>
</template>

<script setup>
import GameScreen from './components/GameScreen.vue'
</script>

<style>
body {
  margin: 0;
  padding: 0;
  font-family: 'Roboto', sans-serif;
}

/* Fuentes adicionales si es necesario */
@import url('https://fonts.googleapis.com/css2?family=Roboto:wght@400;500;700;900&display=swap');
</style>