<template>
  <div>
    <tetris-grid :w="10" :h="15"></tetris-grid>
    <all-pieces></all-pieces>
  </div>
</template>

<script>
import TetrisGrid from "./TetrisGrid";
import AllPieces from "./AllPieces";
export default {
  components: {
    TetrisGrid,
    AllPieces
  }
};
</script>

<style>
</style>