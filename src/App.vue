<script setup>
import { onMounted, ref, onUnmounted, nextTick } from 'vue'
import Lenis from 'lenis'

import OrchidUniverse from './components/OrchidUniverse.vue'
import FallingPetals from './components/FallingPetals.vue'
import FeatureGallery from './components/FeatureGallery.vue'
import FooterSection from './components/FooterSection.vue'
import StoryReader from './components/StoryReader.vue'

const isSiteEntered = ref(false)
const selectedStory = ref(null)

const handleOpenStory = (story) => {
  selectedStory.value = story
  if (lenis) {
    lenis.stop() // Pause background scrolling
  }
}

const handleCloseStory = () => {
  selectedStory.value = null
  if (lenis) {
    lenis.start() // Resume background scrolling
  }
}

// Custom Cursor Logic
const mouseX = ref(0)
const mouseY = ref(0)
const isHovering = ref(false)

const updateCursor = (e) => {
  mouseX.value = e.clientX
  mouseY.value = e.clientY
}

const handleHover = (e) => {
  if (e.target.closest('a, button, .hover-target')) {
    isHovering.value = true
  } else {
    isHovering.value = false
  }
}

let lenis

onMounted(() => {
  // Cursor Listeners
  window.addEventListener('mousemove', updateCursor)
  window.addEventListener('mouseover', handleHover)
})

onUnmounted(() => {
  if (lenis) lenis.destroy()
  window.removeEventListener('mousemove', updateCursor)
  window.removeEventListener('mouseover', handleHover)
})

const enterMainSite = async () => {
  isSiteEntered.value = true
  
  await nextTick()
  
  // Initialize Smooth Scrolling AFTER the main site is rendered
  lenis = new Lenis({
    smoothWheel: true,
    lerp: 0.08,
  })

  // Detect scroll-to-top reversal
  lenis.on('scroll', (e) => {
    // e.scroll is the current scroll position.
    // e.direction is -1 when scrolling up.
    if (e.scroll <= 0 && e.direction === -1) {
      exitMainSite()
    }
  })

  function raf(time) {
    if(!lenis) return
    lenis.raf(time)
    requestAnimationFrame(raf)
  }
  requestAnimationFrame(raf)
  
  window.scrollTo(0, 0)
}

const exitMainSite = () => {
  if (!isSiteEntered.value) return
  isSiteEntered.value = false
  
  if (lenis) {
    lenis.destroy()
    lenis = null
  }
}
</script>

<template>
  <!-- Dynamic cursor color based on state (white on dark universe, purple on white site) -->
  <div 
    class="custom-cursor fixed top-0 left-0 w-5 h-5 rounded-full pointer-events-none mix-blend-difference z-[9999] transition-transform duration-100 ease-out"
    :class="{ 
      'active': isHovering,
      'bg-white': !isSiteEntered,
      'bg-[var(--color-orchid-dark)] mix-blend-normal': isSiteEntered
    }"
    :style="{ transform: `translate(${mouseX}px, ${mouseY}px) scale(${isHovering ? 2 : 1})` }"
  ></div>

  <!-- Hero Universe View -->
  <transition name="fade">
    <div v-if="!isSiteEntered" class="fixed inset-0 z-50">
      <OrchidUniverse @enterSite="enterMainSite" />
    </div>
  </transition>

  <!-- Main White Site View -->
  <div v-if="isSiteEntered" class="app-wrapper bg-white min-h-screen text-black overflow-hidden relative transition-colors duration-1000">
    <FallingPetals />
    
    <!-- Render main content above the canvas -->
    <div class="relative z-10 w-full">
      <FeatureGallery @openStory="handleOpenStory" />
      <FooterSection />
    </div>

    <!-- Story Reader Overlay -->
    <transition name="slide-up">
      <StoryReader v-if="selectedStory" :story="selectedStory" @close="handleCloseStory" />
    </transition>
  </div>
</template>

<style>
.fade-enter-active,
.fade-leave-active {
  transition: opacity 1.5s ease-in-out;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

.slide-up-enter-active,
.slide-up-leave-active {
  transition: transform 0.8s cubic-bezier(0.8, 0, 0.2, 1);
}

.slide-up-enter-from {
  transform: translateY(100%);
}

.slide-up-leave-to {
  transform: translateY(100%);
}
</style>
