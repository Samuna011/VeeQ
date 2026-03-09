<script setup>
import { onMounted, ref } from 'vue'
import gsap from 'gsap'
import ScrollTrigger from 'gsap/ScrollTrigger'

gsap.registerPlugin(ScrollTrigger)

const footerText = ref(null)

onMounted(() => {
  gsap.fromTo(footerText.value, 
    { y: 100, opacity: 0, scale: 0.9 },
    { y: 0, opacity: 1, scale: 1, duration: 1.5, ease: 'power4.out',
      scrollTrigger: {
        trigger: 'footer',
        start: 'top 80%',
      }
    }
  )
})
</script>

<template>
  <footer class="relative min-h-[50vh] flex flex-col justify-center items-center py-16 bg-white/80 backdrop-blur-md border-t border-gray-200 z-10">
    <div class="absolute inset-0 bg-gradient-to-t from-[var(--color-orchid-dark)] to-transparent opacity-5 pointer-events-none"></div>

    <div class="container mx-auto px-6 text-center z-10">
      <h2 ref="footerText" class="text-4xl md:text-8xl font-black uppercase tracking-tighter text-black mb-8 hover-target cursor-none transition-transform hover:scale-105 duration-300 drop-shadow-sm">
        Say Hello to VeeQ.
      </h2>
      
      <div class="flex justify-center flex-wrap gap-8 text-sm md:text-xl text-gray-600 font-light mt-12 transition-colors">
        <a href="#" class="hover:text-[var(--color-orchid-dark)] hover-target uppercase tracking-widest transition-colors duration-300">Readings</a>
        <a href="#" class="hover:text-[var(--color-orchid-dark)] hover-target uppercase tracking-widest transition-colors duration-300">Twitter</a>
        <a href="#" class="hover:text-[var(--color-orchid-dark)] hover-target uppercase tracking-widest transition-colors duration-300">Contact</a>
      </div>
      
      <div class="mt-20 text-xs text-gray-400 uppercase tracking-widest">
        &copy; {{ new Date().getFullYear() }} VeeQ's Imagination. All rights reserved.
      </div>
    </div>
  </footer>
</template>

<style scoped>
/* Any additional footer styles */
</style>
