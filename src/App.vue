<script setup>
import {nextTick, onMounted, ref} from "vue";
import {createTimeline} from "animejs";

import {imgList} from "@/data/imgList.js";

import Space from "@/components/Space.vue";
import Donat from "@/components/Donat.vue";
import Ozon from "@/components/Ozon.vue";
import Forest from "@/components/Forest.vue";
import Empty from "@/components/Empty.vue";
import About from "@/components/About.vue";
import Music from "@/components/Music.vue";
import Loading from "@/components/Loading.vue";

const mainWhiteRef = ref(null)

const donatVisible = ref(false)
const donatDotsNumber = ref(0)
const donatTextScale = ref(0)
const donatLoadingTranslate = ref(300)
const donateFingerPosition = ref(0)

const forestVisible = ref(false)
const forestProgress = ref(0)

const ozonVisible = ref(false)
const ozonProgress = ref(0)

const windowHeight = ref(0)

let ticking = false
const handleScroll = () => {
  const scrollY = window.scrollY

  const whiteBgMaxScroll = windowHeight.value * 2
  const whiteBgProgress = Math.min(scrollY / whiteBgMaxScroll, 1)
  timeline.seek(whiteBgProgress * 1000)

  donatVisible.value = whiteBgProgress > 0.5
  donatTextScale.value = whiteBgProgress
  donatLoadingTranslate.value = 300 - (300 * whiteBgProgress)
  if (scrollY > whiteBgMaxScroll && scrollY < windowHeight.value * 4) {
    const donatMaxScroll = windowHeight.value * 2
    const donatScroll = scrollY - whiteBgMaxScroll
    const donatProgress = Math.min(donatScroll / donatMaxScroll, 1)

    if (donatProgress < 0.33) donatDotsNumber.value = 0
    else if (donatProgress < 0.66) donatDotsNumber.value = 1
    else if (donatProgress < 0.99) donatDotsNumber.value = 2
    else donatDotsNumber.value = 3

    const part = Math.floor(donatProgress / 0.25)
    const fingerPercent = (donatProgress % 0.25) / 0.25 * 100
    if (part % 2 === 0) {
      donateFingerPosition.value = fingerPercent
    } else {
      donateFingerPosition.value = 100 - fingerPercent
    }
  } else {
    donatDotsNumber.value = scrollY > whiteBgMaxScroll ? 3 : 0
    donateFingerPosition.value = 0
  }

  const forestStartScroll = windowHeight.value * 4
  const forestMaxHeight = windowHeight.value * 3
  if (scrollY > forestStartScroll) {
    forestVisible.value = true

    const forestScroll = scrollY - forestStartScroll
    forestProgress.value = Math.min(forestScroll / forestMaxHeight, 1) * 100
  } else {
    forestVisible.value = false
  }

  const ozonStartScroll = windowHeight.value * 7
  const ozonMaxHeight = windowHeight.value * 2
  if (scrollY > ozonStartScroll) {
    ozonVisible.value = true

    const ozonScroll = scrollY - ozonStartScroll
    ozonProgress.value = Math.min(ozonScroll / ozonMaxHeight, 1) * 100
  } else {
    ozonVisible.value = false
  }
}
let timeline = null

const emptyVisible = ref(false)
const checkWidth = () => {
  emptyVisible.value = window.innerWidth < 1920
}

const visibleAbout = ref(false)
const visibleMusic = ref(false)

const isLoading = ref(true)
onMounted(async() => {
  const preloadImages = async () => {
    return new Promise((resolve) => {
      let loadedCount = 0
      const total = imgList.length

      if (total === 0) resolve()

      imgList.forEach((src) => {
        const img = new Image()
        img.src = src

        img.onload = img.onerror = () => {
          loadedCount++

          if (loadedCount === total) {
            setTimeout(() => {
              resolve()
            }, 500)
          }
        }
      })
    })
  }

  await preloadImages()
  isLoading.value = false

  await nextTick()

 timeline = createTimeline({
    autoplay: false,
    easing: 'linear',
  })

  timeline.add(mainWhiteRef.value, {
    width: ['30vw', '100vw'],
    height: ['0vh', '100vh'],
    bottom: ['-10%', '0%'],
    borderRadius: ['100px 100px 0 0', '0px 0px 0 0'],
    duration: 1000,
  })

  windowHeight.value = window.innerHeight
  checkWidth()

  window.addEventListener('scroll', () => {
    if (!ticking) {
      requestAnimationFrame(() => {
        handleScroll()
        ticking = false
      })
    }
    ticking = true
  })

  window.addEventListener('resize', () => {
    windowHeight.value = window.innerHeight
    checkWidth()
  })

  document.addEventListener('contextmenu', (event) => {
    // if (!event.target.closest('.ozon__card')) {
    //   event.preventDefault()
    // }
  })

  setTimeout(() => {
    visibleAbout.value = true
    visibleMusic.value = true
  }, 300)
})

const closeMusic = () => {
  setTimeout(() => {
    visibleAbout.value = true
  }, 700)
}
const closeAbout = () => {
  setTimeout(() => {
    visibleMusic.value = true
  }, 300)
}
</script>

<template>

  <Loading v-if="isLoading"/>

  <Empty class="position-fixed" v-else-if="!isLoading && emptyVisible"/>

  <main class="main position-relative" v-else>
    <div class="main__sticky h-100-vh position-sticky overflow-hidden">

      <Transition name="fade">
        <About v-show="visibleAbout"
               @open="visibleMusic = false"
               @close="closeAbout"
        />
      </Transition>

      <KeepAlive>
        <Transition name="fade">
          <Music v-show="visibleMusic"
                 @open="visibleAbout = false"
                 @close="closeMusic"
          />
        </Transition>
      </KeepAlive>

      <Space class="z-10"/>

      <div class="main__white z-100" ref="mainWhiteRef"></div>

      <Donat class="z-1000"
             v-if="donatVisible"
             :visible-dots-number="donatDotsNumber"
             :text-scale="donatTextScale"
             :loading-translate="donatLoadingTranslate"
             :finger-position="donateFingerPosition"
      />

      <Forest v-show="forestVisible" :scroll="forestProgress"/>

      <Ozon v-show="ozonVisible"
            :scroll-progress="ozonProgress"
      />

    </div>
  </main>

</template>