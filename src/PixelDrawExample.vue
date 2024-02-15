<script setup lang="ts">
import { onMounted, ref, watch, type Ref, reactive, defineEmits } from 'vue'
import type Example from './Example'
import MultiPath from './MultiPath.vue'
const props = defineProps<{
  example: Example
}>()

const emits = defineEmits(['changeResolution'])

const { width, height } = props.example
const pixels = ref([]) as Ref<number[][]>
const colors = ref<HTMLElement | null>(null)

const paths = ref<number[][][]>([[]])
const randomHexColorCode = () => {
  let n = (Math.random() * 0xfffff * 1000000).toString(16)
  return '#' + n.slice(0, 6) + 'FF'
}
const colorSample: string[] = []
for (let i = 0; i < 100; i++) {
  colorSample.push(randomHexColorCode())
}
const properties = reactive({
  dpr: window.devicePixelRatio,
  lineWidth: 1,
  scale: 10,
  imageRenderer: 'pixelated',
  opacity: 128,
  lineCap: 'square',
  lineJoin: 'miter',
  globalCompositeOperation: 'source-over'
})
const canvasWrapper = ref<HTMLElement | null>(null)

onMounted(() => {
  colors.value?.style.setProperty('grid-template-columns', `${getGridTemplateColumns(width)}`)
  animate()
})

const getGridTemplateColumns = (n: number) => {
  let str = ''
  for (let i = 0; i < n; i++) {
    str += '1fr '
  }
  return str
}

const draw = () => {
  if (canvasWrapper.value !== null) {
    canvasWrapper.value.innerHTML = ''
  }

  colors.value?.style.setProperty('grid-template-columns', `${getGridTemplateColumns(width)}`)

  const canvas = document.createElement('canvas')
  canvas.width = width
  canvas.height = height

  const ctx = canvas.getContext('2d')

  const pathCanvas = document.createElement('canvas')
  pathCanvas.style.setProperty('border', `1px solid #000`)
  pathCanvas.style.setProperty('width', `${(width / properties.dpr) * properties.scale}px`)
  pathCanvas.style.setProperty('height', `${(height / properties.dpr) * properties.scale}px`)

  pathCanvas.width = width * properties.scale
  pathCanvas.height = height * properties.scale

  const pathCtx = pathCanvas.getContext('2d')!
  pathCtx.lineCap = properties.lineCap as CanvasLineCap
  pathCtx.lineJoin = properties.lineJoin as CanvasLineJoin
  pathCtx.lineWidth = properties.lineWidth

  if (ctx) {
    ctx.globalAlpha = 1.0

    pixels.value = new Array(width * height).fill(0).map(() => [0, 0, 0, 0]) as number[][]
    paths.value.forEach((path, index) => {
      ctx.clearRect(0, 0, canvas.width, canvas.height)
      ctx.imageSmoothingEnabled = false
      ctx.lineWidth = properties.lineWidth
      const generatedColor = colorSample[index] as string
      ctx.fillStyle = generatedColor
      ctx.globalCompositeOperation = properties.globalCompositeOperation as GlobalCompositeOperation
      ctx.strokeStyle = generatedColor
      ctx.lineCap = properties.lineCap as CanvasLineCap
      ctx.lineJoin = properties.lineJoin as CanvasLineJoin
      ctx.beginPath()
      path.forEach((point, i) => {
        if (i === 0) {
          ctx.moveTo(point[0], point[1])
          pathCtx.moveTo(point[0] * properties.scale, point[1] * properties.scale)
        } else {
          ctx.lineTo(point[0], point[1])
          pathCtx.lineTo(point[0] * properties.scale, point[1] * properties.scale)
        }
      })
      pathCtx.closePath()
      ctx.closePath()
      if (
        path.length > 1 &&
        path[0][0] === path[path.length - 1][0] &&
        path[0][1] === path[path.length - 1][1]
      ) {
        ctx.fill()
        pathCtx.fill()
      }
      pathCtx.stroke()
      ctx.stroke()

      const data = ctx.getImageData(0, 0, width, height).data

      for (let i = 0; i < data.length / 4; i++) {
        let alpha = null
        if (data[i * 4 + 3] >= properties.opacity) {
          alpha = 255
        } else {
          alpha = 0
        }

        //generatedColor hex to rgb
        const r = parseInt(generatedColor.slice(1, 3), 16)
        const g = parseInt(generatedColor.slice(3, 5), 16)
        const b = parseInt(generatedColor.slice(5, 7), 16)

        if (data[i * 4 + 3] > 0 || data[i * 4] > 0 || data[i * 4 + 1] > 0 || data[i * 4 + 2] > 0) {
          pixels.value[i][0] = r
          pixels.value[i][1] = g
          pixels.value[i][2] = b
          pixels.value[i][3] = alpha
        }
      }
    })
  }
  canvas.style.setProperty('border', `1px solid #ddd`)
  canvas.style.setProperty('width', `${(width / properties.dpr) * properties.scale}px`)
  canvas.style.setProperty('height', `${(height / properties.dpr) * properties.scale}px`)
  canvas.style.setProperty('image-rendering', properties.imageRenderer)

  const clone = canvas.cloneNode(true) as HTMLCanvasElement
  const ctx2 = clone.getContext('2d')
  if (ctx2) {
    const imageData = ctx2.createImageData(width, height)
    const clonePixels = pixels.value.map((pixel) => [...pixel])
    const data = new Uint8ClampedArray(clonePixels.flat())
    imageData.data.set(data)
    ctx2.putImageData(imageData, 0, 0)
    clone.style.setProperty('border', `1px solid #000`)
    canvasWrapper.value?.appendChild(createElement(clone, 'Chosen pixels'))
    const pathsCanvas = createElement(pathCanvas, 'Polygon paths')
    pathsCanvas.style.setProperty('position', 'absolute')
    pathsCanvas.style.setProperty('left', `${(width / properties.dpr) * properties.scale + 10}px`)
    canvasWrapper.value?.appendChild(pathsCanvas)
  }
}

