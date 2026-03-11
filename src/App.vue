<script setup>
import { defineAsyncComponent, ref, onUnmounted, nextTick, onErrorCaptured } from 'vue'
import Lenis from 'lenis'

const OrchidUniverse = defineAsyncComponent({
  loader: () => import('./components/OrchidUniverse.vue'),
  loadingComponent: { template: '<div class="fixed inset-0 bg-[#020003] flex items-center justify-center text-white/60 text-sm tracking-widest">Loading...</div>' },
  errorComponent: { template: '<div class="fixed inset-0 bg-[#020003] flex flex-col items-center justify-center text-white/80 p-8"><p class="text-lg mb-4">Something went wrong.</p><button onclick="location.reload()" class="px-6 py-2 border border-white/40 rounded-full text-sm uppercase tracking-wider">Reload</button></div>' },
  delay: 200,
})
import FallingPetals from './components/FallingPetals.vue'
import FeatureGallery from './components/FeatureGallery.vue'
import FooterSection from './components/FooterSection.vue'
import StoryReader from './components/StoryReader.vue'

const isSiteEntered = ref(false)
const isTransitioningToSite = ref(false)
const selectedStory = ref(null)
const appError = ref(null)

onErrorCaptured(() => {
  appError.value = true
  return false
})

const handleOpenStory = (story) => {
  selectedStory.value = story
  if (lenis) lenis.stop()
}

const handleCloseStory = () => {
  selectedStory.value = null
  if (lenis) lenis.start()
}

let lenis

onUnmounted(() => {
  if (lenis) lenis.destroy()
})

const enterMainSite = async () => {
  isTransitioningToSite.value = true
  await new Promise(r => setTimeout(r, 700))
  isSiteEntered.value = true
  isTransitioningToSite.value = false
  await nextTick()

  lenis = new Lenis({
    smoothWheel: true,
    lerp: 0.08,
  })
  function raf(t) {
    if (!lenis) return
    lenis.raf(t)
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
  <!-- Error fallback -->
  <div v-if="appError" class="fixed inset-0 flex flex-col items-center justify-center bg-[#020003] text-white p-8 text-center">
    <p class="text-lg mb-4">Something went wrong.</p>
    <button @click="appError = null" class="px-6 py-2 border border-white/50 rounded-full uppercase tracking-wider text-sm">Try again</button>
  </div>

  <template v-else>
    <!-- Hero Universe View -->
    <transition name="fade">
      <div v-if="!isSiteEntered && !isTransitioningToSite" class="fixed inset-0 z-50">
        <OrchidUniverse @enterSite="enterMainSite" />
      </div>
    </transition>

    <!-- Page transition overlay -->
    <transition name="fade">
      <div v-if="isTransitioningToSite" class="fixed inset-0 bg-white z-[60] pointer-events-none" aria-hidden="true" />
    </transition>

    <!-- Main White Site View -->
    <main v-if="isSiteEntered" class="app-wrapper bg-white min-h-screen text-black overflow-hidden relative transition-colors duration-1000" id="main-content">
      <FallingPetals :is-paused="!!selectedStory" />
      <div class="relative z-10 w-full">
        <FeatureGallery @openStory="handleOpenStory" />
        <FooterSection />
      </div>
      <transition name="slide-up">
        <StoryReader v-if="selectedStory" :story="selectedStory" @close="handleCloseStory" />
      </transition>
    </main>
  </template>
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
