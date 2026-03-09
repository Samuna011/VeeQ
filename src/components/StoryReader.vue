<script setup>
import { onMounted, onUnmounted } from 'vue'

const props = defineProps({
  story: Object
})
const emit = defineEmits(['close'])

onMounted(() => {
  // Disable main page scrolling
  document.body.style.overflow = 'hidden'
})

onUnmounted(() => {
  document.body.style.overflow = ''
})

function closeReader() {
  emit('close')
}
</script>

<template>
  <div class="fixed inset-0 z-[100] bg-[#fbfbfd] text-black overflow-y-auto w-screen h-screen">
    <!-- Close Button -->
    <button @click="closeReader" 
            class="fixed top-8 right-8 z-[110] px-6 py-2 border border-black/20 rounded-full text-xs tracking-widest uppercase hover:bg-black hover:text-white transition-colors duration-300 hover-target cursor-none bg-white/80 backdrop-blur-sm">
      Close
    </button>
    
    <div class="max-w-3xl mx-auto py-32 px-8">
      <div class="mb-20 text-center border-b border-gray-200 pb-12">
        <span class="text-[var(--color-orchid-dark)] text-sm font-bold tracking-[0.3em] uppercase block mb-4">Story {{ story.number }}</span>
        <h1 class="text-5xl md:text-7xl font-black uppercase tracking-tighter mb-8">{{ story.title }}</h1>
        <p class="text-xl text-gray-500 font-light italic max-w-xl mx-auto leading-relaxed">{{ story.description }}</p>
      </div>
      
      <div class="prose prose-xl md:prose-2xl mx-auto font-serif text-gray-800 leading-[2] md:leading-[2.2] space-y-8 font-light">
        <p class="first-letter:text-7xl first-letter:font-bold first-letter:text-[var(--color-orchid-dark)] first-letter:mr-3 first-letter:float-left first-line:uppercase first-line:tracking-widest">
          The pages of this script lie unwritten, waiting for VeeQ to pour her imagination into reality. Her mind races with the possibilities of what could be, painting worlds built on profound emotion and absolute contrast.
        </p>
        <p>This is merely the canvas. The real narrative begins when the words finally fall like dark petals onto the stark white expanse, bringing life, structure, and intense meaning to the void.</p>
        <p class="text-center italic mt-16 text-gray-400">
          More from <strong>{{ story.title }}</strong> coming soon.
        </p>
      </div>
    </div>
  </div>
</template>

<style scoped>
/* Optional: stylistic override for the prose if tailwind typography isn't installed */
p {
  margin-bottom: 2rem;
}
</style>
