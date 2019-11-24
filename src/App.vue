<template>
  <div id="app">
    <h1>Tetris</h1>
    <game></game>
  </div>
</template>

<script>
import Game from "./components/Game.vue";

export default {
  name: "app",
  components: {
    Game
  }
};
</script>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
h1 {
  text-transform: uppercase;
}
</style>
