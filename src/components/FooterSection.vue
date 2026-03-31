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
    { y: 60, opacity: 0, scale: 0.98, filter: 'blur(10px)' },
    {
      y: 0,
      opacity: 1,
      scale: 1,
      filter: 'blur(0px)',
      duration: 1.2,
      ease: 'power3.out',
      scrollTrigger: {
        trigger: footerRef.value,
        start: 'top 85%',
      }
    }
  )
  gsap.fromTo(linksRef.value?.children || [],
    { y: 20, opacity: 0 },
    {
      y: 0,
      opacity: 1,
      duration: 0.8,
      stagger: 0.1,
      delay: 0.3,
      ease: 'power2.out',
      scrollTrigger: {
        trigger: footerRef.value,
        start: 'top 80%',
      }
    }
  )
})
</script>

<template>
  <footer ref="footerRef" id="contact" class="relative min-h-[60vh] flex flex-col justify-center items-center py-20 bg-white/60 backdrop-blur-2xl border-t border-black/5 z-10 overflow-hidden" role="contentinfo">
    <div class="absolute bottom-0 left-1/2 -translate-x-1/2 w-full max-w-3xl h-64 bg-[var(--color-orchid-light)] opacity-[0.05] blur-[120px] rounded-[100%] pointer-events-none"></div>

    <div class="container mx-auto px-6 text-center z-10">
      <h2 ref="headingRef" class="text-4xl md:text-7xl lg:text-8xl font-light uppercase tracking-[0.1em] text-black mb-12 hover:tracking-[0.15em] transition-all duration-700 cursor-default" style="font-family: var(--font-display);">
        Say Hello <br/><span class="text-black/30">To VeeQ</span>
      </h2>

      <div ref="linksRef" class="flex justify-center flex-wrap gap-8 md:gap-16 text-xs md:text-sm text-black/50 font-semibold tracking-[0.3em] uppercase">
        <a href="#gallery" class="hover:text-black transition-colors duration-500 relative group">
          Stories
          <span class="absolute -bottom-2 left-0 w-0 h-px bg-black group-hover:w-full transition-all duration-500"></span>
        </a>
        <a href="#" class="hover:text-black transition-colors duration-500 relative group" target="_blank" rel="noopener">
          Twitter
          <span class="absolute -bottom-2 left-0 w-0 h-px bg-black group-hover:w-full transition-all duration-500"></span>
        </a>
        <a href="#contact" class="hover:text-black transition-colors duration-500 relative group">
          Contact
          <span class="absolute -bottom-2 left-0 w-0 h-px bg-black group-hover:w-full transition-all duration-500"></span>
        </a>
      </div>

      <div class="mt-32 text-[10px] text-black/30 uppercase tracking-[0.4em] flex flex-col items-center gap-4">
        <div class="w-px h-12 bg-black/10"></div>
        &copy; {{ new Date().getFullYear() }} VeeQ's Imagination
      </div>
    </div>
  </footer>
</template>
