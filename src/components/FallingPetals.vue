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

// We pre-render the petals to completely eliminate heavy per-frame API calls
const textureCount = 10
const textures = []
const baseSizeY = 60
const baseSizeX = 25

onMounted(() => {
  ctx = canvas.value.getContext('2d')
  createPetalTextures()
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
  textures.length = 0 // clean memory
})

function createPetalTextures() {
  // Pre-bake 10 different stages of blur and opacity for high performance
  for (let i = 0; i < textureCount; i++) {
    const offscreen = document.createElement('canvas')
    const offCtx = offscreen.getContext('2d')
    
    // i=0 is sharpest foreground, i=9 is blurriest background
    const blurAmount = (i / (textureCount - 1)) * 14
    
    // Ensure canvas has enough margin so the blur doesn't clip at the edges
    const margin = Math.ceil(blurAmount) * 2 + 10
    offscreen.width = baseSizeX * 2 + margin * 2
    offscreen.height = baseSizeY * 2 + margin * 2
    
    const centerX = offscreen.width / 2
    const centerY = offscreen.height / 2
    
    offCtx.save()
    offCtx.translate(centerX, centerY)
    
    if (blurAmount > 0) {
      offCtx.filter = `blur(${blurAmount}px)`
    }
    
    const zDepth = 1 - (i / (textureCount - 1))
    const opacity = zDepth * 0.4 + 0.4
    
    const gradient = offCtx.createLinearGradient(0, -baseSizeY, 0, baseSizeY)
    gradient.addColorStop(0, `rgba(5, 2, 10, ${opacity})`)
    gradient.addColorStop(0.5, `rgba(25, 5, 45, ${opacity})`)
    gradient.addColorStop(1, `rgba(0, 0, 5, ${opacity})`)
    
    offCtx.fillStyle = gradient
    offCtx.beginPath()
    offCtx.moveTo(0, -baseSizeY)
    offCtx.bezierCurveTo(baseSizeX, -baseSizeY * 0.5, baseSizeX * 1.4, baseSizeY * 0.5, 0, baseSizeY)
    offCtx.bezierCurveTo(-baseSizeX * 1.4, baseSizeY * 0.5, -baseSizeX, -baseSizeY * 0.5, 0, -baseSizeY)
    offCtx.fill()
    offCtx.restore()
    
    textures.push(offscreen)
  }
}

function handleResize() {
  resizeCanvas()
  particles = []
  initParticles()
}

function resizeCanvas() {
  width = window.innerWidth
  height = window.innerHeight
  // Using devicePixelRatio keeps it crisp on Retina displays, but can be disabled if still heavy.
  const dpr = window.devicePixelRatio || 1
  canvas.value.width = width * dpr
  canvas.value.height = height * dpr
  ctx.scale(dpr, dpr)
}

function initParticles() {
  const particleCount = Math.floor((width / 40) * (height / 800)) + 12

  for (let i = 0; i < particleCount; i++) {
    const zDepth = Math.random()
    
    const sizeY = (zDepth * 40) + 20
    
    // Choose texture closest to this zDepth. (0 is far, 1 is near)
    // We reverse it because texture index 0 is near, 9 is far.
    let textureIndex = Math.floor((1 - zDepth) * textureCount)
    if (textureIndex >= textureCount) textureIndex = textureCount - 1

    particles.push({
      x: Math.random() * width,
      y: Math.random() * height,
      sizeY,
      textureIndex,
      speedY: (zDepth * 2.2) + 0.8 + Math.random() * 0.6,
      speedX: (Math.random() - 0.5) * 1.5,
      rotation: Math.random() * Math.PI * 2,
      rotationSpeed: (Math.random() - 0.5) * 0.02,
      phase: Math.random() * Math.PI * 2,
      sway: (zDepth * 1.8) + 0.8,
      perspectiveY: 0.5 + Math.random() * 0.5
    })
  }
}

function drawPetal(p) {
  ctx.save()
  ctx.translate(p.x, p.y)
  ctx.rotate(p.rotation)

  const spinFactor = Math.abs(Math.sin(time * 0.8 + p.phase)) * p.perspectiveY + 0.2
  ctx.scale(1, spinFactor)

  const texture = textures[p.textureIndex]
  const scale = p.sizeY / baseSizeY
  
  const drawWidth = texture.width * scale
  const drawHeight = texture.height * scale
  
  // Render the pre-baked hardware texture instead of drawing paths and filters
  ctx.drawImage(texture, -drawWidth / 2, -drawHeight / 2, drawWidth, drawHeight)

  ctx.restore()
}

function animate() {
  if (props.isPaused) return
  time += 0.015
  ctx.clearRect(0, 0, width, height)

  for (let i = 0; i < particles.length; i++) {
    const p = particles[i]

    p.y += p.speedY
    p.x += p.speedX
    p.rotation += p.rotationSpeed
    p.x += Math.sin(p.y * 0.005 + time * 0.2) * p.sway
    p.x += Math.sin(time * 0.4 + p.phase) * 0.5

    drawPetal(p)

    // Reset when out of frame
    const margin = p.sizeY * 3
    if (p.y > height + margin) {
      p.y = -margin
      p.x = Math.random() * width
    }
    if (p.x > width + margin) p.x = -margin
    if (p.x < -margin) p.x = width + margin
  }

  if (!props.isPaused) animationId = requestAnimationFrame(animate)
}
</script>

<template>
  <canvas ref="canvas" class="fixed inset-0 pointer-events-none z-0 w-screen h-screen opacity-90" style="mix-blend-mode: multiply;"></canvas>
</template>
