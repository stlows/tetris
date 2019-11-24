<template>
  <rect :width="cellSize" :height="cellSize" :x="x" :y="y" />
</template>

<script>
export default {
  props: {
    cellSize,
    x,
    y,
    type
  }
};
</script>

<style scoped>
.type1 {
  fill: yellow;
}
</style>