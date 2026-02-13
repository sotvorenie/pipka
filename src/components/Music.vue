<script setup>
import {computed, nextTick, onMounted, ref, watch} from "vue";
import {createTimeline} from "animejs";

import {menuList, list} from "@/data/music.js";

const emits = defineEmits(['open', 'close']);

const openRef = ref(null)
const iconVisible = ref(true)
let timeline = null
const visible = ref(false)

const handleOpen = () => {
  timeline.seek(1000)
  openRef.value.classList.add("is-open")
  iconVisible.value = false
  emits('open')

  setTimeout(() => {
    visible.value = true
  }, 400)
}
const handleClose = () => {
  visible.value = false
  emits('close')
  timeline.seek(0)
  openRef.value.classList.remove("is-open")

  setTimeout(() => {
    iconVisible.value = true
  }, 800)
}

onMounted(() => {
  timeline = createTimeline({
    autoplay: false,
    defaults: {
      autoplay: false,
      easing: "easeOutExpo",
    }
  })

  timeline.add(openRef.value, {
    scale: [1, 80],
    duration: 1000,
  })
})

const canvasRef = ref(null);
const drawVignette = () => {
  const canvas = canvasRef.value;
  if (!canvas) return;
  const ctx = canvas.getContext('2d');

  canvas.width = window.innerWidth;
  canvas.height = window.innerHeight;
  const w = canvas.width;
  const h = canvas.height;

  // 1. Полная заливка черным
  ctx.fillStyle = 'black';
  ctx.fillRect(0, 0, w, h);

  // 2. Вырезаем маленькое аккуратное пятно
  ctx.save();
  ctx.globalCompositeOperation = 'destination-out';

  // Центр пятна еще ближе к углу
  ctx.translate(w * 0.9, h * 0.12);

  // Уменьшаем масштаб эллипса, чтобы он не разваливался на пол-экрана
  ctx.scale(1.2, 1);

  // Третий параметр (радиус) делаем меньше — например, w * 0.4 вместо 0.9
  const radius = w * 0.65;
  const gradient = ctx.createRadialGradient(0, 0, 0, 0, 0, radius);

  // Сдвигаем стопы: прозрачность быстро исчезает
  gradient.addColorStop(0, 'rgba(0, 0, 0, 1)');      // В самом центре - дырка
  gradient.addColorStop(0.1, 'rgba(0, 0, 0, 0.9)');  // Очень резкий переход к тени
  gradient.addColorStop(0.4, 'rgba(0, 0, 0, 0.3)');  // Здесь уже почти всё темно
  gradient.addColorStop(0.7, 'rgba(0, 0, 0, 0)');    // Конец пятна

  ctx.fillStyle = gradient;
  ctx.beginPath();
  ctx.arc(0, 0, radius, 0, Math.PI * 2);
  ctx.fill();

  ctx.restore();
};

watch(
    () => visible.value,
    async (newVal) => {
      document.documentElement.classList.toggle('is-lock', newVal)

      await nextTick();
      drawVignette();

      if (newVal) {
        checkWidth()
        updateMusicBlock()
        updateMenuBlock()
      }

      if (newVal && pathRef.value) {
        pathRef.value.setAttribute('d', isPlaying.value ? pausePath : playPath);
      }
    }
)

const isPlaying = ref(false)

const pathRef = ref(null);
const pausePath = "M6,5 L9.5,5 L9.5,19 L6,19 Z M14.5,5 L18,5 L18,19 L14.5,19 Z";
const playPath = "M8.5,5 L14,8.5 L14,15.5 L8.5,19 Z M14,8.5 L19.5,12 L19.5,12 L14,15.5 Z";
const animateMorph = (playing) => {
  const tl = createTimeline({
    defaults: { easing: "easeInOutQuad" }
  });

  tl.add(pathRef.value, {
    d: playing ? pausePath : playPath,
    duration: 180,
  });
}

const currentTime = ref(0);
const duration = ref(0);
const progressStyle = computed(() => {
  const progress = (currentTime.value / duration.value) * 100
  return {
    background: `linear-gradient(to right, var(--color-accent) ${progress}%, rgba(255, 255, 255, 0.15) ${progress}%)`
  };
});
const handleTimeline = (event) => {
  audio.currentTime = +event.target.value
}
const formatTime = (time) => {
  const minutes = Math.floor(time / 60)
  const seconds = Math.floor(time % 60)

  return `${String(minutes).padStart(2, '0')}:${String(seconds).padStart(2, '0')}`
}