const createElement = (element: HTMLElement, text: string) => {
  const container = document.createElement('div')
  container.style.setProperty('text-align', 'center')
  container.style.setProperty('display', 'inline-block')
  container.style.setProperty('transition', 'all 0.3s ease')
  container.style.setProperty('margin', '0 0 10px 0')
  const textElement = document.createElement('div')
  textElement.style.setProperty('text-align', 'center')
  textElement.style.setProperty('font-size', '12px')
  textElement.style.setProperty('line-height', '80%')
  textElement.textContent = text
  container.appendChild(element)
  container.appendChild(textElement)
  return container
}

const animate = () => {
  draw()
}

const changeResolution = () => {
  const width = Number((document.getElementById('width') as HTMLInputElement).value)
  const height = Number((document.getElementById('height') as HTMLInputElement).value)
  if (width > 1 && height > 1 && width < 51 && height < 51) {
    emits('changeResolution', width, height)
    animate()
  } else {
    alert('Width and height must be between 2 and 50')
  }
}

watch(properties, () => {
  draw()
})

watch(
  paths,
  () => {
    draw()
  },
  { deep: true }
)

const mode = ref<'overlap' | 'normal'>('normal')
watch(mode, (newMode) => {
  if (canvasWrapper.value?.children?.length === 2) {
    const child = canvasWrapper.value.children[1]
    if (child instanceof HTMLElement) {
      if (newMode === 'overlap') {
        child.style.setProperty('left', '0')
      } else {
        child.style.setProperty('left', `${(width / properties.dpr) * properties.scale + 10}px`)
      }
    }
  }
})
</script>
<template>
  <div class="container">
    <div>
      <div style="margin-bottom: 10px">
        <div style="display: inline-block">Resolution</div>
        &nbsp;
        <input id="width" type="number" min="2" max="50" :value="width" /> X
        <input id="height" type="number" min="2" max="50" :value="height" />
        &nbsp;
        <button @click="changeResolution" style="min-width: 10px">Change</button>
      </div>
      <button v-if="mode === 'normal'" @click="mode = 'overlap'">
        Bitmap and polygon paths overlap
      </button>
      <button v-if="mode === 'overlap'" @click="mode = 'normal'">Back to original position</button>
      <div ref="canvasWrapper" class="canvasWrapper"></div>
      <ul>
        <li class="form">
          <div>devicePixelRatio:</div>
          <div>
            {{ properties.dpr }}
            <input type="range" v-model="properties.dpr" step="0.1" min="0.5" max="5" />
          </div>
        </li>
        <li class="form">
          <div>width:</div>
          <div>{{ width }} px</div>
        </li>
        <li class="form">
          <div>height:</div>
          <div>{{ height }} px</div>
        </li>
        <li class="form">
          <div>lineWidth:</div>
          <div>
            {{ properties.lineWidth }}

            <input type="range" v-model="properties.lineWidth" step="0.5" min="0" max="10" />
          </div>
        </li>
        <li class="form">
          <div>scale:</div>
          <div>
            {{ properties.scale }}

            <input type="range" v-model="properties.scale" step="0.5" min="1" max="20" />
          </div>
        </li>
        <li class="form">
          <div>image rendering</div>
          <div>
            <label>
              <input type="radio" v-model="properties.imageRenderer" value="auto" /> auto</label
            >
            <label>
              <input type="radio" v-model="properties.imageRenderer" value="crisp-edges" />
              crisp-edges</label
            >
            <label
              ><input type="radio" v-model="properties.imageRenderer" value="pixelated" />
              pixelated</label
            >
          </div>
        </li>
        <li class="form">
          <div>lineCap</div>
          <div>
            <label> <input type="radio" v-model="properties.lineCap" value="butt" /> butt</label>
            <label>
              <input type="radio" v-model="properties.lineCap" value="round" />
              round</label
            >
            <label><input type="radio" v-model="properties.lineCap" value="square" /> square</label>
          </div>
        </li>
        <li class="form">
          <div>lineJoin</div>
          <div>
            <label> <input type="radio" v-model="properties.lineJoin" value="bevel" /> bevel</label>
            <label>
              <input type="radio" v-model="properties.lineJoin" value="round" />
              round</label
            >
            <label><input type="radio" v-model="properties.lineJoin" value="miter" /> miter</label>
          </div>
        </li>
        <li class="form">
          <div>globalCompositeOperation</div>
          <div>
            <label
              v-for="(item, i) in [
                'source-over',
                'source-in',
                'source-out',
                'source-atop',
                'destination-over',
                'destination-in',
                'destination-out',
                'destination-atop',
                'lighter',
                'darker',
                'xor',
                'copy'
              ]"
              :key="i"
            >
              <input type="radio" v-model="properties.globalCompositeOperation" :value="item" />
              {{ item }}</label
            >
          </div>
        </li>
        <li class="form">
          <div>Opacity threshold:</div>
          <div>
            {{ properties.opacity }}

            <input type="range" v-model="properties.opacity" step="1" min="1" max="255" />
          </div>
        </li>
        <li>
          <div class="item">pixels:</div>
          <br />
          <div class="item" style="border: 1px solid #000; padding: 2px" ref="colors">
            <div
              v-for="(pixel, j) in pixels"
              :title="`rgba( ${pixel[0]}, ${pixel[1]}, ${pixel[2]}, ${pixel[3]} )`"
              :key="j"
              :style="{
                backgroundColor: `rgba(${pixel[0]}, ${pixel[1]}, ${pixel[2]}, ${(pixel[3] >= properties.opacity ? 255 : pixel[3]) / 255})`
              }"
              :class="pixel[3] === 0 ? '' : 'hasColor'"
            ></div>
          </div>
        </li>
      </ul>
    </div>
    <MultiPath v-model="paths" :height="height" :width="width" />
  </div>
