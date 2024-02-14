<template>
  <div class="container">
    <div class="title">Paths</div>
    <button @click="addPath">Add path</button>
    <ul v-if="paths">
      <li v-for="(path, k) in paths" :key="k">
        <div class="pixel">
          <div v-for="i in height" :key="i" class="row">
            <div
              v-for="j in width"
              :key="j"
              class="col"
              :class="
                paths[k] && paths[k].find((d) => d[0] === j - 1 && d[1] === i - 1)
                  ? 'active'
                  : 'inactive'
              "
              @click="() => addCoordinate(k, j - 1, i - 1)"
            >
              {{
                paths[k] && paths[k].find((d) => d[0] === j - 1 && d[1] === i - 1)
                  ? paths[k].findIndex((d) => d[0] === j - 1 && d[1] === i - 1) + 1
                  : ''
              }}
            </div>
          </div>
          <button
            @click="
              () => {
                removePath(k)
              }
            "
          >
            Remove
          </button>
        </div>
      </li>
    </ul>
  </div>
</template>
<script setup lang="ts">
const paths = defineModel<number[][][]>()
defineProps<{
  width: number
  height: number
}>()
const addPath = () => {
  paths.value?.push([])
}
const removePath = (index: number) => {
  paths.value?.splice(index, 1)
}
const addCoordinate = (index: number, x: number, y: number) => {
  if (paths.value) {
    if (paths.value[index]) {
      paths.value[index].push([x, y])
    } else {
      paths.value[index] = [[x, y]]
    }
  }
}
</script>
<style scoped lang="scss">
.container {
  display: block !important;
}
.title {
  font-weight: bold;
}

.code {
  font-size: 9px;
  background-color: #f8f8f8;
  font-weight: bolder;
  font-family: monospace;
  width: 100%;
  padding: 5px;
}

ul {
  list-style: none;
  margin: 0;
  padding: 0;
  li {
    margin-top: 20px;
    button {
      margin-top: 10px;
      display: inline-block;
    }
    .row {
      margin: 0;
      padding: 0;
      height: 10px;
    }

    .col {
      overflow: hidden;
      display: inline-block;
      font-size: 5px;
      font-weight: bold;
      text-align: center;
      width: 10px;
      height: 10px;
      border: 1px solid #000;
      &:hover {
        background-color: #000;
        cursor: pointer;
      }
      &.active {
        background-color: black;
        color: #fff;
      }
    }
  }
}
</style>
