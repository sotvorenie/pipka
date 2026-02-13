<script setup>
import {ref, watch} from "vue";

const emits = defineEmits(['open', 'close'])

const visible = ref(false)

const open = () => {
  emits('open')
  visible.value = true
}

const close = () => {
  emits('close')
  visible.value = false
}

const closeBtnVisible = ref(false)

const languages = [
  {
    name: 'SCSS',
    class: 'scss',
    icon: '/about/scss.webp'
  },
  {
    name: 'Vue.js',
    class: 'vue',
    icon: '/about/vue.webp'
  },
  {
    name: 'Anime.js',
    class: 'anime',
    icon: '/about/anime.webp'
  },
]
const visibleLanguages = ref([])

watch(
    () => visible.value,
    async (newVal) => {
      document.documentElement.classList.toggle('is-lock', newVal)

      setTimeout(() => {
        closeBtnVisible.value = newVal
      }, 500)

      if (newVal) {
        visibleLanguages.value = []
        setTimeout(() => {
          visibleLanguages.value = [...languages];
        }, 100);
      } else {
        visibleLanguages.value = []
      }
    }
)
</script>

<template>

  <div>
    <div class="about__open position-absolute cursor-pointer" :class="{'is-active': visible}">
      <div class="about__img-container img-container position-relative"
           :title="visible ? 'закрыть' : 'секретик'"
           @click="visible ? close() : open()"
      >
        <img src="/билли.jpg" alt="Билли" draggable="false"/>
      </div>

      <Transition name="sex">
        <a href="https://avatars.mds.yandex.net/i?id=94e296ca0d8899804e20b70c068eaddd_l-5234859-images-thumbs&n=13"
           class="about__btn sex position-absolute text-w500"
           type="button"
           v-if="visible"
           target="_blank"
        >секс</a>
      </Transition>
      <Transition name="drugs">
        <a href="https://avatars.mds.yandex.net/i?id=2a703956f89f0bd247c6c798b949d8b6_l-10313458-images-thumbs&n=13"
           class="about__btn drugs position-absolute text-w500"
           type="button"
           v-if="visible"
           target="_blank"
        >наркотики</a>
      </Transition>
      <Transition name="bonus">
        <a href="https://avatars.mds.yandex.net/i?id=24b258f60b47064d0d7d4294da24745a_l-4756892-images-thumbs&n=13"
           class="about__btn bonus position-absolute text-w500"
           type="button"
           v-if="visible"
           target="_blank"
        >бонус</a>
      </Transition>
    </div>

    <Transition name="fade">
      <div class="about__content position-absolute flex-center" v-if="visible">
        <Transition name="scale">
          <button class="about__close flex flex-align-center about__btn position-absolute"
                  type="button"
                  @click="close"
                  v-if="closeBtnVisible"
          >
            <img src="/about/fox.gif" alt="fox" draggable="false"/>
            Закрыть
          </button>
        </Transition>

        <TransitionGroup name="languages" tag="ul" class="about__list flex">
          <li v-for="(item, index) in visibleLanguages"
              :key="item.name"
              :class="`about__item ${item.class} cursor-pointer`"
              :style="{ transitionDelay: `${index * 0.15}s` }"
          >
            <img :src="item.icon" :alt="item.name" draggable="false"/>
            <span class="about__title text-w700">{{item.name}}</span>
          </li>
        </TransitionGroup>
      </div>
    </Transition>

  </div>
</template>