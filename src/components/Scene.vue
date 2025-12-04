<template>
  <div class="scene-image w-full h-full flex items-center justify-center bg-gray-900 relative overflow-hidden">
    <img 
      :src="currentSrc" 
      class="w-full h-full object-cover transition-transform duration-[2000ms] ease-in-out hover:scale-105"
      alt="Scene"
    />
    <div class="absolute inset-0 bg-black/20"></div>
    <div class="absolute bottom-12 left-8 max-w-md text-white">
      <p class="text-lg font-light tracking-wide drop-shadow-md">{{ caption }}</p>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch, onUnmounted } from 'vue'

const props = defineProps({
  src: [String, Array],
  caption: String,
  animationSpeed: {
    type: Number,
    default: 1000 // ms per frame
  },
  isActive: Boolean
})

const emit = defineEmits(['complete'])

const currentFrameIndex = ref(0)
let animationInterval = null
let completionTimeout = null

const currentSrc = computed(() => {
  if (Array.isArray(props.src)) {
    return props.src[currentFrameIndex.value]
  }
  return props.src
})

const startAnimation = () => {
  if (Array.isArray(props.src) && props.src.length > 1) {
    stopAnimation()
    
    // Play animation
    animationInterval = setInterval(() => {
      currentFrameIndex.value = (currentFrameIndex.value + 1) % props.src.length
    }, props.animationSpeed)

    // Schedule completion
    const duration = props.src.length * props.animationSpeed
    completionTimeout = setTimeout(() => {
      emit('complete')
    }, duration)
  } else {
    // Static image
    completionTimeout = setTimeout(() => {
      emit('complete')
    }, 1000) // Brief pause for static scenes
  }
}

const stopAnimation = () => {
  if (animationInterval) clearInterval(animationInterval)
  if (completionTimeout) clearTimeout(completionTimeout)
}

watch(() => props.isActive, (active) => {
  if (active) startAnimation()
  else stopAnimation()
}, { immediate: true })

onUnmounted(stopAnimation)
</script>
