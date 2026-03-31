<script setup>
import { onMounted, ref, onUnmounted } from 'vue'
import * as THREE from 'three'

const canvasContainer = ref(null)

let scene, camera, renderer, particles
let animationId
let mouseX = 0
let mouseY = 0
let scrollY = 0
let targetScrollY = 0
let time = 0

const isMobile = typeof window !== 'undefined' && window.innerWidth < 768

onMounted(() => {
  initThree()
  createUniverse()
  animate()
  window.addEventListener('resize', onWindowResize)
  window.addEventListener('mousemove', onMouseMove)
  window.addEventListener('scroll', onScroll, { passive: true })
})

onUnmounted(() => {
  cancelAnimationFrame(animationId)
  if (renderer) renderer.dispose()
  window.removeEventListener('resize', onWindowResize)
  window.removeEventListener('mousemove', onMouseMove)
  window.removeEventListener('scroll', onScroll)
})

function onMouseMove(e) {
  mouseX = (e.clientX / window.innerWidth) * 2 - 1
  mouseY = -(e.clientY / window.innerHeight) * 2 + 1
}

function onScroll() {
  targetScrollY = window.scrollY
}

function initThree() {
  scene = new THREE.Scene()

  camera = new THREE.PerspectiveCamera(isMobile ? 70 : 60, window.innerWidth / window.innerHeight, 0.1, 4000)
  camera.position.z = 600
  camera.position.y = 0

  renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true })
  renderer.setSize(window.innerWidth, window.innerHeight)
  renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))
  canvasContainer.value.appendChild(renderer.domElement)
}

function createTexturedStar() {
  const canvas = document.createElement('canvas')
  canvas.width = 64
  canvas.height = 64
  const ctx = canvas.getContext('2d')
  const gradient = ctx.createRadialGradient(32, 32, 0, 32, 32, 32)
  gradient.addColorStop(0, 'rgba(255,255,255,1)')
  gradient.addColorStop(0.2, 'rgba(233,213,255,0.8)') // Light pink/purple
  gradient.addColorStop(0.5, 'rgba(192,132,252,0.4)') 
  gradient.addColorStop(1, 'rgba(0,0,0,0)')
  ctx.fillStyle = gradient
  ctx.fillRect(0,0,64,64)
  return new THREE.CanvasTexture(canvas)
}

