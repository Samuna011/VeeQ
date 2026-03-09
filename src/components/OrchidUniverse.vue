<script setup>
import { onMounted, ref, onUnmounted } from 'vue'
import * as THREE from 'three'
import gsap from 'gsap'

const emit = defineEmits(['enterSite'])

const canvasContainer = ref(null)
const showEnterButton = ref(false)

let scene, camera, renderer, particles
let animationId
let isTransitioning = false
let time = 0

// Constants for interaction
let targetSpeed = 0.5
let currentSpeed = 0.5
const starCount = 15000

onMounted(() => {
  initThree()
  createUniverse()
  animate()
  
  window.addEventListener('wheel', handleInteraction, { once: true })
  window.addEventListener('click', handleInteraction, { once: true })
})

onUnmounted(() => {
  cancelAnimationFrame(animationId)
  if (renderer) renderer.dispose()
  window.removeEventListener('wheel', handleInteraction)
  window.removeEventListener('click', handleInteraction)
})

function initThree() {
  scene = new THREE.Scene()
  scene.background = new THREE.Color('#030005') // Extremely dark purple-black
  scene.fog = new THREE.FogExp2('#030005', 0.0015)

  camera = new THREE.PerspectiveCamera(60, window.innerWidth / window.innerHeight, 0.1, 2000)
  camera.position.z = 0

  renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true })
  renderer.setSize(window.innerWidth, window.innerHeight)
  renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))
  canvasContainer.value.appendChild(renderer.domElement)

  window.addEventListener('resize', onWindowResize)
}

function createCircleTexture() {
  const canvas = document.createElement('canvas')
  canvas.width = 32
  canvas.height = 32
  const ctx = canvas.getContext('2d')
  ctx.beginPath()
  ctx.arc(16, 16, 16, 0, Math.PI * 2)
  ctx.fillStyle = 'white'
  ctx.fill()
  return new THREE.CanvasTexture(canvas)
}

function createUniverse() {
  const geometry = new THREE.BufferGeometry()
  const circleTexture = createCircleTexture()
  const positions = new Float32Array(starCount * 3)
  const colors = new Float32Array(starCount * 3)
  
  const color1 = new THREE.Color('#ffffff') // bright white stars
  const color2 = new THREE.Color('#4b0082') // dark orchid
  const color3 = new THREE.Color('#9932cc') // light orchid
  const color4 = new THREE.Color('#220033') // deep space purple

  for (let i = 0; i < starCount; i++) {
    // Distribute stars in a massive long tunnel area
    positions[i * 3] = (Math.random() - 0.5) * 2000 // x
    positions[i * 3 + 1] = (Math.random() - 0.5) * 2000 // y
    positions[i * 3 + 2] = (Math.random() - 1.0) * 4000 // z (deep into screen)

    const randCol = Math.random()
    let mixedColor
    if (randCol > 0.8) mixedColor = color1
    else if (randCol > 0.4) mixedColor = color2
    else if (randCol > 0.1) mixedColor = color3
    else mixedColor = color4

    colors[i * 3] = mixedColor.r
    colors[i * 3 + 1] = mixedColor.g
    colors[i * 3 + 2] = mixedColor.b
  }

  geometry.setAttribute('position', new THREE.BufferAttribute(positions, 3))
  geometry.setAttribute('color', new THREE.BufferAttribute(colors, 3))

  const material = new THREE.PointsMaterial({
    size: 4,
    vertexColors: true,
    blending: THREE.AdditiveBlending,
    transparent: true,
    opacity: 0.8,
    sizeAttenuation: true,
    map: circleTexture,
    depthWrite: false
  })

  particles = new THREE.Points(geometry, material)
  scene.add(particles)
}

function handleInteraction() {
  if (isTransitioning) return
  isTransitioning = true

  // 1. Kick into high-speed wrap travel
  gsap.to(camera, {
    fov: 120, // Stretch stars
    duration: 2.5,
    ease: "power2.inOut",
    onUpdate: () => camera.updateProjectionMatrix()
  })

  // Accelerate particle travel speed drastically
  gsap.to(window, {
    targetSpeed: 40,
    duration: 2.5,
    ease: "power2.inOut",
    onUpdate: function() {
      // gsap is tweening the global targetSpeed var we declared (not strictly accurate in Vue script setup, so lets use a proxy object)
    }
  })

  gsap.set('.lotus-path', { strokeDasharray: 1000, strokeDashoffset: 1000 })

  const speedProxy = { val: currentSpeed }
  const tl = gsap.timeline({
    onComplete: () => {
      showEnterButton.value = true
    }
  })

  // The warp sequence
  tl.to(speedProxy, {
    val: 60, // Peak speed
    duration: 2,
    ease: "power3.in",
    onUpdate: () => { targetSpeed = speedProxy.val }
  })
  .to(speedProxy, {
    val: 1, // Slow down into the center of a nebula
    duration: 3,
    ease: "power3.out",
    onUpdate: () => { targetSpeed = speedProxy.val }
  })
  
  // Bring FOV back to normal after bursting through
  tl.to(camera, {
    fov: 60,
    duration: 3,
    ease: "power2.out",
    onUpdate: () => camera.updateProjectionMatrix()
  }, "-=3")
  
  // Animate the Lotus outline drawing itself
  tl.to('#lotus-container', {
    opacity: 1,
    scale: 1,
    duration: 5,
    ease: "power3.out"
  }, "-=3")
  .to('.lotus-path', {
    strokeDashoffset: 0,
    duration: 5,
    ease: "power2.inOut"
  }, "-=4")
  
  // Slowly rotate the camera during the slow-down to give a drifting feel at the core
  tl.to(camera.rotation, {
    z: Math.PI / 4,
    x: 0.1,
    y: 0.1,
    duration: 5,
    ease: "power1.out"
  }, "-=4")
}

