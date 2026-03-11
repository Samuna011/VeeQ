<script setup>
import { onMounted, ref } from 'vue'
import gsap from 'gsap'
import ScrollTrigger from 'gsap/ScrollTrigger'

const emit = defineEmits(['openStory'])

gsap.registerPlugin(ScrollTrigger)

const gallerySection = ref(null)
const scrollProgress = ref(0)
let horizontalTween = null

function scrollToPanel(index) {
  if (!horizontalTween?.scrollTrigger) return
  const st = horizontalTween.scrollTrigger
  const panels = gsap.utils.toArray('.feature-panel')
  const progress = panels.length > 1 ? index / (panels.length - 1) : 0
  const targetScroll = st.start + progress * (st.end - st.start)
  window.scrollTo({ top: targetScroll, behavior: 'smooth' })
}

onMounted(() => {
  const ctx = gsap.context(() => {
    const panels = gsap.utils.toArray('.feature-panel')

    horizontalTween = gsap.to(panels, {
      xPercent: -100 * (panels.length - 1),
      ease: "none",
      scrollTrigger: {
        trigger: gallerySection.value,
        pin: true,
        scrub: 1.2,
        snap: 1 / (panels.length - 1),
        end: () => "+=" + (gallerySection.value.offsetWidth * (panels.length - 1)),
        onUpdate: (self) => { scrollProgress.value = self.progress }
      }
    })

    panels.forEach((panel) => {
      const numberEl = panel.querySelector('.feature-number')
      const titleEl = panel.querySelector('.feature-title')
      const descEl = panel.querySelector('.feature-desc')
      const ctaEl = panel.querySelector('.feature-cta')
      const labelEl = panel.querySelector('.feature-label')

      const stagger = 0.12
      gsap.fromTo([labelEl, numberEl], 
        { y: 40, opacity: 0, filter: 'blur(8px)' },
        { 
          y: 0, 
          opacity: 1, 
          filter: 'blur(0px)', 
          duration: 0.9, 
          stagger,
          ease: 'power3.out',
          scrollTrigger: {
            trigger: panel,
            containerAnimation: horizontalTween,
            start: "left 75%",
            toggleActions: "play none none reverse"
          }
        }
      )
      gsap.fromTo(titleEl,
        { y: 120, opacity: 0, filter: 'blur(12px)' },
        { 
          y: 0, 
          opacity: 1, 
          filter: 'blur(0px)', 
          duration: 1,
          ease: 'power3.out',
          scrollTrigger: {
            trigger: panel,
            containerAnimation: horizontalTween,
            start: "left 70%",
            toggleActions: "play none none reverse"
          }
        }
      )
      gsap.fromTo(descEl,
        { y: 60, opacity: 0, filter: 'blur(6px)' },
        { 
          y: 0, 
          opacity: 1, 
          filter: 'blur(0px)', 
          duration: 0.85,
          delay: 0.15,
          ease: 'power3.out',
          scrollTrigger: {
            trigger: panel,
            containerAnimation: horizontalTween,
            start: "left 65%",
            toggleActions: "play none none reverse"
          }
        }
      )
      gsap.fromTo(ctaEl,
        { y: 40, opacity: 0 },
        { 
          y: 0, 
          opacity: 1, 
          duration: 0.8,
          delay: 0.25,
          ease: 'power3.out',
          scrollTrigger: {
            trigger: panel,
            containerAnimation: horizontalTween,
            start: "left 60%",
            toggleActions: "play none none reverse"
          }
        }
      )

      // Subtle 3D tilt on panel based on scroll position
      ScrollTrigger.create({
        trigger: panel,
        containerAnimation: horizontalTween,
        start: "left right",
        end: "right left",
        onUpdate: (self) => {
          const progress = self.progress
          const rotateY = (progress - 0.5) * 8
          gsap.set(panel, { rotateY, transformPerspective: 1200 })
        }
      })
    })
  }, gallerySection.value)

  return () => ctx.revert()
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
    gradient: 'from-gray-100 to-[#f5f0f8]'
  },
  {
    id: 'coming-soon',
    title: 'More Stories',
    description: 'New worlds and characters are taking shape. Stay tuned for the next chapter from VeeQ\'s imagination.',
    number: '03',
    gradient: 'from-[#f5f0f8] to-[#ede8f0]'
  }
]
</script>

<template>
  <section ref="gallerySection" id="gallery" class="relative min-h-screen flex items-center overflow-hidden" aria-label="Story gallery">
    <div class="flex h-screen bg-transparent" :style="{ width: `${features.length * 100}vw`, perspective: '1200px' }">
      <div
        v-for="(feature, index) in features"
        :key="index"
        class="feature-panel w-screen h-screen flex flex-col justify-center items-center relative backdrop-blur-[2px] origin-center"
        style="backface-visibility: hidden;"
      >
        <div class="absolute inset-0 bg-gradient-to-br opacity-90 z-0 transition-opacity duration-500" :class="feature.gradient"></div>

        <div class="z-10 container mx-auto px-6 lg:px-24 flex flex-col justify-center h-full text-black">
          <div class="mb-6 flex items-baseline gap-6">
            <span class="feature-label text-[var(--color-orchid-dark)] text-sm md:text-base font-semibold tracking-[0.35em] uppercase">Story</span>
            <span class="feature-number text-4xl md:text-6xl font-black text-black/10 tracking-tighter" style="font-family: var(--font-display);">{{ feature.number }}</span>
          </div>
          <h2 class="feature-title text-5xl md:text-8xl lg:text-9xl font-black uppercase tracking-tighter mb-10 text-black drop-shadow-sm" style="font-family: var(--font-display); letter-spacing: -0.03em;">
            {{ feature.title }}
          </h2>
          <p class="feature-desc text-xl md:text-3xl text-gray-600 max-w-2xl font-light leading-relaxed mb-14" style="font-family: var(--font-serif);">
            {{ feature.description }}
          </p>
          <div class="feature-cta">
            <button
              @click="$emit('openStory', feature)"
              class="px-10 py-4 border-2 border-black/25 rounded-full text-black tracking-[0.25em] uppercase hover:bg-black hover:text-white transition-all duration-400 hover:scale-[1.02] font-medium"
            >
              Read Story
            </button>
          </div>
        </div>
      </div>
    </div>

    <!-- Scroll progress indicator -->
    <div class="absolute bottom-8 left-1/2 -translate-x-1/2 flex items-center gap-4 z-20 pointer-events-none">
      <div class="flex gap-2 pointer-events-auto">
        <button
          v-for="(_, i) in features"
          :key="i"
          type="button"
          class="w-2 h-2 rounded-full transition-all duration-300 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-black/30"
          :class="Math.abs(scrollProgress - (features.length > 1 ? i / (features.length - 1) : 0)) < 0.15 ? 'bg-black scale-125' : 'bg-black/25 hover:bg-black/50'"
          :aria-label="`Go to story ${i + 1}`"
          @click="scrollToPanel(i)"
        />
      </div>
      <div class="w-24 h-0.5 bg-black/10 rounded-full overflow-hidden" role="progressbar" :aria-valuenow="Math.round(scrollProgress * 100)" aria-valuemin="0" aria-valuemax="100">
        <div class="h-full bg-black/40 transition-all duration-300 ease-out" :style="{ width: `${scrollProgress * 100}%` }" />
      </div>
    </div>
  </section>
</template>

<style scoped>
.feature-panel {
  will-change: transform;
}
</style>