const tagIndex = ref(0)
const activeIndex = ref(0)
const audio = new Audio()
audio.onloadedmetadata = () => {
  duration.value = Math.round(audio.duration)
}
audio.ontimeupdate = () => {
  currentTime.value = Math.floor(audio.currentTime)
}
audio.onended = () => {
  if (activeIndex.value < list[tagIndex.value].length - 1) {
    nextItem()
  } else {
    isPlaying.value = false
    currentTime.value = 0
  }
}

const disabledPrevButton = computed(() => {
  return activeIndex.value === 0
})
const disabledNextButton = computed(() => {
  return activeIndex.value === list[tagIndex.value].length - 1
})

const loadAndPlay = () => {
  currentTime.value = 0
  audio.src = list[tagIndex.value][activeIndex.value].music
  audio.load()
  checkWidth()

  if (isPlaying.value) {
    audio.play()
  }
}
const handleMusicItem = (index) => {
  activeIndex.value = index
  updateMusicBlock()
  loadAndPlay()
}
const handleTag = (index) => {
  tagIndex.value = index
  activeIndex.value = 0
  updateMenuBlock()
  updateMusicBlock()
  loadAndPlay()
}
const prevItem = () => {
  activeIndex.value--
  loadAndPlay()
}
const nextItem = () => {
  activeIndex.value++
  loadAndPlay()
}

const activeTrack = computed(() => {
  return list[tagIndex.value][activeIndex.value]
})

const isNameOverflow = ref(false)
const isNameShadow = ref(false)
const nameTrackRef = ref(null)
const checkWidth = async () => {
  isNameOverflow.value = false
  isNameShadow.value = false

  await nextTick()

  if (nameTrackRef.value) {
    const containerWidth = nameTrackRef.value.parentElement.clientWidth
    const contentWidth = nameTrackRef.value.scrollWidth

    const check = contentWidth > containerWidth

    isNameShadow.value = check
    nameTrackRef.value.style.animationDuration = `${contentWidth / containerWidth * 20}s`
    setTimeout(() => {
      isNameOverflow.value = check
    }, 1000)
  }
}

loadAndPlay()

watch(() => isPlaying.value, (newVal) => {
  animateMorph(newVal)
  if (newVal) {
    audio.play()
  } else {
    audio.pause()
  }
})

const menuActive = ref(null)
const updateMenuBlock = () => {
  menuActive.value.style.setProperty('--active-index', tagIndex.value);
}
const musicActive = ref(null)
const updateMusicBlock = () => {
  musicActive.value.style.setProperty('--active-index', activeIndex.value);
}
</script>