function onWindowResize() {
  camera.aspect = window.innerWidth / window.innerHeight
  camera.updateProjectionMatrix()
  renderer.setSize(window.innerWidth, window.innerHeight)
}

function animate() {
  animationId = requestAnimationFrame(animate)
  time += 0.01

  // Smoothly interpolate current speed towards target speed for organic feeling
  currentSpeed += (targetSpeed - currentSpeed) * 0.1

  if (particles) {
    const positions = particles.geometry.attributes.position.array
    
    // Move particles towards camera (positive Z)
    for (let i = 0; i < starCount; i++) {
      positions[i * 3 + 2] += currentSpeed // z axis movement

      // Also add slight swirly rotation if we are deeply inside the warp
      if (currentSpeed > 5) {
        // Simple rotation matrix around Z
        const x = positions[i * 3]
        const y = positions[i * 3 + 1]
        const angle = currentSpeed * 0.0001
        const cosDiff = Math.cos(angle)
        const sinDiff = Math.sin(angle)
        positions[i * 3] = x * cosDiff - y * sinDiff
        positions[i * 3 + 1] = x * sinDiff + y * cosDiff
      }

      // If particle passes behind camera, reset it far into the distance
      if (positions[i * 3 + 2] > 200) {
        positions[i * 3 + 2] = -4000
        // Randomize X and Y slightly again to avoid recognizable patterns continuously
        positions[i * 3] = (Math.random() - 0.5) * 2000
        positions[i * 3 + 1] = (Math.random() - 0.5) * 2000
      }
    }
    particles.geometry.attributes.position.needsUpdate = true
  }

  renderer.render(scene, camera)
}
</script>

<template>
  <div class="relative w-screen h-screen overflow-hidden bg-[#030005] text-white">
    <div ref="canvasContainer" class="absolute inset-0 cursor-none"></div>
    
    <!-- Ethereal Glow Overlay for when in the inner nebula -->
    <div class="absolute inset-0 z-0 mix-blend-screen pointer-events-none transition-opacity duration-[3000ms] ease-out"
         :class="showEnterButton ? 'opacity-30' : 'opacity-0'"
         style="background: radial-gradient(circle at center, #9932cc 0%, transparent 50%);">
    </div>

    <!-- Glowing Lotus Outline -->
    <div id="lotus-container" class="absolute inset-0 flex items-center justify-center pointer-events-none z-20 opacity-0 scale-50" style="filter: drop-shadow(0 0 15px rgba(153, 50, 204, 0.8));">
      <svg width="400" height="400" viewBox="0 0 100 100" class="overflow-visible">
        <path class="lotus-path" d="M 50,90 C 40,50 40,20 50,10 C 60,20 60,50 50,90 Z M 50,90 C 30,80 15,65 10,40 C 25,45 40,65 50,90 Z M 50,90 C 70,80 85,65 90,40 C 75,45 60,65 50,90 Z M 50,90 C 20,95 0,85 0,60 C 20,70 35,85 50,90 Z M 50,90 C 80,95 100,85 100,60 C 80,70 65,85 50,90 Z" fill="none" stroke="url(#lotusGrad)" stroke-width="0.5" stroke-linecap="round" stroke-linejoin="round" />
        <defs>
          <linearGradient id="lotusGrad" x1="0%" y1="0%" x2="100%" y2="100%">
            <stop offset="0%" stop-color="#ffffff" />
            <stop offset="100%" stop-color="#9932cc" />
          </linearGradient>
        </defs>
      </svg>
    </div>

    <!-- Title that fades out -->
    <div class="absolute inset-0 flex flex-col items-center justify-center pointer-events-none transition-opacity duration-1000 z-10"
         :class="{ 'opacity-0': isTransitioning }">
      <h1 class="text-3xl md:text-5xl font-light tracking-[0.5em] uppercase text-white/70">Dive into VeeQ's Mind</h1>
      <p class="text-xs tracking-widest mt-4 text-[var(--color-orchid-light)] animate-pulse">Scroll or Click to discover the stories</p>
    </div>

    <!-- Entrance Button -->
    <transition name="fade">
      <div v-if="showEnterButton" 
           class="absolute inset-0 flex items-center justify-center pointer-events-none z-50">
        <button @click="$emit('enterSite')" 
                class="pointer-events-auto px-12 py-4 border border-white/30 rounded-full text-white tracking-[0.3em] uppercase hover:bg-white hover:text-black transition-all duration-500 hover:scale-105 backdrop-blur-sm cursor-none hover-target">
          Enter The Canvas
        </button>
      </div>
    </transition>
  </div>
</template>

<style scoped>
.fade-enter-active,
.fade-leave-active {
  transition: opacity 2s ease;
}
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>
