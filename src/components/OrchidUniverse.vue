<script setup>
import { onMounted, ref, onUnmounted } from 'vue'
import * as THREE from 'three'
import gsap from 'gsap'

const emit = defineEmits(['enterSite'])

const canvasContainer = ref(null)
const showEnterButton = ref(false)

let scene, camera, renderer, particles, particlesFar
let animationId
let isTransitioning = false
let time = 0
let mouseX = 0
let mouseY = 0

let targetSpeed = 0.5
let currentSpeed = 0.5
let warpComplete = false
const isMobile = typeof window !== 'undefined' && window.innerWidth < 768
const prefersReducedMotion = typeof window !== 'undefined' && window.matchMedia('(prefers-reduced-motion: reduce)').matches
const starCount = isMobile ? 8000 : 28000
const starCountFar = isMobile ? 4000 : 14000

onMounted(() => {
  initThree()
  createUniverse()
  animate()
  window.addEventListener('wheel', handleInteraction, { once: true })
  window.addEventListener('click', handleInteraction, { once: true })
  window.addEventListener('resize', onWindowResize)
  window.addEventListener('mousemove', onMouseMove)
})

onUnmounted(() => {
  cancelAnimationFrame(animationId)
  if (renderer) renderer.dispose()
  window.removeEventListener('wheel', handleInteraction)
  window.removeEventListener('click', handleInteraction)
  window.removeEventListener('resize', onWindowResize)
  window.removeEventListener('mousemove', onMouseMove)
})

function onMouseMove(e) {
  mouseX = (e.clientX / window.innerWidth) * 2 - 1
  mouseY = -(e.clientY / window.innerHeight) * 2 + 1
}

function initThree() {
  scene = new THREE.Scene()
  scene.background = new THREE.Color('#020003')
  scene.fog = new THREE.FogExp2('#020003', 0.0012)

  camera = new THREE.PerspectiveCamera(58, window.innerWidth / window.innerHeight, 0.1, 2000)
  camera.position.z = 0

  renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true })
  renderer.setSize(window.innerWidth, window.innerHeight)
  renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))
  renderer.toneMapping = THREE.ACESFilmicToneMapping
  renderer.toneMappingExposure = 0.9
  canvasContainer.value.appendChild(renderer.domElement)
}

function createSoftStarTexture() {
  const canvas = document.createElement('canvas')
  const size = 64
  canvas.width = size
  canvas.height = size
  const ctx = canvas.getContext('2d')
  const gradient = ctx.createRadialGradient(size/2, size/2, 0, size/2, size/2, size/2)
  gradient.addColorStop(0, 'rgba(255,255,255,0.95)')
  gradient.addColorStop(0.25, 'rgba(255,255,255,0.4)')
  gradient.addColorStop(0.5, 'rgba(200,150,255,0.1)')
  gradient.addColorStop(1, 'rgba(255,255,255,0)')
  ctx.fillStyle = gradient
  ctx.fillRect(0, 0, size, size)
  return new THREE.CanvasTexture(canvas)
}

