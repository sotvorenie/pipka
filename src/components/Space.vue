<script setup>
import {onMounted, ref} from "vue";
import {animate, createTimeline} from "animejs";

import SpaceBackground from "@/components/SpaceBackground.vue";

const spaceRef = ref(null)

onMounted(() => {
  const timeline = createTimeline({
    defaults: {
      easing: "easeOutExpo",
    }
  })

  timeline.add('.space__background', {
        duration: 1000,
        opacity: [0, 1],
  }).add('.space__title', {
    duration: 1000,
    opacity: [0, 1],
  }, '-=500').play()


  animate('.space__img-container', {
    duration: 100000,
    translateX: [-100, 200, -100],
    translateY: [-200, spaceRef.value.clientHeight - 180, -200],
    loop: true,
    direction: 'alternate',
    easing: 'easeInOutSine',
  })

  animate('.space__fox', {
    duration: 100000,
    rotate: [0, 720, 0],
    loop: true,
    alternate: true,
    easing: 'linear',
  })
})
</script>

<template>

  <div class="space flex-center h-100" ref="spaceRef">
    <div class="space__background">
      <SpaceBackground/>
    </div>

    <div class="space__img-container position-absolute z-1">
      <img src="/space-fox.webp"
           class="space__fox"
           alt="space-fox"
           draggable="false"
      />
    </div>

    <h1 class="space__title"><span class="space__span text-upper">лис</span>тай ниже</h1>
  </div>

</template>