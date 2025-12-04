<template>
  <div class="end-scene w-full h-full flex items-center justify-center bg-black relative overflow-hidden">
    <!-- Background Image -->
    <div class="absolute inset-0 transition-opacity duration-1000" :class="{ 'opacity-100': isActive, 'opacity-0': !isActive }">
      <img 
        :src="src" 
        class="w-full h-full object-cover opacity-60"
        alt="End Scene"
      />
      <div class="absolute inset-0 bg-gradient-to-t from-black via-transparent to-black"></div>
    </div>

    <!-- Story Text -->
    <div class="relative z-10 max-w-2xl px-8 text-center">
      <p 
        class="text-2xl md:text-4xl font-light text-white leading-relaxed tracking-wide transition-all duration-1000"
        :class="{ 'opacity-100 translate-y-0': showText, 'opacity-0 translate-y-4': !showText }"
      >
        {{ currentText }}
      </p>
      <div v-if="isActive && !ended" class="mt-8 animate-bounce text-white/30 text-sm uppercase tracking-widest">
        Press ↓
      </div>
    </div>

    <!-- Audio Element -->
    <audio ref="audioPlayer" :src="audioSrc" loop preload="auto"></audio>
    
    <!-- Replay Button (Optional, appears at end) -->
    <button 
      v-if="ended"
      @click="replay"
      class="absolute bottom-10 px-6 py-2 border border-white/30 text-white/50 hover:text-white hover:border-white rounded-full transition-all uppercase tracking-widest text-sm z-20"
    >
      Replay Story
    </button>
  </div>
</template>

<script setup>
import { ref, watch, onUnmounted, onMounted } from 'vue'

const props = defineProps({
  isActive: Boolean,
  src: String,
  audioSrc: String,
  script: {
    type: Array,
    default: () => []
  }
})

const emit = defineEmits(['complete'])

const audioPlayer = ref(null)
const currentText = ref('')
const currentLineIndex = ref(-1)
const showText = ref(false)
const ended = ref(false)

const startSequence = () => {
  if (!audioPlayer.value) return
  
  // Reset
  audioPlayer.value.currentTime = 0
  audioPlayer.value.volume = 0
  audioPlayer.value.play().catch(e => console.log("Audio play failed (interaction needed?):", e))
  
  // Fade in audio
  const fadeAudio = setInterval(() => {
    if (audioPlayer.value.volume < 0.9) {
      audioPlayer.value.volume += 0.1
    } else {
      clearInterval(fadeAudio)
    }
  }, 200)

  // Start with first line
  nextLine()
}

const nextLine = () => {
  if (ended.value) return

  if (currentLineIndex.value < props.script.length - 1) {
    // Fade out old
    showText.value = false
    
    setTimeout(() => {
      currentLineIndex.value++
      currentText.value = props.script[currentLineIndex.value]
      showText.value = true
    }, 500)
  } else {
    // End
    ended.value = true
    emit('complete')
  }
}

const stopSequence = () => {
  if (audioPlayer.value) {
    // Fade out audio
    const fadeAudio = setInterval(() => {
      if (audioPlayer.value.volume > 0.1) {
        audioPlayer.value.volume -= 0.1
      } else {
        audioPlayer.value.pause()
        clearInterval(fadeAudio)
      }
    }, 200)
  }
}

const replay = () => {
  ended.value = false
  currentLineIndex.value = -1
  startSequence()
}

const handleKeydown = (e) => {
  if (!props.isActive) return
  if (e.key === 'ArrowDown') {
    nextLine()
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