function createUniverse() {
  const softTexture = createSoftStarTexture()
  const color1 = new THREE.Color('#ffffff')
  const color2 = new THREE.Color('#6b21a8')
  const color3 = new THREE.Color('#a855f7')
  const color4 = new THREE.Color('#3b0764')
  const color5 = new THREE.Color('#1e0333')

  // Main starfield — closer, brighter
  const geometry = new THREE.BufferGeometry()
  const positions = new Float32Array(starCount * 3)
  const colors = new Float32Array(starCount * 3)
  const randoms = new Float32Array(starCount) // for twinkle phase

  for (let i = 0; i < starCount; i++) {
    positions[i * 3] = (Math.random() - 0.5) * 2200
    positions[i * 3 + 1] = (Math.random() - 0.5) * 2200
    positions[i * 3 + 2] = (Math.random() - 1.0) * 3800

    const r = Math.random()
    let c
    if (r > 0.75) c = color1
    else if (r > 0.45) c = color2
    else if (r > 0.2) c = color3
    else c = color4
    colors[i * 3] = c.r
    colors[i * 3 + 1] = c.g
    colors[i * 3 + 2] = c.b
    randoms[i] = Math.random() * Math.PI * 2
  }

  geometry.setAttribute('position', new THREE.BufferAttribute(positions, 3))
  geometry.setAttribute('color', new THREE.BufferAttribute(colors, 3))
  geometry.setAttribute('phase', new THREE.BufferAttribute(randoms, 1))

  const material = new THREE.PointsMaterial({
    size: 3.5,
    vertexColors: true,
    blending: THREE.AdditiveBlending,
    transparent: true,
    opacity: 0.85,
    sizeAttenuation: true,
    map: softTexture,
    depthWrite: false,
  })
  particles = new THREE.Points(geometry, material)
  scene.add(particles)

  // Far layer — smaller, dimmer, more purple
  const geometryFar = new THREE.BufferGeometry()
  const posFar = new Float32Array(starCountFar * 3)
  const colFar = new Float32Array(starCountFar * 3)
  for (let i = 0; i < starCountFar; i++) {
    posFar[i * 3] = (Math.random() - 0.5) * 2400
    posFar[i * 3 + 1] = (Math.random() - 0.5) * 2400
    posFar[i * 3 + 2] = (Math.random() - 1.0) * 4500
    const r = Math.random()
    const c = r > 0.5 ? color4 : color5
    colFar[i * 3] = c.r
    colFar[i * 3 + 1] = c.g
    colFar[i * 3 + 2] = c.b
  }
  geometryFar.setAttribute('position', new THREE.BufferAttribute(posFar, 3))
  geometryFar.setAttribute('color', new THREE.BufferAttribute(colFar, 3))
  particlesFar = new THREE.Points(geometryFar, new THREE.PointsMaterial({
    size: 1.8,
    vertexColors: true,
    blending: THREE.AdditiveBlending,
    transparent: true,
    opacity: 0.5,
    sizeAttenuation: true,
    map: softTexture,
    depthWrite: false,
  }))
  scene.add(particlesFar)
}

function handleInteraction() {
  if (isTransitioning) return
  isTransitioning = true

  gsap.set('.lotus-path', { strokeDasharray: 1000, strokeDashoffset: 1000 })

  const speedProxy = { val: currentSpeed }
  const dur = prefersReducedMotion ? 0.6 : 1.2
  const dur2 = prefersReducedMotion ? 0.4 : 0.8
  const dur3 = prefersReducedMotion ? 1 : 2
  const dur4 = prefersReducedMotion ? 1 : 1.8

  const tl = gsap.timeline({
    onComplete: () => {
      warpComplete = true
      showEnterButton.value = true
      if (!prefersReducedMotion) {
        gsap.to('.lily-glow', { opacity: 0.5, duration: 2, repeat: -1, yoyo: true, ease: 'sine.inOut' })
      }
    }
  })

  tl.to(camera, {
    fov: 100,
    duration: dur,
    ease: 'power2.in',
    onUpdate: () => camera.updateProjectionMatrix()
  })
  tl.to(speedProxy, {
    val: 50,
    duration: dur,
    ease: 'power3.in',
    onUpdate: () => { targetSpeed = speedProxy.val }
  }, '<')
  tl.to({}, { duration: dur2 })
  tl.to(speedProxy, {
    val: 0.8,
    duration: dur3,
    ease: 'power2.out',
    onUpdate: () => { targetSpeed = speedProxy.val }
  }, prefersReducedMotion ? '-=0.5' : '-=1.2')
  tl.to(camera, {
    fov: 58,
    duration: dur3,
    ease: 'power2.out',
    onUpdate: () => camera.updateProjectionMatrix()
  }, '<')
  tl.to('#lotus-container', {
    opacity: 1,
    scale: 1,
    duration: dur3,
    ease: 'power2.out'
  }, '<')
  tl.to('.lotus-path', {
    strokeDashoffset: 0,
    duration: dur4,
    ease: 'power2.inOut'
  }, prefersReducedMotion ? '-=0.8' : '-=1.4')
  tl.to(camera.rotation, {
    z: Math.PI / 6,
    x: 0.05,
    y: 0.05,
    duration: dur4,
    ease: 'power2.out'
  }, '<')
}