<template>

  <div>
    <div class="music__open position-absolute cursor-pointer"
         title="музычка"
         @click="handleOpen"
         ref="openRef"
    >
      <i class="music__icon w-100 h-100" v-if="iconVisible">
        <svg fill="#000000" height="800px" width="800px" version="1.1" id="_x31_" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink"
             viewBox="0 0 128 128" xml:space="preserve">
              <ellipse transform="matrix(0.1607 -0.987 0.987 0.1607 -3.8377 74.789)" cx="42.1" cy="39.7" rx="17.3" ry="17.3"/>
          <path d="M84.7,40.1c-0.6-1.6-1.6-3-3-4.1c-3.9-2.8-9.3-2-12.1,1.9c-2.8,3.9-2,9.3,1.9,12.1c3.9,2.8,9.3,2,12.1-1.9
                c0.1-0.2,0.2-0.3,0.3-0.5l5.9,0.6L66.3,65.2l-32.2-5.4c-5.4-0.9-10.4,2.7-11.2,8.1l-7.3,45.1h35.8c1.6-5.6,2.8-11.4,3.8-17.3
                c0.1-0.9,0.5-3,0.5-3.3c0.8-4.8,0.4-9.9-1.2-14.3l12.7,2.1c1.8,0.3,3.9-0.1,5.7-1.4l28.1-20.2c2.7-2,3.7-5.6,2.7-8.6l10.9,1.2
                l0.8-7.5L84.7,40.1z"/>
          </svg>
      </i>
    </div>

    <Transition name="fade">
      <div class="music__content position-absolute" v-if="visible">
        <canvas
            ref="canvasRef"
            class="position-absolute w-100 h-100 pointer-events-none"
            style="z-index: 99;"
        ></canvas>

        <button class="music__close position-absolute z-100"
                type="button"
                @click="handleClose"
        >
          Закрыть
        </button>

        <p class="music__rights position-absolute z-100 text-w500">
          ©Все права нахуй никем не защищены. Мы спиздили и вы пиздьте (ну или как там..))
        </p>

        <ul class="music__menu position-absolute z-100 flex flex-column">
          <li v-for="(item, index) in menuList"
              :key="item.id"
              class="music__menu-item w-100"
          >
            <button class="flex flex-align-center text-w500 w-100"
                    type="button"
                    :class="{'is-active': tagIndex === index}"
                    @click="handleTag(index)"
            >
              <Component :is="item.icon"/>
              {{item.name}}
            </button>
          </li>
          <li class="music__menu-active position-absolute" ref="menuActive"/>
        </ul>

        <ul class="music__list position-absolute z-100 flex flex-column">
          <li v-for="(item, index) in list[tagIndex]"
              :key="item.id"
              class="music__item flex cursor-pointer"
              :class="{'is-active': activeIndex === index}"
              @click="handleMusicItem(index)"
          >
            <div class="music__item-left img-container">
              <img :src="item.icon" :alt="item.name">
            </div>
            <div class="music__item-right">
              <p class="music__item-name text-w500 w-100">{{item.name}}</p>
              <p class="music__item-artist w-100">{{item.artist}}</p>
              <p class="music__item-duration">{{item.duration}}</p>
            </div>
          </li>
          <li class="music__item-active position-absolute w-100" ref="musicActive"/>
        </ul>

        <div class="music__img-container img-container position-absolute">
          <img :src="activeTrack.icon" :alt="activeTrack.name" draggable="false">
        </div>

        <div class="music__card position-absolute z-100">
          <Transition name="list">
            <div class="music__info" :key="activeTrack.name">
              <div class="music__title text-w700 w-100 overflow-hidden position-relative"
                   :class="{'shadow': isNameShadow}"
              >
                <div class="music__track" :class="{ 'animate': isNameOverflow }" ref="nameTrackRef">
                  <span>{{ activeTrack.name }}</span>
                  <span v-if="isNameOverflow" class="music__spacer">{{ activeTrack.name }}</span>
                </div>
              </div>

              <div class="music__author w-100 overflow-hidden position-relative">
                <span>{{ activeTrack.artist }}</span>
              </div>
            </div>
          </Transition>

          <div class="music__controllers">
            <input class="music__timeline w-100"
                   type="range"
                   v-model="currentTime"
                   :max="duration"
                   :style="progressStyle"
                   @input="handleTimeline"
            >

            <div class="music__time flex">
              <span>{{formatTime(currentTime)}}</span>
              <span>{{activeTrack.duration}}</span>
            </div>

            <div class="music__btn-bar flex-center">
              <button class="music__btn prev"
                      type="button"
                      :disabled="disabledPrevButton"
                      @click="prevItem"
              >
                <svg width="800px" height="800px" viewBox="0 0 20 20" version="1.1" xmlns="http://www.w3.org/2000/svg">
                  <g id="layer1">
                    <path d="M 7,3.2910156 6.2910156,4 6.6464844,4.3535156 12.291016,10 6.6464844,15.646484 6.2910156,16 7,16.708984 7.3535156,16.353516 13.708984,10 7.3535156,3.6464844 Z" style="fill:currentColor; fill-opacity:1; stroke:none; stroke-width:0px;"/>
                  </g>
                </svg>
              </button>

              <button class="music__btn play-pause"
                      type="button"
                      @click="isPlaying = !isPlaying"
              >
                <svg width="100" height="100" viewBox="0 0 24 24">
                  <defs>
                    <filter id="gooey-tight">
                      <feGaussianBlur in="SourceGraphic" stdDeviation="0.8" result="blur" />
                      <feColorMatrix
                          in="blur"
                          mode="matrix"
                          values="1 0 0 0 0  0 1 0 0 0  0 0 1 0 0  0 0 0 24 -11"
                      />
                    </filter>
                  </defs>

                  <path
                      ref="pathRef"
                      fill="currentColor"
                      stroke="currentColor"
                      stroke-width="1.5"
                      stroke-linejoin="round"
                      stroke-linecap="round"
                      style="filter: url(#gooey-tight)"
                  />
                </svg>
              </button>

              <button class="music__btn next"
                      type="button"
                      :disabled="disabledNextButton"
                      @click="nextItem"
              >
                <svg width="800px" height="800px" viewBox="0 0 20 20" version="1.1" xmlns="http://www.w3.org/2000/svg">
                  <g id="layer1">
                    <path d="M 7,3.2910156 6.2910156,4 6.6464844,4.3535156 12.291016,10 6.6464844,15.646484 6.2910156,16 7,16.708984 7.3535156,16.353516 13.708984,10 7.3535156,3.6464844 Z" style="fill:currentColor; fill-opacity:1; stroke:none; stroke-width:0px;"/>
                  </g>
                </svg>
              </button>
            </div>
          </div>
        </div>
      </div>
    </Transition>
  </div>

</template>