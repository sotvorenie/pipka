<script setup>
import {onMounted, ref, watch} from "vue";
import {createTimeline} from "animejs";

const props = defineProps({
  visibleDotsNumber: {
    type: Number,
    default: 0
  },
  textScale: {
    type: Number,
    default: 0
  },
  loadingTranslate: {
    type: Number,
    default: 300
  },
  fingerPosition: {
    type: Number,
    default: 0
  }
})

const fingerRef = ref(null)
let timeline = null

onMounted(() => {
  timeline = createTimeline({
    autoplay: false,
    easing: 'linear',
  })

  timeline.add(fingerRef.value, {
    translateX: [0, window.innerWidth <= 1920 ? 450 : 550],
    duration: 100,
  })
})

watch(
    () => props.fingerPosition,
    (newVal) => {
      if (timeline) {
        timeline.seek(newVal)
      }
    }
)

</script>

<template>

  <div class="donat h-100 w-100 flex-center flex-column position-fixed">
    <p class="donat__title" :style="{scale: textScale}">
      Загрузка<span v-if="visibleDotsNumber >= 1">.</span>
      <span v-if="visibleDotsNumber >= 2">.</span>
      <span v-if="visibleDotsNumber === 3">.</span>
    </p>

    <div class="donat__loading flex flex-align-center" :style="`transform: translateY(${loadingTranslate}%)`">
      <img class="donat__finger position-relative"
           src="/палец.webp"
           alt="палец"
           ref="fingerRef"
           draggable="false"
      />
      <div class="donat__donat position-relative">
        <img class="donat__donat-left" src="/пончик.webp" alt="пончик" draggable="false"/>
        <img class="donat__donat-right position-absolute" src="/пончик.webp" alt="пончик" draggable="false"/>
      </div>
    </div>
  </div>

</template>