function createUniverse() {
  // A huge dense cluster to look like a magical morphing star/nebula
  const particleCount = isMobile ? 8000 : 25000
  const geometry = new THREE.BufferGeometry()
  const positions = new Float32Array(particleCount * 3)
  const origPositions = new Float32Array(particleCount * 3)
  const colors = new Float32Array(particleCount * 3)
  const sizes = new Float32Array(particleCount)
  const phases = new Float32Array(particleCount)

  const colorPrimary = new THREE.Color('#ffffff')
  const colorSecondary = new THREE.Color('#e9d5ff')
  const colorAccent = new THREE.Color('#d8b4fe')
  const colorDark = new THREE.Color('#c084fc')

  for(let i=0; i<particleCount; i++) {
    // Generate particles in a spherical formation
    const radius = 220 + Math.random() * 80
    // Fibonacci sphere distribution for a mathematically perfect star core
    const phi = Math.acos(-1 + (2 * i) / particleCount)
    const theta = Math.sqrt(particleCount * Math.PI) * phi
    
    // Cloud scattering
    const scatter = Math.random() * 60
    
    const x = radius * Math.cos(theta) * Math.sin(phi) + (Math.random()-0.5) * scatter
    const y = radius * Math.sin(theta) * Math.sin(phi) + (Math.random()-0.5) * scatter
    const z = radius * Math.cos(phi) + (Math.random()-0.5) * scatter

    positions[i*3] = x
    positions[i*3+1] = y
    positions[i*3+2] = z

    origPositions[i*3] = x
    origPositions[i*3+1] = y
    origPositions[i*3+2] = z

    const r = Math.random()
    let mixColor
    if (r > 0.8) mixColor = colorPrimary
    else if (r > 0.5) mixColor = colorSecondary
    else if (r > 0.2) mixColor = colorAccent
    else mixColor = colorDark
    
    colors[i*3] = mixColor.r
    colors[i*3+1] = mixColor.g
    colors[i*3+2] = mixColor.b

    sizes[i] = Math.random() * 3.5 + 2.0
    phases[i] = Math.random() * Math.PI * 2
  }

  geometry.setAttribute('position', new THREE.BufferAttribute(positions, 3))
  geometry.setAttribute('origPosition', new THREE.BufferAttribute(origPositions, 3))
  geometry.setAttribute('color', new THREE.BufferAttribute(colors, 3))
  geometry.setAttribute('size', new THREE.BufferAttribute(sizes, 1))
  geometry.setAttribute('phase', new THREE.BufferAttribute(phases, 1))

  const material = new THREE.ShaderMaterial({
    uniforms: {
      time: { value: 0 },
      pointTexture: { value: createTexturedStar() }
    },
    vertexShader: `
      attribute float size;
      attribute float phase;
      attribute vec3 origPosition;
      
      varying vec3 vColor;
      varying float vPhase;
      uniform float time;
      
      void main() {
        vColor = color;
        vPhase = phase;
        
        vec3 pos = origPosition;
        
        // Fluid organic displacement (simulate a pulsing plasma ball)
        float noise = sin(origPosition.x * 0.015 + time) * cos(origPosition.y * 0.015 + time) * sin(origPosition.z * 0.015 + time);
        pos += normalize(origPosition) * noise * 40.0;
        
        // Complex majestic rotation right in the shader
        float s = sin(time * 0.15);
        float c = cos(time * 0.15);
        
        float xNew = pos.x * c - pos.z * s;
        float zNew = pos.x * s + pos.z * c;
        
        vec3 rotated = vec3(xNew, pos.y, zNew);
        
        vec4 mvPosition = modelViewMatrix * vec4(rotated, 1.0);
        
        // Shrink slightly but aggressively as it displaces
        gl_PointSize = size * (400.0 / -mvPosition.z) * (1.0 + noise * 0.6);
        gl_Position = projectionMatrix * mvPosition;
      }
    `,
    fragmentShader: `
      uniform float time;
      uniform sampler2D pointTexture;
      varying vec3 vColor;
      varying float vPhase;
      void main() {
        float alpha = 0.5 + 0.5 * sin(time * 2.0 + vPhase);
        vec4 texColor = texture2D(pointTexture, gl_PointCoord);
        gl_FragColor = vec4(vColor, alpha) * texColor;
      }
    `,
    blending: THREE.AdditiveBlending,
    depthTest: false,
    transparent: true,
    vertexColors: true
  })

  particles = new THREE.Points(geometry, material)
  scene.add(particles)
}

function onWindowResize() {
  camera.aspect = window.innerWidth / window.innerHeight
  camera.updateProjectionMatrix()
  renderer.setSize(window.innerWidth, window.innerHeight)
}

function animate() {
  animationId = requestAnimationFrame(animate)
  time += 0.01

  scrollY += (targetScrollY - scrollY) * 0.08

  if (particles) {
    particles.material.uniforms.time.value = time
  }

  // Camera subtly reacts to mouse to look around the beautiful star
  const targetCamX = mouseX * 40
  const targetCamY = mouseY * 40 - scrollY * 1.5 

  camera.position.x += (targetCamX - camera.position.x) * 0.05
  camera.position.y += (targetCamY - camera.position.y) * 0.05
  
  // Plunge deeper as you scroll
  camera.position.z = 600 - scrollY * 0.6
  
  camera.lookAt(0, -scrollY * 0.2, 0)

  renderer.render(scene, camera)
}
</script>

<template>
  <div class="fixed inset-0 z-0 h-screen w-screen overflow-hidden pointer-events-none">
    <div ref="canvasContainer" class="absolute inset-0 w-full h-full pointer-events-none mix-blend-screen"></div>
    <div class="absolute inset-0 bg-[#06010a]/10 pointer-events-none mix-blend-multiply"></div>
  </div>
</template>

<style scoped>
</style>