function onWindowResize() {
  camera.aspect = window.innerWidth / window.innerHeight
  camera.updateProjectionMatrix()
  renderer.setSize(window.innerWidth, window.innerHeight)
}

function animate() {
  animationId = requestAnimationFrame(animate)
  time += 0.014
  currentSpeed += (targetSpeed - currentSpeed) * 0.09

  // Subtle mouse parallax only before warp completes (so we don't override lily view rotation)
  if (!warpComplete && currentSpeed < 3) {
    const parallax = 12
    camera.position.x += (mouseX * parallax - camera.position.x) * 0.03
    camera.position.y += (mouseY * parallax - camera.position.y) * 0.03
    camera.lookAt(0, 0, 0)
  }

  // Twinkling: subtle global pulse (skip when user prefers reduced motion)
  if (!prefersReducedMotion && particles && currentSpeed < 2) {
    const pulse = 0.72 + 0.18 * Math.sin(time * 1.2)
    particles.material.opacity = pulse
  }
  if (!prefersReducedMotion && particlesFar && currentSpeed < 2) {
    const pulse = 0.4 + 0.12 * Math.sin(time * 0.9 + 1)
    particlesFar.material.opacity = pulse
  }

  if (particles) {
    const positions = particles.geometry.attributes.position.array
    for (let i = 0; i < starCount; i++) {
      positions[i * 3 + 2] += currentSpeed
      if (currentSpeed > 4) {
        const x = positions[i * 3]
        const y = positions[i * 3 + 1]
        const angle = currentSpeed * 0.00012
        const cos = Math.cos(angle)
        const sin = Math.sin(angle)
        positions[i * 3] = x * cos - y * sin
        positions[i * 3 + 1] = x * sin + y * cos
      }
      if (positions[i * 3 + 2] > 200) {
        positions[i * 3 + 2] = -4000
        positions[i * 3] = (Math.random() - 0.5) * 2200
        positions[i * 3 + 1] = (Math.random() - 0.5) * 2200
      }
    }
    particles.geometry.attributes.position.needsUpdate = true
  }

  if (particlesFar) {
    const positions = particlesFar.geometry.attributes.position.array
    for (let i = 0; i < starCountFar; i++) {
      positions[i * 3 + 2] += currentSpeed * 0.7
      if (positions[i * 3 + 2] > 250) {
        positions[i * 3 + 2] = -4500
        positions[i * 3] = (Math.random() - 0.5) * 2400
        positions[i * 3 + 1] = (Math.random() - 0.5) * 2400
      }
    }
    particlesFar.geometry.attributes.position.needsUpdate = true
  }

  renderer.render(scene, camera)
}
</script>

