<script setup>
import { onMounted, onUnmounted } from 'vue'
import gsap from 'gsap'

const props = defineProps({
  story: Object
})
const emit = defineEmits(['close'])

onMounted(() => {
  document.body.style.overflow = 'hidden'
  gsap.fromTo('.story-reader-content', { y: 30, opacity: 0 }, { y: 0, opacity: 1, duration: 0.6, ease: 'power3.out' })
  gsap.fromTo('.story-reader-header', { y: 20, opacity: 0 }, { y: 0, opacity: 1, duration: 0.5, ease: 'power3.out', delay: 0.1 })
  gsap.fromTo('.story-reader-prose p', { y: 24, opacity: 0 }, { y: 0, opacity: 1, duration: 0.5, stagger: 0.08, delay: 0.2, ease: 'power2.out' })
})

onUnmounted(() => {
  document.body.style.overflow = ''
})

function closeReader() {
  emit('close')
}
</script>

<template>
  <div class="fixed inset-0 z-[100] bg-[#faf9fc] text-black overflow-y-auto w-screen h-screen">
    <div class="absolute inset-0 bg-gradient-to-b from-[var(--color-orchid-dark)]/5 to-transparent pointer-events-none"></div>

    <button
      @click="closeReader"
      class="fixed top-8 right-8 z-[110] px-6 py-3 border-2 border-black/10 rounded-full text-xs tracking-[0.2em] uppercase hover:bg-black hover:text-white hover:border-black transition-all duration-300 bg-white/90 backdrop-blur-md font-medium"
    >
      Close
    </button>

    <div class="story-reader-content max-w-3xl mx-auto py-32 px-8 relative">
      <div class="story-reader-header mb-20 text-center border-b border-gray-200/80 pb-14">
        <span class="text-[var(--color-orchid-dark)] text-sm font-bold tracking-[0.3em] uppercase block mb-5">Story {{ story.number }}</span>
        <h1 class="text-5xl md:text-7xl font-black uppercase tracking-tighter mb-8 text-black" style="font-family: var(--font-display); letter-spacing: -0.02em;">
          {{ story.title }}
        </h1>
        <p class="text-xl text-gray-500 font-light italic max-w-xl mx-auto leading-relaxed" style="font-family: var(--font-serif);">
          {{ story.description }}
        </p>
      </div>

      <div class="story-reader-prose prose prose-xl md:prose-2xl mx-auto text-gray-800 leading-[2] md:leading-[2.2] space-y-8 font-light" style="font-family: var(--font-serif);">
        <p class="first-letter:text-7xl first-letter:font-bold first-letter:text-[var(--color-orchid-dark)] first-letter:mr-3 first-letter:float-left first-line:uppercase first-line:tracking-widest">
          The pages of this script lie unwritten, waiting for VeeQ to pour her imagination into reality. Her mind races with the possibilities of what could be, painting worlds built on profound emotion and absolute contrast.
        </p>
        <p>This is merely the canvas. The real narrative begins when the words finally fall like dark petals onto the stark white expanse, bringing life, structure, and intense meaning to the void.</p>
        <p class="text-center italic mt-16 text-gray-400" style="font-family: var(--font-serif);">
          More from <strong class="text-gray-600">{{ story.title }}</strong> coming soon.
        </p>
      </div>
    </div>
  </div>
</template>

<style scoped>
p {
  margin-bottom: 2rem;
}
</style>
