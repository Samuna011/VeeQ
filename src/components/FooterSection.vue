<script setup>
import { onMounted, ref } from 'vue'
import gsap from 'gsap'
import ScrollTrigger from 'gsap/ScrollTrigger'

gsap.registerPlugin(ScrollTrigger)

const footerRef = ref(null)
const headingRef = ref(null)
const linksRef = ref(null)

onMounted(() => {
  gsap.fromTo(headingRef.value,
    { y: 80, opacity: 0, scale: 0.96, filter: 'blur(10px)' },
    {
      y: 0,
      opacity: 1,
      scale: 1,
      filter: 'blur(0px)',
      duration: 1.2,
      ease: 'power3.out',
      scrollTrigger: {
        trigger: footerRef.value,
        start: 'top 82%',
      }
    }
  )
  gsap.fromTo(linksRef.value?.children || [],
    { y: 30, opacity: 0 },
    {
      y: 0,
      opacity: 1,
      duration: 0.8,
      stagger: 0.1,
      delay: 0.2,
      ease: 'power2.out',
      scrollTrigger: {
        trigger: footerRef.value,
        start: 'top 78%',
      }
    }
  )
})
</script>

<template>
  <footer ref="footerRef" id="contact" class="relative min-h-[55vh] flex flex-col justify-center items-center py-20 bg-white/90 backdrop-blur-md border-t border-black/5 z-10 overflow-hidden" role="contentinfo">
    <div class="absolute inset-0 bg-gradient-to-t from-[var(--color-orchid-dark)] to-transparent opacity-[0.06] pointer-events-none"></div>
    <div class="absolute bottom-0 left-1/2 -translate-x-1/2 w-[80%] h-32 bg-[var(--color-orchid-light)]/10 blur-[80px] rounded-full pointer-events-none"></div>

    <div class="container mx-auto px-6 text-center z-10">
      <h2 ref="headingRef" class="text-4xl md:text-7xl lg:text-8xl font-black uppercase tracking-tighter text-black mb-10 transition-transform hover:scale-[1.02] duration-500 drop-shadow-sm cursor-default" style="font-family: var(--font-display); letter-spacing: -0.02em;">
        Say Hello to VeeQ.
      </h2>

      <div ref="linksRef" class="flex justify-center flex-wrap gap-10 text-sm md:text-lg text-gray-500 font-medium tracking-widest uppercase">
        <a href="#gallery" class="hover:text-[var(--color-orchid-dark)] transition-colors duration-300">Stories</a>
        <a href="#" class="hover:text-[var(--color-orchid-dark)] transition-colors duration-300" target="_blank" rel="noopener">Twitter</a>
        <a href="#contact" class="hover:text-[var(--color-orchid-dark)] transition-colors duration-300">Contact</a>
      </div>

      <div class="mt-24 text-xs text-gray-400 uppercase tracking-[0.2em]">
        &copy; {{ new Date().getFullYear() }} VeeQ's Imagination. All rights reserved.
      </div>
    </div>
  </footer>
</template>
