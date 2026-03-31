<script setup>
import { onMounted, ref } from 'vue'
import gsap from 'gsap'
import ScrollTrigger from 'gsap/ScrollTrigger'

const emit = defineEmits(['openStory'])

gsap.registerPlugin(ScrollTrigger)

const gallerySection = ref(null)

onMounted(() => {
  const ctx = gsap.context(() => {
    const items = gsap.utils.toArray('.story-item')
    items.forEach((el, index) => {
      gsap.fromTo(el,
        { opacity: 0, y: 30, scale: 0.98 },
        {
          opacity: 1,
          y: 0,
          scale: 1,
          duration: 0.7,
          ease: 'power3.out',
          scrollTrigger: {
            trigger: el,
            start: 'top 85%',
            toggleActions: 'play none none reverse'
          }
        }
      )
    })
  }, gallerySection.value)
  return () => ctx.revert()
})

const features = [
  {
    id: 'out-of-nowhere',
    title: 'Out of Nowhere',
    description: 'An unreal romance that weaves through surreal dimensions, twisting toward a culty climax.',
    number: '01'
  },
  {
    id: 'bidad',
    title: 'Bidad',
    description: 'A full sci-fi drama exploring humanity\'s survival in the harsh, unforgiving depths of the unknown.',
    number: '02'
  },
  {
    id: 'coming-soon',
    title: 'More Stories',
    description: 'New worlds and characters are taking shape. Stay tuned for the next chapter from VeeQ\'s imagination.',
    number: '03'
  }
]
</script>

<template>
  <section ref="gallerySection" id="gallery" class="relative min-h-screen py-20 md:py-32" aria-label="Story gallery">
    <div class="container mx-auto px-6 sm:px-8 max-w-4xl relative z-10">
      <header class="mb-20 md:mb-24 text-center">
        <h2 class="text-3xl md:text-5xl font-light text-black tracking-widest uppercase mb-4" style="font-family: var(--font-display);">
          Stories
        </h2>
        <div class="w-12 h-px bg-black/20 mx-auto mb-6"></div>
        <p class="text-sm md:text-base text-black/50 tracking-widest uppercase" style="font-family: var(--font-display);">
          Select an experience
        </p>
      </header>

      <ul class="flex flex-col gap-8 md:gap-12">
        <li
          v-for="(feature, index) in features"
          :key="feature.id"
          class="story-item"
        >
          <button
            type="button"
            class="story-card w-full text-left rounded-[2rem] border border-black/5 bg-white/70 backdrop-blur-2xl px-6 py-8 md:px-12 md:py-10 hover:border-[var(--color-orchid-dark)]/20 hover:bg-white/95 hover:shadow-xl hover:shadow-[var(--color-orchid-light)]/5 transition-all duration-500 ease-out group relative overflow-hidden"
            @click="$emit('openStory', feature)"
          >
            <!-- Hover light glow effect -->
            <div class="absolute inset-0 opacity-0 group-hover:opacity-100 transition-opacity duration-700 pointer-events-none"
                 style="background: radial-gradient(800px circle at 50% 50%, rgba(153, 50, 204, 0.03), transparent 40%);"></div>
                 
            <div class="flex flex-col md:flex-row md:items-center justify-between gap-6 md:gap-10 relative z-10">
              <div class="min-w-0 flex-1">
                <div class="flex items-center gap-4 mb-4">
                  <span class="text-xs font-semibold tracking-[0.3em] text-[var(--color-orchid-light)] group-hover:text-[var(--color-orchid-dark)] transition-colors duration-500">
                    CHAPTER {{ feature.number }}
                  </span>
                  <div class="h-px bg-black/5 flex-1 group-hover:bg-[var(--color-orchid-light)]/20 transition-colors duration-500"></div>
                </div>
                
                <h3 class="text-2xl md:text-4xl font-normal text-black group-hover:text-[var(--color-orchid-dark)] transition-colors duration-500 tracking-wide mb-4" style="font-family: var(--font-display);">
                  {{ feature.title }}
                </h3>
                
                <p class="text-sm md:text-lg text-black/60 leading-relaxed font-light max-w-2xl" style="font-family: var(--font-serif);">
                  {{ feature.description }}
                </p>
                
                <div class="mt-8 flex items-center gap-3">
                  <div class="w-8 h-8 rounded-full border border-black/10 flex items-center justify-center group-hover:bg-black group-hover:border-black transition-all duration-500">
                    <svg class="w-3 h-3 text-black group-hover:text-white transition-colors duration-500" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5l7 7-7 7" />
                    </svg>
                  </div>
                  <span class="text-xs font-medium tracking-[0.2em] uppercase text-black/40 group-hover:text-black transition-colors duration-500">
                    Enter Reader
                  </span>
                </div>
              </div>
            </div>
          </button>
        </li>
      </ul>
    </div>
  </section>
</template>

<style scoped>
.story-card:focus-visible {
  outline: 2px solid var(--color-orchid-dark);
  outline-offset: 4px;
}
</style>
