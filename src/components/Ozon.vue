<script setup>
import {onMounted, ref, watch} from "vue";
import {createTimeline} from "animejs";

const props = defineProps({
  scrollProgress: {
    type: Number,
    default: 0
  }
})

const ozonRef = ref(null)
const leftImg = ref(null)
const rightImg = ref(null)
const cardRef = ref(null)
const btnRef = ref(null)

let timeline = null
let imgTimeline = null
let cardTimeline = null
let btnTimeline = null

const imgVisible = ref(false)
const qrVisible = ref(false)
const btnVisible = ref(false)
const listVisible = ref(false)

const copyArr = ref([])
const handleCopy = () => {
  const id = Date.now()
  copyArr.value.push({id, text: 'Просто скачай фото)'})

  setTimeout(() => {
    const index = copyArr.value.findIndex(i => i.id === id)
    if (index !== -1) {
      copyArr.value.splice(index, 1)
    }
  }, 3000)
}

onMounted(() => {
  timeline = createTimeline({
    autoplay: false,
    easing: 'linear',
  })
  imgTimeline = createTimeline({
    autoplay: false,
    easing: 'linear',
  })
  cardTimeline = createTimeline({
    autoplay: false,
    easing: 'linear',
  })
  btnTimeline = createTimeline({
    autoplay: false,
    easing: 'linear',
  })


  timeline.add(ozonRef.value, {
    translateY: [window.innerHeight / 2 + 100, 0],
    duration: 50,
  })
  timeline.add(ozonRef.value, {
    borderRadius: ['52%', '0%'],
    scale: [1, 15],
    duration: 50
  })

  imgTimeline.add(leftImg.value, {
    left: ['-100%', '0'],
    opacity: [0, 1],
    duration: 30
  })
  imgTimeline.add(rightImg.value, {
    right: ['-100%', '0'],
    opacity: [0, 1],
    duration: 30
  }, '-=30')

  cardTimeline.add(cardRef.value, {
    scale: [0, 1],
    rotateY: [-180, 0],
    rotateZ: [15, 0],
    translateZ: [120, 120],
    duration: 65
  })

  btnTimeline.add(btnRef.value, {
    translateX: [200, 0],
    translateY: ['50vh', 0],
    scale: [0, 1],
    duration: 20,
    easing: 'easeOutQuad'
  })
})

watch(
    () => props.scrollProgress,
    (newVal) => {
      if (timeline && ozonRef.value) {
        timeline.seek(newVal)

        imgVisible.value = newVal > 70
        qrVisible.value = newVal > 35
        btnVisible.value = newVal > 70
        listVisible.value = newVal > 90

        imgTimeline.seek(newVal - 70)
        cardTimeline.seek(newVal - 35)
        btnTimeline.seek(newVal - 70)
      }
    }
)
</script>

<template>

  <div class="ozon position-fixed h-100 w-100">
    <div class="ozon__content" ref="ozonRef" />

    <TransitionGroup tag="div"
                     class="ozon__list position-absolute flex flex-column"
                     name="list"
                     v-show="listVisible"
    >
      <div class="ozon__item"
           v-for="item in copyArr"
           :key="item.id"
      >
        {{item.text}}
      </div>
    </TransitionGroup>

    <button class="ozon__copy position-absolute text-w600"
            type="button"
            v-show="btnVisible"
            ref="btnRef"
            @click="handleCopy"
    >скопировать</button>

    <div class="ozon__card" v-show="qrVisible" ref="cardRef">
      <img class="w-100 h-100" src="/qr.webp" alt="qr" draggable="false"/>
      <div class="backface position-absolute w-100 h-100"></div>
    </div>

    <img v-show="imgVisible"
         class="ozon__img left position-absolute h-100"
         src="/ozon.webp"
         alt="ozon"
         ref="leftImg"
         draggable="false"
    />
    <img v-show="imgVisible"
         class="ozon__img right position-absolute h-100"
         src="/ozon.webp"
         alt="ozon"
         ref="rightImg"
         draggable="false"
    />
  </div>

</template>