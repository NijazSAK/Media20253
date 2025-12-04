<template>
  <div class="end-scene w-full h-full flex items-center justify-center bg-black relative overflow-hidden">
    <!-- Background Image Slideshow -->
    <div class="absolute inset-0">
      <transition-group name="fade">
        <div 
          v-for="(image, index) in src" 
          v-show="index === currentIndex"
          :key="image"
          class="absolute inset-0 w-full h-full"
        >
          <img 
            :src="image" 
            class="w-full h-full object-cover opacity-60"
            alt="End Scene"
          />
        </div>
      </transition-group>
      <div class="absolute inset-0 bg-gradient-to-t from-black via-transparent to-black z-10"></div>
    </div>

    <!-- Story Text -->
    <div class="relative z-20 max-w-2xl px-8 text-center">
      <transition name="fade" mode="out-in">
        <p 
          :key="currentIndex"
          class="text-2xl md:text-4xl font-light text-white leading-relaxed tracking-wide"
        >
          {{ script[currentIndex] }}
        </p>
      </transition>
      
      <div v-if="isActive && !hasStarted" class="mt-8 animate-bounce text-white/30 text-sm uppercase tracking-widest">
        Press ↓ to Start Flashback
      </div>
    </div>

    <!-- Audio Element -->
    <audio ref="audioPlayer" :src="audioSrc" loop preload="auto"></audio>
    
    <!-- Replay Button -->
    <button 
      v-if="ended"
      @click="replay"
      class="absolute bottom-10 px-6 py-2 border border-white/30 text-white/50 hover:text-white hover:border-white rounded-full transition-all uppercase tracking-widest text-sm z-30"
    >
      Replay Story
    </button>
  </div>
</template>

<script setup>
import { ref, watch, onUnmounted, onMounted } from 'vue'

const props = defineProps({
  isActive: Boolean,
  src: {
    type: Array,
    default: () => []
  },
  audioSrc: String,
  script: {
    type: Array,
    default: () => []
  }
})

const emit = defineEmits(['complete'])

const audioPlayer = ref(null)
const currentIndex = ref(0)
const hasStarted = ref(false)
const ended = ref(false)
let slideInterval = null

const startSequence = () => {
  if (!audioPlayer.value) return
  
  // Reset
  audioPlayer.value.currentTime = 0
  audioPlayer.value.volume = 0
  audioPlayer.value.play().catch(e => console.log("Audio play failed:", e))
  
  // Fade in audio
  const fadeAudio = setInterval(() => {
    if (audioPlayer.value.volume < 0.9) {
      audioPlayer.value.volume += 0.1
    } else {
      clearInterval(fadeAudio)
    }
  }, 200)
}

const startSlideshow = () => {
  if (hasStarted.value) return
  hasStarted.value = true
  
  slideInterval = setInterval(() => {
    if (currentIndex.value < props.src.length - 1) {
      currentIndex.value++
    } else {
      // End
      clearInterval(slideInterval)
      ended.value = true
      emit('complete')
    }
  }, 5000) // 5 seconds per slide
}

const stopSequence = () => {
  if (audioPlayer.value) {
    const fadeAudio = setInterval(() => {
      if (audioPlayer.value.volume > 0.1) {
        audioPlayer.value.volume -= 0.1
      } else {
        audioPlayer.value.pause()
        clearInterval(fadeAudio)
      }
    }, 200)
  }
  if (slideInterval) clearInterval(slideInterval)
}

const replay = () => {
  ended.value = false
  hasStarted.value = false
  currentIndex.value = 0
  startSequence()
}

const handleKeydown = (e) => {
  if (!props.isActive) return
  if (e.key === 'ArrowDown') {
    startSlideshow()
  }
}

watch(() => props.isActive, (active) => {
  if (active) {
    startSequence()
  } else {
    stopSequence()
  }
})

onMounted(() => {
  window.addEventListener('keydown', handleKeydown)
})

onUnmounted(() => {
  stopSequence()
  window.removeEventListener('keydown', handleKeydown)
})
</script>

<style scoped>
.fade-enter-active,
.fade-leave-active {
  transition: opacity 1.5s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>
