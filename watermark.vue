<script lang="ts">
import {
  defineComponent,
  onMounted,
  computed,
  ref,
  PropType,
} from '@nuxtjs/composition-api'
import { TranslateResult } from 'vue-i18n/types'

export default defineComponent({
  name: 'Watermark',
  props: {
    fz: {
      type: Number,
      default: 20,
    },
    gap: {
      type: Array as PropType<number[]>,
      default: () => [20, 0],
    },
    color: {
      type: String,
      default: 'rgba(0, 0, 0, 1)',
    },
    text: {
      type: String,
      default: 'SOLARGROUP',
    },
  },
  setup({ fz, color, text, gap }) {
    const width = ref(242)
    const height = ref(100)

    function getTextDimensions(text: TranslateResult) {
      const svg = document.createElementNS('http://www.w3.org/2000/svg', 'svg')
      const textElement = document.createElementNS(
        'http://www.w3.org/2000/svg',
        'text'
      )

      textElement.setAttribute('font-size', String(fz))
      textElement.setAttribute('font-family', 'Montserrat')
      textElement.textContent = String(text)

      svg.appendChild(textElement)
      document.body.appendChild(svg)
      const bbox = textElement.getBBox()
      document.body.removeChild(svg)

      return {
        width: bbox.width,
        height: bbox.height,
      }
    }

    const bg = computed(() => {
      // TODO Добавить поддержку шрифтов
      const svgString = `<svg xmlns='http://www.w3.org/2000/svg' width='${width.value}' height='${height.value}'><text x='0' y='${fz}' font-size='${fz}' font-family="Montserrat" fill='${color}'>${text}</text></svg>`
      const encodedSVG = `data:image/svg+xml,${encodeURIComponent(svgString)}`
      return {
        encoded: `background: url("${encodedSVG}") repeat`,
        svg: svgString,
      }
    })

    onMounted(() => {
      const wh = getTextDimensions(text)
      width.value = Math.round(wh.width) // + gap[0]
      height.value = Math.round(wh.height * 4) // + gap[1]
    })
    return {
      bg,
      width,
      height,
    }
  },
})
</script>

<template>
  <div class="watermark-body absolute">
    <div class="child1" :style="bg.encoded"></div>
    <div class="child2" :style="bg.encoded"></div>
  </div>
</template>

<style scoped lang="postcss">
.watermark-body {
  overflow: hidden;
  inset: 0;
}
.child1 {
  @apply absolute;
  rotate: -45deg;
  width: 100vmax;
  height: 100vmax;
  translate: -20% -50%;
  background-size: auto;
}
.child2 {
  @apply absolute;
  rotate: -45deg;
  width: 100vmax;
  height: 100vmax;
  /* Временное решение, потом надо будет переработать расчёт смещения, для удобства */
  translate: calc(68px + -20%) -50%;
  background-size: auto;
}
</style>
