<script setup>
import { onMounted, onUnmounted } from 'vue'
import gsap from 'gsap'

const props = defineProps({
  story: Object
})
const emit = defineEmits(['close'])

onMounted(() => {
  gsap.fromTo('.story-reader-content', { y: 40, opacity: 0 }, { y: 0, opacity: 1, duration: 0.8, ease: 'power3.out' })
  gsap.fromTo('.story-reader-header', { y: 20, opacity: 0 }, { y: 0, opacity: 1, duration: 0.8, ease: 'power3.out', delay: 0.1 })
  gsap.fromTo('.story-reader-prose p', { y: 24, opacity: 0 }, { y: 0, opacity: 1, duration: 0.6, stagger: 0.1, delay: 0.2, ease: 'power2.out' })
})

onUnmounted(() => {
})

function closeReader() {
  gsap.to('.story-reader-content', { y: 20, opacity: 0, duration: 0.4, ease: 'power2.in' })
  setTimeout(() => {
    emit('close')
  }, 400)
}
</script>

<template>
  <div data-lenis-prevent class="fixed inset-0 z-[100] bg-[#faf9fc]/90 backdrop-blur-3xl text-black overflow-y-auto w-screen h-screen">
    <div class="absolute inset-0 bg-gradient-to-b from-[var(--color-orchid-light)]/5 to-transparent pointer-events-none"></div>

    <button
      @click="closeReader"
      class="fixed top-6 right-6 md:top-10 md:right-10 z-[110] px-6 py-3 border border-black/10 rounded-full text-xs tracking-[0.2em] uppercase hover:bg-black hover:border-black hover:text-white transition-all duration-300 bg-white/60 backdrop-blur-lg font-medium text-black/90"
    >
      Close
    </button>

    <div class="story-reader-content max-w-3xl mx-auto py-32 px-6 md:px-8 relative z-10">
      <div class="story-reader-header mb-20 text-center border-b border-black/10 pb-16">
        <span class="text-[var(--color-orchid-dark)] text-sm font-bold tracking-[0.4em] uppercase block mb-6">Chapter {{ story.number }}</span>
        <h1 class="text-5xl md:text-7xl lg:text-8xl font-light uppercase tracking-widest mb-10 text-black" style="font-family: var(--font-display);">
          {{ story.title }}
        </h1>
        <p class="text-lg md:text-xl text-black/50 font-light italic max-w-xl mx-auto leading-relaxed" style="font-family: var(--font-serif);">
          {{ story.description }}
        </p>
      </div>

      <div class="story-reader-prose prose prose-lg md:prose-2xl mx-auto text-black/80 leading-[2.2] space-y-10 font-light" style="font-family: var(--font-serif);">
        <p class="first-letter:text-7xl first-letter:font-light first-letter:text-[var(--color-orchid-dark)] first-letter:mr-4 first-letter:float-left first-line:uppercase first-line:tracking-[0.2em]">
          The pages of this script lie unwritten, waiting for VeeQ to pour her imagination into reality. Her mind races with the possibilities of what could be, painting worlds built on profound emotion and absolute contrast.
        </p>
        <p>This is merely the canvas. The real narrative begins when the words finally fall like dark petals onto the stark white expanse, bringing life, structure, and intense meaning to the void. She explores surreal romance, sci-fi drama, and the fragile boundaries in between.</p>
        <p class="text-center italic mt-20 text-black/40 text-base" style="font-family: var(--font-serif);">
          More from <strong class="text-black/60">{{ story.title }}</strong> coming soon.
        </p>
      </div>
    </div>
  </div>
</template>

<style scoped>
.story-reader-prose p {
  margin-bottom: 2.5rem;
}
</style>
