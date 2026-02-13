<template>
  <canvas ref="canvasRef" class="space-canvas"></canvas>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'

const SNOW_CONFIG = {
  STAR_COUNT: 250,      // Количество звезд
  COMET_CHANCE: 0.005,  // Шанс появления кометы в кадре (0.5%)
  COMET_SPEED: 4,      // Скорость кометы
}

const canvasRef = ref(null)
const ctx = ref(null)
const animationId = ref(null)
const stars = ref([])
const activeComet = ref(null) // Только одна комета за раз

// Создание звезды
const createStar = (canvas) => ({
  x: Math.random() * canvas.width,
  y: Math.random() * canvas.height,
  size: Math.random() * 1.5 + 0.5,
  opacity: Math.random(),
  pulseSpeed: Math.random() * 0.05 + 0.01, // Скорость мерцания
  phase: Math.random() * Math.PI * 2       // Начальная фаза (чтобы не все сразу)
})

// Создание кометы
const createComet = (canvas) => {
  return {
    x: Math.random() * canvas.width,
    y: 0,
    len: Math.random() * 80 + 100, // Длина хвоста
    speed: SNOW_CONFIG.COMET_SPEED,
    angle: Math.PI / 4 // Летит под углом 45 градусов
  }
}

const setupCanvas = () => {
  if (!canvasRef.value) return
  const canvas = canvasRef.value
  canvas.width = window.innerWidth
  canvas.height = window.innerHeight
  ctx.value = canvas.getContext('2d')

  stars.value = Array.from({ length: SNOW_CONFIG.STAR_COUNT }, () => createStar(canvas))
}

const draw = () => {
  if (!ctx.value || !canvasRef.value) return
  const canvas = canvasRef.value
  const c = ctx.value

  c.clearRect(0, 0, canvas.width, canvas.height)

  // 1. Рисуем звезды
  stars.value.forEach(star => {
    // Плавная пульсация через синус
    star.phase += star.pulseSpeed
    const currentOpacity = (Math.sin(star.phase) + 1) / 2 * 0.8 + 0.2

    c.beginPath()
    c.arc(star.x, star.y, star.size, 0, Math.PI * 2)
    c.fillStyle = `rgba(255, 255, 255, ${currentOpacity})`
    c.fill()
  })

  // 2. Логика кометы
  if (!activeComet.value && Math.random() < SNOW_CONFIG.COMET_CHANCE) {
    activeComet.value = createComet(canvas)
  }

  if (activeComet.value) {
    const comet = activeComet.value

    // Рисуем хвост (градиент)
    const gradient = c.createLinearGradient(
        comet.x, comet.y,
        comet.x - Math.cos(comet.angle) * comet.len,
        comet.y - Math.sin(comet.angle) * comet.len
    )
    gradient.addColorStop(0, 'rgba(255, 255, 255, 1)')
    gradient.addColorStop(1, 'rgba(255, 255, 255, 0)')

    c.beginPath()
    c.moveTo(comet.x, comet.y)
    c.lineTo(comet.x - Math.cos(comet.angle) * comet.len, comet.y - Math.sin(comet.angle) * comet.len)
    c.strokeStyle = gradient
    c.lineWidth = 2
    c.stroke()

    // Двигаем комету
    comet.x += comet.speed
    comet.y += comet.speed

    // Удаляем, если улетела
    if (comet.y > canvas.height || comet.x > canvas.width) {
      activeComet.value = null
    }
  }

  animationId.value = requestAnimationFrame(draw)
}

onMounted(() => {
  setupCanvas()
  draw()
  window.addEventListener('resize', setupCanvas)
})

onUnmounted(() => {
  window.removeEventListener('resize', setupCanvas)
  cancelAnimationFrame(animationId.value)
})
</script>

<style scoped>
.space-canvas {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  z-index: -1;
}
</style>