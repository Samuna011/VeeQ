<script setup>
import { onMounted, ref, onUnmounted } from 'vue'

const canvas = ref(null)
let ctx
let particles = []
let animationId
let width, height

onMounted(() => {
  ctx = canvas.value.getContext('2d')
  resizeCanvas()
  initParticles()
  animate()
  
  window.addEventListener('resize', handleResize)
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
  // Fix DPI blurriness
  const dpr = window.devicePixelRatio || 1
  canvas.value.width = width * dpr
  canvas.value.height = height * dpr
  ctx.scale(dpr, dpr)
}

function initParticles() {
  const particleCount = Math.floor(width / 15) // Responsive count
  
  for (let i = 0; i < particleCount; i++) {
    particles.push({
      x: Math.random() * width,
      y: Math.random() * height,
      size: Math.random() * 8 + 3, // Petal size
      speedY: Math.random() * 1.5 + 0.5,
      speedX: Math.random() * 1 - 0.5,
      rotation: Math.random() * Math.PI * 2,
      rotationSpeed: (Math.random() - 0.5) * 0.05,
      opacity: Math.random() * 0.4 + 0.1,
      // Color variations: Black or Very Dark Purple
      color: Math.random() > 0.5 ? '#000000' : '#220033' 
    })
  }
}

function drawPetal(p) {
  ctx.save()
  ctx.translate(p.x, p.y)
  ctx.rotate(p.rotation)
  ctx.globalAlpha = p.opacity
  ctx.fillStyle = p.color
  
  // Draw an elliptical leaf/petal shape
  ctx.beginPath()
  ctx.ellipse(0, 0, p.size, p.size * 2, 0, 0, Math.PI * 2)
  ctx.fill()
  
  ctx.restore()
}

function animate() {
  ctx.clearRect(0, 0, width, height)
  
  for (let i = 0; i < particles.length; i++) {
    let p = particles[i]
    
    p.y += p.speedY
    p.x += p.speedX
    p.rotation += p.rotationSpeed
    
    // Slight sway
    p.x += Math.sin(p.y * 0.01) * 0.5
    
    drawPetal(p)
    
    // Reset petal to top when it falls off screen
    if (p.y > height + p.size * 2) {
      p.y = -p.size * 2
      p.x = Math.random() * width
    }
    // wrap horizontal bounds
    if (p.x > width + p.size) p.x = -p.size
    if (p.x < -p.size) p.x = width + p.size
  }
  
  animationId = requestAnimationFrame(animate)
}
</script>

<template>
  <!-- Fixed background canvas, pointer events none so it doesn't block scroll -->
  <canvas ref="canvas" class="fixed inset-0 pointer-events-none z-0 w-screen h-screen opacity-70"></canvas>
</template>
