<script setup>
import { onMounted, ref } from 'vue'
import gsap from 'gsap'
import ScrollTrigger from 'gsap/ScrollTrigger'

const emit = defineEmits(['openStory'])

gsap.registerPlugin(ScrollTrigger)

const gallerySection = ref(null)
const featureItems = ref([])

onMounted(() => {
  // Pin the gallery section and horizontally scroll the items
  let ctx = gsap.context(() => {
    const panels = gsap.utils.toArray('.feature-panel')
    
    gsap.to(panels, {
      xPercent: -100 * (panels.length - 1),
      ease: "none",
      scrollTrigger: {
        trigger: gallerySection.value,
        pin: true,
        scrub: 1,
        snap: 1 / (panels.length - 1),
        end: () => "+=" + gallerySection.value.offsetWidth
      }
    })

    // Individual text animations inside panels
    panels.forEach((panel) => {
      const texts = panel.querySelectorAll('.reveal-text')
      gsap.from(texts, {
        y: 100,
        opacity: 0,
        duration: 1,
        stagger: 0.2,
        scrollTrigger: {
          trigger: panel,
          containerAnimation: gsap.getById(gallerySection.value),
          start: "left center",
          toggleActions: "play none none reverse"
        }
      })
    })

  }, gallerySection.value)

  return () => ctx.revert() // cleanup
})

const features = [
  {
    id: 'out-of-nowhere',
    title: 'Out of Nowhere',
    description: 'An unreal romance that weaves through surreal dimensions, twisting toward a culty climax.',
    number: '01',
    gradient: 'from-white to-gray-100' 
  },
  {
    id: 'bidad',
    title: 'Bidad',
    description: 'A full sci-fi drama exploring humanity\'s survival in the harsh, unforgiving depths of the unknown.',
    number: '02',
    gradient: 'from-gray-100 to-[#f0f0f5]' 
  }
]
</script>

<template>
  <section ref="gallerySection" class="relative min-h-screen flex items-center overflow-hidden">
    <!-- Transparent backgrounds to let the canvas show through -->
    <div class="flex w-[200vw] h-screen bg-transparent">
      <div 
        v-for="(feature, index) in features" 
        :key="index"
        class="feature-panel w-screen h-screen flex flex-col justify-center items-center relative backdrop-blur-[2px]"
      >
        <!-- Very light gradient overlay over the petals -->
        <div class="absolute inset-0 bg-gradient-to-br opacity-80 z-0" :class="feature.gradient"></div> 
        
        <div class="z-10 container mx-auto px-6 lg:px-24 flex flex-col justify-center h-full text-black">
          <div class="mb-4">
            <span class="text-[var(--color-orchid-dark)] text-sm md:text-xl font-bold tracking-widest uppercase reveal-text">Story {{ feature.number }}</span>
          </div>
          <h2 class="text-5xl md:text-8xl font-black uppercase tracking-tight mb-8 reveal-text drop-shadow-sm text-black">
            {{ feature.title }}
          </h2>
          <p class="text-xl md:text-3xl text-gray-700 max-w-2xl font-light reveal-text mix-blend-multiply mb-12">
            {{ feature.description }}
          </p>
          <div class="reveal-text">
            <button @click="$emit('openStory', feature)"
                    class="px-8 py-4 border border-black/20 rounded-full text-black tracking-[0.2em] uppercase hover:bg-black hover:text-white transition-all duration-300 hover-target cursor-none">
              Read Story
            </button>
          </div>
        </div>
      </div>
    </div>
  </section>
</template>

<style scoped>
.feature-panel {
  will-change: transform;
}
</style>
