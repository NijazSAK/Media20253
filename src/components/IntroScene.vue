<template>
  <div class="intro-scene w-full h-full flex items-center justify-center bg-black relative overflow-hidden">
    <transition name="fade" mode="in-out">
      <div :key="currentIndex" class="absolute inset-0 w-full h-full">
        <img 
          :src="currentSlide.src" 
          class="w-full h-full object-cover opacity-80"
          alt="Intro Scene"
        />
        <div class="absolute inset-0 bg-black/40"></div>
        <div class="absolute inset-0 flex items-center justify-center">
          <h1 class="text-4xl md:text-6xl font-bold text-white tracking-widest uppercase drop-shadow-2xl text-center px-4 animate-pulse">
            {{ currentSlide.caption }}
          </h1>
        </div>
      </div>
    </transition>
  </div>
</template>

<script setup>
import { ref, computed, watch, onUnmounted } from 'vue'

const props = defineProps({
  slides: {
    type: Array,
    required: true,
    // Expected format: [{ src: string, caption: string, duration: number }]
  },
  isActive: Boolean
})

const emit = defineEmits(['complete'])

const currentIndex = ref(0)
const currentSlide = computed(() => props.slides[currentIndex.value])
let slideTimeout = null

const startSequence = () => {
  clearTimeout(slideTimeout)
  currentIndex.value = 0
  scheduleNextSlide()
}

const scheduleNextSlide = () => {
  const slide = props.slides[currentIndex.value]
  const duration = slide.duration || 5000
  
  slideTimeout = setTimeout(() => {
    if (currentIndex.value < props.slides.length - 1) {
      currentIndex.value++
      scheduleNextSlide()
    } else {
      emit('complete')
    }
  }, duration)
}

const stopSequence = () => {
  clearTimeout(slideTimeout)
}

watch(() => props.isActive, (active) => {
  if (active) startSequence()
  else stopSequence()
}, { immediate: true })

onUnmounted(stopSequence)
</script>

<style scoped>
.fade-enter-active,
.fade-leave-active {
  transition: opacity 2s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>
