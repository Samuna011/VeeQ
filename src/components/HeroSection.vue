<script setup>
import { onMounted, ref } from 'vue'
import gsap from 'gsap'

const heroSection = ref(null)
const titleChars = ref([])
const subtitleChars = ref([])

onMounted(() => {
  const tl = gsap.timeline({ defaults: { ease: 'power4.out' } })

  tl.fromTo(titleChars.value, 
    { y: 100, opacity: 0, rotateX: -90 },
    { y: 0, opacity: 1, rotateX: 0, duration: 1.2, stagger: 0.05, delay: 0.2 }
  )
  .fromTo(subtitleChars.value,
    { y: 50, opacity: 0 },
    { y: 0, opacity: 1, duration: 1, stagger: 0.02 },
    '-=0.8'
  )
  .fromTo('.hero-bg-gradient',
    { scale: 1.5, opacity: 0 },
    { scale: 1, opacity: 0.8, duration: 2, ease: 'power2.out' },
    '-=1.5'
  )
})
</script>

<template>
  <section ref="heroSection" class="relative min-h-screen flex flex-col justify-center items-center overflow-hidden">
    <!-- Abstract Orchid Gradient Background -->
    <div class="hero-bg-gradient absolute inset-0 -z-10 bg-black">
      <div class="absolute top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2 w-[80vw] h-[80vw] max-w-[800px] max-h-[800px] rounded-full blur-[120px] mix-blend-screen opacity-60"
           style="background: radial-gradient(circle, var(--color-orchid-light) 0%, var(--color-orchid-dark) 40%, transparent 70%);">
      </div>
    </div>

    <div class="container mx-auto px-6 z-10 text-center">
      <h1 class="text-7xl md:text-9xl font-extrabold tracking-tighter uppercase mb-6 flex justify-center flex-wrap gap-x-4">
        <span class="overflow-hidden inline-block" v-for="(word, i) in ['Digital', 'Orchid']" :key="i">
          <span class="inline-block hover-target cursor-none" 
                v-for="(char, j) in word.split('')" 
                :key="j"
                ref="titleChars"
                style="display: inline-block;">
            {{ char }}
          </span>
        </span>
      </h1>
      <p class="text-xl md:text-3xl font-light text-gray-300 max-w-2xl mx-auto flex justify-center flex-wrap gap-x-1">
        <span class="overflow-hidden inline-block" v-for="(word, i) in ['Award-winning', 'aesthetic', 'experiences.']" :key="i">
          <span class="inline-block" 
                v-for="(char, j) in (word + ' ').split('')" 
                :key="j"
                ref="subtitleChars"
                style="display: inline-block; white-space: pre;">
            {{ char }}
          </span>
        </span>
      </p>
    </div>
  </section>
</template>

<style scoped>
/* Scoped styles if needed, but Tailwind v4 handles most utilities */
</style>
