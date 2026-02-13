<script setup>
import {onMounted, ref, watch} from "vue";
// import * as THREE from 'three';
// import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader';
// import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';
import {createTimeline} from "animejs";

const props = defineProps({
  scroll: {
    type: Number,
    default: 0
  }
})

// const container = ref(null)
// let model = null

// onMounted(() => {
//   const scene = new THREE.Scene()
//   const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000)
//   camera.position.z = 5
//
//   const renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true })
//   renderer.setSize(window.innerWidth, window.innerHeight)
//   container.value.appendChild(renderer.domElement)
//
//   const controls = new OrbitControls(camera, renderer.domElement);
//   controls.enableDamping = true;
//   controls.dampingFactor = 0.05;
//   controls.screenSpacePanning = false;
//   controls.minDistance = 2;
//   controls.maxDistance = 10;
//   controls.enableZoom = false;
//   controls.enablePan = false;
//
//   const light = new THREE.AmbientLight(0xffffff, 2)
//   scene.add(light)
//   const directionalLight = new THREE.DirectionalLight(0xffffff, 1)
//   directionalLight.position.set(5, 5, 5)
//   scene.add(directionalLight)
//
//   const loader = new GLTFLoader()
//   loader.load('/heart.glb', (gltf) => {
//     model = gltf.scene
//     model.scale.set(2.5,2.5,2.5)
//
//     const box = new THREE.Box3().setFromObject(model);
//     const center = new THREE.Vector3();
//     box.getCenter(center);
//     model.children[0].position.sub(center)
//     model.position.x -= -10;
//
//     scene.add(model)
//
//     const animate = () => {
//       requestAnimationFrame(animate)
//       renderer.render(scene, camera)
//     };
//     animate()
//   })
// })
//
// watch(
//     () => props.scroll,
//     (newVal) => {
//       if (model) {
//         model.rotation.y = newVal * Math.PI * 2
//         model.rotation.x = newVal * 0.5
//
//         const startX = -10
//         const endX = 0
//         model.position.x = startX + (endX - startX) * newVal
//       }
//     }
// )

const valentineIsOpen = ref(false)

let timeline = null

onMounted(() => {
  timeline = createTimeline({
    defaults: {
      autoplay: false,
      easing: "easeOutExpo",
    }
  })

  timeline
      .add('.forest__sky', {
        opacity: [0, 1],
        duration: 100,
      })
      .add('.forest__cloud-1', {
        scale: [0, 1],
        duration: 100
      })
      .add('.forest__cloud-2', {
        scale: [0, 1],
        duration: 100
      }, '-=50')
      .add('.forest__cloud-3', {
        scale: [0, 1],
        duration: 100
      }, '-=50')
      .add('.forest__cloud-4', {
        scale: [0, 1],
        duration: 100
      }, '-=50')
      .add('.forest__forest', {
        translateY: [window.innerHeight * 1.3, 0],
        duration: 100
      }, '-=50')
      .add('.forest__fox', {
        translateX: [-window.innerWidth / 2, 0],
        duration: 100
      })
      .add('.forest__valentine', {
        translateY: [window.innerWidth / 2, 0],
        duration: 100
      })
      .add('.forest__fox-text', {
        scale: [0, 1],
        duration: 100
      })
})

watch(
    () => props.scroll,
    (newVal) => {
      if (timeline) {
        timeline.seek(newVal * 8)
      }
    }
)
</script>

<template>

  <div class="forest position-fixed h-100 w-100">
<!--    <div class="forest__object" ref="container"></div>-->

    <img class="forest__sky w-100 h-100" src="/forest/sky.webp" alt="sky"/>

    <img class="forest__cloud-1" src="/forest/cloud-1.webp" alt="cloud" draggable="false"/>
    <img class="forest__cloud-2" src="/forest/cloud-2.webp" alt="cloud" draggable="false"/>
    <img class="forest__cloud-3" src="/forest/cloud-3.webp" alt="cloud" draggable="false"/>
    <img class="forest__cloud-4" src="/forest/cloud-4.webp" alt="cloud" draggable="false"/>

    <img class="forest__forest w-100" src="/forest/forest.webp" alt="forest" draggable="false"/>

    <div class="forest__fox-text position-absolute z-10">
      <img class="position-absolute" src="/forest/text.webp" alt="text" draggable="false">
      <span>кликни по сердцу</span>
    </div>
    <img class="forest__fox" src="/forest/fox.webp" alt="fox" draggable="false"/>

    <div class="forest__valentine cursor-pointer z-100" @click="valentineIsOpen = !valentineIsOpen">
      <div class="forest__heart-outside position-absolute z-10 w-100 h-100" :class="{'is-open': valentineIsOpen}">
        <img class="face w-100 h-100" src="/forest/heart.webp" alt="heart" draggable="false"/>
        <img class="backface w-100 h-100" src="/forest/heart-backface.webp" alt="heart-inside" draggable="false"/>
      </div>
      <div class="forest__heart-inside z-1 w-100 h-100 flex-center">
        <img src="/forest/heart-inside.webp" alt="heart-inside" draggable="false"/>
        <span class="forest__text whitespace-nowrap text-w500">с праздничком!! плотни))</span>
      </div>
    </div>

    <div class="forest__background position-absolute z-10" />
  </div>

</template>