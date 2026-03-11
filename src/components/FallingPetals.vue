<script setup>
import { onMounted, ref, onUnmounted, watch } from 'vue'

const props = defineProps({
  isPaused: { type: Boolean, default: false }
})

const canvas = ref(null)
let ctx
let particles = []
let animationId
let width, height
let time = 0

onMounted(() => {
  ctx = canvas.value.getContext('2d')
  resizeCanvas()
  initParticles()
  if (!props.isPaused) animate()
  window.addEventListener('resize', handleResize)
})

watch(() => props.isPaused, (paused) => {
  if (paused && animationId) {
    cancelAnimationFrame(animationId)
    animationId = null
  } else if (!paused && canvas.value && ctx) {
    animate()
  }
})

onUnmounted(() => {
  cancelAnimationFrame(animationId)
  window.removeEventListener('resize', handleResize)
})

function handleResize() {
  resizeCanvas()
  particles = []
  initParticles()
}

function resizeCanvas() {
  width = window.innerWidth
  height = window.innerHeight
  const dpr = window.devicePixelRatio || 1
  canvas.value.width = width * dpr
  canvas.value.height = height * dpr
  ctx.scale(dpr, dpr)
}

function initParticles() {
  const particleCount = Math.floor((width / 12) * (height / 800)) + 40

  const colors = [
    '#0f0a14', '#1a0d24', '#2d1b4e', '#3b0764', '#4c1d95', '#1e0333', '#000000'
  ]

  for (let i = 0; i < particleCount; i++) {
    const sizeY = Math.random() * 14 + 6
    const sizeX = sizeY * (0.35 + Math.random() * 0.35)
    particles.push({
      x: Math.random() * width,
      y: Math.random() * height,
      sizeX,
      sizeY,
      speedY: Math.random() * 1.2 + 0.4,
      speedX: (Math.random() - 0.5) * 0.8,
      rotation: Math.random() * Math.PI * 2,
      rotationSpeed: (Math.random() - 0.5) * 0.04,
      opacity: Math.random() * 0.35 + 0.08,
      color: colors[Math.floor(Math.random() * colors.length)],
      phase: Math.random() * Math.PI * 2,
      sway: 0.8 + Math.random() * 1.2,
    })
  }
}

function drawPetal(p) {
  ctx.save()
  ctx.translate(p.x, p.y)
  ctx.rotate(p.rotation)

  const gradient = ctx.createLinearGradient(-p.sizeX, -p.sizeY, p.sizeX, p.sizeY)
  gradient.addColorStop(0, p.color)
  gradient.addColorStop(0.5, p.color)
  gradient.addColorStop(1, p.color)

  ctx.globalAlpha = p.opacity * (0.85 + 0.15 * Math.sin(time * 0.5 + p.phase))
  ctx.fillStyle = gradient

  ctx.beginPath()
  ctx.ellipse(0, 0, p.sizeX, p.sizeY, 0, 0, Math.PI * 2)
  ctx.fill()

  ctx.restore()
}

function animate() {
  if (props.isPaused) return
  time += 0.02
  ctx.clearRect(0, 0, width, height)

  for (let i = 0; i < particles.length; i++) {
    const p = particles[i]

    p.y += p.speedY
    p.x += p.speedX
    p.rotation += p.rotationSpeed
    p.x += Math.sin(p.y * 0.008 + time * 0.3) * p.sway
    p.x += Math.sin(time * 0.5 + p.phase) * 0.3

    drawPetal(p)

    if (p.y > height + p.sizeY * 2) {
      p.y = -p.sizeY * 2
      p.x = Math.random() * width
    }
    if (p.x > width + p.sizeX) p.x = -p.sizeX
    if (p.x < -p.sizeX) p.x = width + p.sizeX
  }

  if (!props.isPaused) animationId = requestAnimationFrame(animate)
}
</script>

<template>
  <canvas ref="canvas" class="fixed inset-0 pointer-events-none z-0 w-screen h-screen opacity-80" style="mix-blend-mode: multiply;"></canvas>
</template>