<template>
  <div class="relative w-screen h-screen overflow-hidden bg-[#020003] text-white">
    <div ref="canvasContainer" class="absolute inset-0"></div>

    <div class="absolute inset-0 z-0 mix-blend-screen pointer-events-none transition-opacity duration-[3000ms] ease-out"
         :class="showEnterButton ? 'opacity-40' : 'opacity-0'"
         style="background: radial-gradient(ellipse 80% 80% at 50% 50%, #a855f7 0%, #4c1d95 30%, transparent 60%);">
    </div>

    <div id="lotus-container" class="absolute inset-0 flex flex-col items-center justify-center pointer-events-none z-20 opacity-0 scale-50 transition-transform duration-1000" style="filter: drop-shadow(0 0 25px rgba(168, 85, 247, 0.5));">
      <div class="lily-glow absolute inset-0 flex items-center justify-center opacity-0 pointer-events-none" style="background: radial-gradient(circle, rgba(168,85,247,0.15) 0%, transparent 60%); width: 120%; height: 120%;"></div>
      <svg width="420" height="420" viewBox="0 0 100 100" class="overflow-visible relative shrink-0">
        <defs>
          <linearGradient id="lotusGrad" x1="0%" y1="0%" x2="100%" y2="100%">
            <stop offset="0%" stop-color="#ffffff" />
            <stop offset="50%" stop-color="#e9d5ff" />
            <stop offset="100%" stop-color="#a855f7" />
          </linearGradient>
          <filter id="lily-glow">
            <feGaussianBlur stdDeviation="0.4" result="blur" />
            <feMerge>
              <feMergeNode in="blur" />
              <feMergeNode in="SourceGraphic" />
            </feMerge>
          </filter>
        </defs>
        <g filter="url(#lily-glow)">
          <path class="lotus-path" transform="rotate(0 50 50)"   d="M 50,50 C 28,42 18,18 50,6 C 82,18 72,42 50,50 Z" fill="none" stroke="url(#lotusGrad)" stroke-width="0.6" stroke-linecap="round" stroke-linejoin="round" />
          <path class="lotus-path" transform="rotate(60 50 50)"  d="M 50,50 C 28,42 18,18 50,6 C 82,18 72,42 50,50 Z" fill="none" stroke="url(#lotusGrad)" stroke-width="0.6" stroke-linecap="round" stroke-linejoin="round" />
          <path class="lotus-path" transform="rotate(120 50 50)" d="M 50,50 C 28,42 18,18 50,6 C 82,18 72,42 50,50 Z" fill="none" stroke="url(#lotusGrad)" stroke-width="0.6" stroke-linecap="round" stroke-linejoin="round" />
          <path class="lotus-path" transform="rotate(180 50 50)" d="M 50,50 C 28,42 18,18 50,6 C 82,18 72,42 50,50 Z" fill="none" stroke="url(#lotusGrad)" stroke-width="0.6" stroke-linecap="round" stroke-linejoin="round" />
          <path class="lotus-path" transform="rotate(240 50 50)" d="M 50,50 C 28,42 18,18 50,6 C 82,18 72,42 50,50 Z" fill="none" stroke="url(#lotusGrad)" stroke-width="0.6" stroke-linecap="round" stroke-linejoin="round" />
          <path class="lotus-path" transform="rotate(300 50 50)" d="M 50,50 C 28,42 18,18 50,6 C 82,18 72,42 50,50 Z" fill="none" stroke="url(#lotusGrad)" stroke-width="0.6" stroke-linecap="round" stroke-linejoin="round" />
        </g>
      </svg>
      <transition name="fade">
        <div v-if="showEnterButton" class="mt-12 pointer-events-auto">
          <button @click="$emit('enterSite')"
                  class="hero-enter-btn">
            Enter
          </button>
        </div>
      </transition>
    </div>

    <div class="absolute inset-0 flex flex-col items-center justify-center pointer-events-none transition-opacity duration-1000 z-10"
         :class="{ 'opacity-0': isTransitioning }">
      <h1 class="hero-title text-3xl md:text-6xl font-light tracking-[0.4em] md:tracking-[0.5em] uppercase text-white/90">
        <span class="inline-block hero-word">Dive into</span>
        <span class="inline-block hero-word mt-2 md:mt-3" style="animation-delay: 0.1s">VeeQ's Mind</span>
      </h1>
      <p class="text-xs md:text-sm tracking-[0.35em] mt-6 text-[var(--color-orchid-light)]/90 animate-pulse" style="animation-duration: 2.5s">Scroll or click to discover</p>
    </div>
  </div>
</template>

<style scoped>
.hero-word {
  opacity: 0;
  transform: translateY(20px);
  animation: heroReveal 1.2s cubic-bezier(0.22, 1, 0.36, 1) forwards;
}
.hero-word:nth-child(2) {
  animation-delay: 0.15s;
}
@keyframes heroReveal {
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.hero-enter-btn {
  font-size: 0.75rem;
  letter-spacing: 0.4em;
  text-transform: uppercase;
  color: rgba(255, 255, 255, 0.9);
  background: transparent;
  border: none;
  padding: 0.6rem 0 0.6rem 0.2rem;
  position: relative;
  transition: color 0.25s ease;
}
.hero-enter-btn::after {
  content: '';
  position: absolute;
  left: 0;
  bottom: 0;
  width: 100%;
  height: 1px;
  background: rgba(255, 255, 255, 0.5);
  transform: scaleX(1);
  transform-origin: left;
  transition: transform 0.3s ease, background 0.25s ease;
}
.hero-enter-btn:hover {
  color: #fff;
}
.hero-enter-btn:hover::after {
  transform: scaleX(1.05);
  background: rgba(255, 255, 255, 0.9);
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 1.2s ease;
}
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>
