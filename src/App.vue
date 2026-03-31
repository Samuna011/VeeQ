<script setup>
import { ref, onMounted, onUnmounted, onErrorCaptured } from 'vue'
import Lenis from 'lenis'

import VeeQUniverse from './components/VeeQUniverse.vue'
import FallingPetals from './components/FallingPetals.vue'
import FeatureGallery from './components/FeatureGallery.vue'
import FooterSection from './components/FooterSection.vue'
import StoryReader from './components/StoryReader.vue'

const selectedStory = ref(null)
const appError = ref(null)
const scrollProgress = ref(0) // 0 (top, dark) to 1 (passed 100vh, light)

onErrorCaptured(() => {
  appError.value = true
  return false
})

let lenis

onMounted(() => {
  lenis = new Lenis({
    smoothWheel: true,
    lerp: 0.08,
  })
  
  lenis.on('scroll', (e) => {
    const vh = window.innerHeight
    let progress = e.scroll / (vh * 0.8) 
    if (progress < 0) progress = 0
    if (progress > 1) progress = 1
    scrollProgress.value = progress
  })

  function raf(t) {
    if (!lenis) return
    lenis.raf(t)
    requestAnimationFrame(raf)
  }
  requestAnimationFrame(raf)
  window.scrollTo(0, 0)
})

onUnmounted(() => {
  if (lenis) lenis.destroy()
})

const handleOpenStory = (story) => {
  selectedStory.value = story
}

const handleCloseStory = () => {
  selectedStory.value = null
}
</script>

<template>
  <div v-if="appError" class="fixed inset-0 flex flex-col items-center justify-center bg-[#fdfcfd] text-black p-8 text-center z-50">
    <p class="text-lg mb-4">Something went wrong.</p>
    <button @click="appError = null" class="px-6 py-2 border border-black/50 rounded-full uppercase tracking-wider text-sm hover:bg-black hover:text-white transition-colors">Try again</button>
  </div>

  <template v-else>
    <div class="app-wrapper min-h-screen relative" id="main-content" :style="{ backgroundColor: `rgba(238, 235, 242, ${scrollProgress})` }">
      
      <!-- Dark 3D Background (Fades out on scroll) -->
      <div class="fixed inset-0 z-0 bg-[#06010a] pointer-events-none" :style="{ opacity: 1 - scrollProgress }">
        <VeeQUniverse />
        
        <!-- Inviting CSS glowing orbs that sit behind the hero text to provide intense warmth and color -->
        <div class="absolute inset-0 flex items-center justify-center mix-blend-screen opacity-50 pointer-events-none">
          <div class="w-[80vw] h-[80vw] max-w-[800px] max-h-[800px] bg-[var(--color-orchid-light)] rounded-full blur-[120px] animate-pulse" style="animation-duration: 5s;"></div>
          <div class="absolute w-[60vw] h-[60vw] max-w-[600px] max-h-[600px] bg-[#f472b6] rounded-full blur-[100px] animate-pulse" style="animation-duration: 3s; animation-delay: 1.5s;"></div>
        </div>
      </div>

      <!-- Light Background with petals (Fades in on scroll) -->
      <!-- Made slightly darker per user request to provide contrast with white chapter cards -->
      <div class="fixed inset-0 z-0 bg-[#eeebf2] pointer-events-none transition-opacity" :style="{ opacity: scrollProgress }">
        <FallingPetals :is-paused="false" />
      </div>

      <!-- Content Layer -->
      <div class="relative z-10 w-full flex flex-col pointer-events-none">
        
        <!-- Hero Section (Dark Theme) -->
        <div class="h-screen w-full flex flex-col items-center justify-center pointer-events-none relative transition-transform" :style="{ opacity: 1 - scrollProgress, transform: `translateY(${scrollProgress * 50}px)` }">
          
          <!-- Added text shadow to make the white text pop on the bright colorful background -->
          <h1 class="text-6xl md:text-9xl font-light tracking-[0.2em] md:tracking-[0.4em] uppercase text-white drop-shadow-[0_0_25px_rgba(255,255,255,0.4)]" style="font-family: var(--font-display);">
            VeeQ
          </h1>
          <p class="mt-6 text-sm md:text-lg tracking-[0.3em] font-medium text-white/90 uppercase drop-shadow-md">
            Stories & Imagination
          </p>

          <div class="absolute bottom-12 flex flex-col items-center gap-2 text-white/70 animate-bounce">
            <span class="text-xs uppercase tracking-widest font-medium">Scroll</span>
            <svg class="w-4 h-4 text-white/70" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 14l-7 7m0 0l-7-7m7 7V3"></path></svg>
          </div>
        </div>

        <!-- Main Content (Light Theme) -->
        <div class="pointer-events-auto w-full">
          <FeatureGallery @openStory="handleOpenStory" />
          <FooterSection />
        </div>
      </div>

      <transition name="slide-up">
        <div v-if="selectedStory" class="fixed inset-0 z-[100] pointer-events-auto">
          <StoryReader :story="selectedStory" @close="handleCloseStory" />
        </div>
      </transition>
    </div>
  </template>
</template>

<style>
.slide-up-enter-active,
.slide-up-leave-active {
  transition: transform 0.8s cubic-bezier(0.8, 0, 0.2, 1), opacity 0.8s ease;
}

.slide-up-enter-from,
.slide-up-leave-to {
  transform: translateY(100%);
  opacity: 0;
}
</style>
