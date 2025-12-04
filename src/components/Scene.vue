<template>
  <div class="scene-image w-full h-full flex items-center justify-center bg-gray-900 relative overflow-hidden">
    <img 
      :src="currentSrc" 
      class="w-full h-full object-cover animate-ken-burns"
      alt="Scene"
    />
    <div class="absolute inset-0 bg-gradient-to-t from-black/80 via-black/20 to-transparent"></div>
    <div class="absolute bottom-20 left-8 md:left-20 max-w-2xl text-white z-10">
      <p class="text-xl md:text-3xl font-light tracking-wide leading-relaxed drop-shadow-lg opacity-0 animate-fade-in-up">
        {{ caption }}
      </p>
    </div>
  </div>
</template>

<style scoped>
.animate-ken-burns {
  animation: ken-burns 20s ease-in-out infinite alternate;
}

@keyframes ken-burns {
  0% { transform: scale(1) translate(0, 0); }
  100% { transform: scale(1.1) translate(-2%, -2%); }
}

.animate-fade-in-up {
  animation: fadeInUp 1s ease-out forwards;
  animation-delay: 0.5s;
}

@keyframes fadeInUp {
  from { opacity: 0; transform: translateY(20px); }
  to { opacity: 1; transform: translateY(0); }
}
</style>

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
