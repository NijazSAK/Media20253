<template>
  <div class="intro-scene w-full h-full flex items-center justify-center bg-black relative overflow-hidden">
    
    <!-- Start Prompt -->
    <div v-if="!hasStarted" class="absolute inset-0 flex flex-col items-center justify-center z-50">
      <div class="animate-bounce text-white/50 font-mono tracking-widest uppercase text-sm">
        Press &darr; to Start
      </div>
    </div>

    <transition name="fade" mode="out-in">
      <div v-if="hasStarted" :key="currentIndex" class="absolute inset-0 w-full h-full">
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
import { ref, computed, watch, onMounted, onUnmounted } from 'vue'

const props = defineProps({
  slides: {
    type: Array,
    required: true,
  },
  isActive: Boolean
})

const emit = defineEmits(['complete'])

const currentIndex = ref(0)
const hasStarted = ref(false)
const currentSlide = computed(() => props.slides[currentIndex.value])
let slideTimeout = null

const startSequence = () => {
  hasStarted.value = true
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
  hasStarted.value = false
}

const handleKeydown = (e) => {
  if (props.isActive && !hasStarted.value && e.key === 'ArrowDown') {
    startSequence()
  }
}

onMounted(() => {
  window.addEventListener('keydown', handleKeydown)
})

onUnmounted(() => {
  stopSequence()
  window.removeEventListener('keydown', handleKeydown)
})

// Watch for deactivation to reset
watch(() => props.isActive, (active) => {
  if (!active) stopSequence()
})
</script>


<style scoped>
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>