</template>

<style scoped lang="scss">
div.container {
  display: grid;
  gap: 30px;
  grid-template-columns: 1fr 1fr;
  width: 100%;
  button {
    min-width: 300px;
    margin-bottom: 10px;
  }
}
div.canvasWrapper {
  position: relative;
}
ul {
  list-style: none;
  padding: 0;
  margin: 0;
  display: block;
  width: 100%;
  position: relative;
  li {
    position: relative;
    font-size: 13px;
    max-width: 100%;
    display: inline-block;

    &.form {
      display: grid;
      grid-template-columns: 1fr 2fr;
      text-align: left;
      div {
        padding: 5px;
      }
      label {
        display: inline-block;
        margin-right: 10px;
      }
      input[type='radio'] {
        margin-right: 0;
        margin-top: 5px;

        vertical-align: top;
      }
      input[type='range'] {
        width: 100%;
      }
    }

    div.item {
      position: relative;
      display: grid;
      gap: 2px;

      div {
        overflow: hidden;
        border: 1px solid gray;
        font-weight: bold;
        padding: 3px;
        font-size: 6px;
        transition: all 0.3s ease;

        &.hasColor {
          border: 1px solid #000;
        }

        &:hover {
          cursor: pointer;
          transform: scale(1.2);
        }

        div.color {
          border: none;
          margin: 0;
          padding: 0;
          line-height: 100%;
          text-align: center;
          font-weight: bolder;
          font-family: monospace;
        }
      }
    }
  }
}
</style>
