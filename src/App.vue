<script setup lang="ts">
import PixelDrawExample from './PixelDrawExample.vue'
import type Example from './Example'
const examples: Example[] = [{ width: 50, height: 50 }]
</script>

<template>
  <ul>
    <li v-for="(example, i) in examples" :key="i">
      <PixelDrawExample :example="example" />
    </li>
  </ul>
</template>

<style scoped lang="scss">
ul {
  list-style: none;
  padding: 0;
  margin: 0;
  position: relative;
  width: 100%;
  display: grid;
  grid-template-columns: 1fr;
  li {
    padding: 10px;
    margin: 10px;
  }
}
</style>
