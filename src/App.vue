<script setup lang="ts">
import PixelDrawExample from './PixelDrawExample.vue'
import type Example from './Example'
import { ref } from 'vue'
const examples: Example[] = [{ width: 30, height: 30 }]
const changeResolution = (index: number, width: number, height: number) => {
  examples[index].width = width
  examples[index].height = height
  refreshKey.value += examples.length
}
const refreshKey = ref(0)
</script>

<template>
  <ul>
    <li v-for="(example, i) in examples" :key="i">
      <PixelDrawExample
        :example="example"
        :key="refreshKey + i"
        @change-resolution="(width, height) => changeResolution(i, width, height)"
      />
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
