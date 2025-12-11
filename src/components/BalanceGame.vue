<template>
  <div class="balance-game relative w-full h-full flex flex-col items-center justify-center bg-black overflow-hidden perspective-container">
    
    <!-- Dynamic Backgrounds -->
    <div class="absolute inset-0 z-0">
      <transition-group name="fade">
        <img 
          v-if="currentBackgroundIndex === 0" 
          key="bg1"
          :src="backgrounds[0]" 
          class="absolute inset-0 w-full h-full object-cover transition-opacity duration-500"
        />
        <img 
          v-if="currentBackgroundIndex === 1" 
          key="bg2"
          :src="backgrounds[1]" 
          class="absolute inset-0 w-full h-full object-cover transition-opacity duration-500"
        />
        <img 
          v-if="currentBackgroundIndex === 2" 
          key="bg3"
          :src="backgrounds[2]" 
          class="absolute inset-0 w-full h-full object-cover transition-opacity duration-500"
        />
      </transition-group>
      <div class="absolute inset-0 bg-black/30"></div>
    </div>

    <!-- Game Layer -->
    <div class="relative w-full h-full max-w-4xl mx-auto flex items-center justify-center perspective-view transition-opacity duration-500" :class="{ 'opacity-100': isActive, 'opacity-0': !isActive }">
      
      <!-- Balance Bar UI (Moved to Bottom) -->
      <div v-if="gameState === 'playing'" class="absolute bottom-40 w-96 h-6 bg-slate-900/90 rounded-full border-2 border-slate-600 overflow-hidden backdrop-blur-sm shadow-xl z-10">
        <!-- Left Bar (Fills from Left) -->
        <div 
          v-if="tilt < 0"
          class="absolute left-0 top-0 bottom-0 bg-gradient-to-r from-red-600 to-red-400 transition-all duration-75 ease-linear"
          :style="{ width: `${Math.min(Math.abs(tilt), 100)}%` }"
        ></div>

        <!-- Right Bar (Fills from Right) -->
        <div 
          v-if="tilt > 0"
          class="absolute right-0 top-0 bottom-0 bg-gradient-to-l from-red-600 to-red-400 transition-all duration-75 ease-linear"
          :style="{ width: `${Math.min(Math.abs(tilt), 100)}%` }"
        ></div>

        <!-- Center Marker (Safe Zone) -->
        <div class="absolute left-1/2 top-0 bottom-0 w-0.5 bg-white/20 transform -translate-x-1/2"></div>
      </div>

      <!-- Countdown Overlay -->
      <div v-if="gameState === 'countdown'" class="absolute inset-0 flex items-center justify-center z-50">
        <div class="text-9xl font-bold text-white animate-pulse drop-shadow-[0_0_20px_rgba(255,255,255,0.8)]">
          {{ countdown }}
        </div>
        <div class="absolute bottom-1/4 text-white/70 text-xl tracking-widest uppercase">Keep it steady...</div>
      </div>



    </div>

    <!-- UI / Controls -->
    <div class="absolute bottom-8 w-full flex flex-col items-center gap-4 z-50 transition-opacity duration-500" :class="{ 'opacity-100': gameState === 'playing', 'opacity-0': gameState !== 'playing' }">
      <div class="w-64 h-2 bg-slate-700 rounded-full overflow-hidden border border-slate-600">
        <div class="h-full bg-green-500 transition-all duration-300" :style="{ width: `${progress * 100}%` }"></div>
      </div>
      
      <div class="flex gap-24 text-white/50 font-mono text-lg font-bold tracking-wider">
        <span :class="{ 'text-red-400 scale-110': tilt < 0, 'opacity-30': tilt > 0 }">&larr; PUSH</span>
        <span :class="{ 'text-red-400 scale-110': tilt > 0, 'opacity-30': tilt < 0 }">PUSH &rarr;</span>
      </div>
    </div>

    <!-- Touch Controls Overlay -->
    <div v-if="gameState === 'playing'" class="absolute inset-0 flex z-40">
      <div 
        class="w-1/2 h-full active:bg-white/5 transition-colors"
        @touchstart.prevent="startPush(-1)"
        @touchend.prevent="stopPush"
        @mousedown="startPush(-1)"
        @mouseup="stopPush"
        @mouseleave="stopPush"
      ></div>
      <div 
        class="w-1/2 h-full active:bg-white/5 transition-colors"
        @touchstart.prevent="startPush(1)"
        @touchend.prevent="stopPush"
        @mousedown="startPush(1)"
        @mouseup="stopPush"
        @mouseleave="stopPush"
      ></div>
    </div>

    <!-- Fall Message (Full Screen Overlay) -->
    <div v-if="isFallen" class="absolute inset-0 flex items-center justify-center bg-black z-[100] animate-fade-in">
      <h2 class="text-5xl md:text-7xl font-bold text-red-600 tracking-widest uppercase drop-shadow-lg text-center px-4 animate-shake">
        OWWW!<br>NOT AGAIN!!!
      </h2>
    </div>

  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted, watch, computed } from 'vue'

const props = defineProps({
  isActive: Boolean,
  backgrounds: {
    type: Array,
    default: () => []
  }
})

const emit = defineEmits(['complete'])

const tilt = ref(0) // -100 to 100
const pushForce = ref(0)
const progress = ref(0) // 0 to 1
const isFallen = ref(false)
const gameState = ref('idle')
const countdown = ref(3)

// Game Config - Easier Settings
const walkSpeed = 0.0015 // Faster progress so it ends sooner
const maxTilt = 100
const driftSpeed = 0.4 // Slow natural drift
const correctionSpeed = 1.5 // Strong player correction

let gameLoop = null
let lastTime = 0
let countdownTimer = null
let driftDirection = 1 // 1 or -1
let driftTimer = 0

const currentBackgroundIndex = computed(() => {
  if (progress.value < 0.33) return 0
  if (progress.value < 0.66) return 1
  return 2
})

const startGameSequence = () => {
  tilt.value = 0
  pushForce.value = 0
  progress.value = 0
  isFallen.value = false
  gameState.value = 'countdown'
  countdown.value = 3
  driftDirection = Math.random() > 0.5 ? 1 : -1
  
  if (countdownTimer) clearInterval(countdownTimer)
  countdownTimer = setInterval(() => {
    countdown.value--
    if (countdown.value <= 0) {
      clearInterval(countdownTimer)
      gameState.value = 'playing'
      lastTime = 0
      gameLoop = requestAnimationFrame(update)
    }
  }, 1000)
}

watch(() => props.isActive, (newVal) => {
  if (newVal) {
    startGameSequence()
  } else {
    gameState.value = 'idle'
    if (gameLoop) cancelAnimationFrame(gameLoop)
    if (countdownTimer) clearInterval(countdownTimer)
  }
}, { immediate: true })

const startPush = (direction) => {
  if (gameState.value !== 'playing') return
  pushForce.value = direction
}

const stopPush = () => {
  pushForce.value = 0
}

const update = (timestamp) => {
  if (gameState.value !== 'playing') return
  
  if (!lastTime) lastTime = timestamp
  const dt = (timestamp - lastTime) / 16
  lastTime = timestamp

  // 1. Progress
  progress.value += walkSpeed * dt

  // 2. Drift Logic
  driftTimer += dt
  if (driftTimer > 100) { // Every ~1.6 seconds
    if (Math.random() > 0.5) driftDirection *= -1
    driftTimer = 0
  }

  let currentDrift = 0
  if (tilt.value === 0) {
    currentDrift = driftDirection * driftSpeed
  } else {
    currentDrift = (tilt.value > 0 ? 1 : -1) * driftSpeed
  }

  // 3. Player Correction
  let correction = 0
  if (pushForce.value === -1 && tilt.value < 0) {
    correction = correctionSpeed // Counteract negative tilt
  } else if (pushForce.value === 1 && tilt.value > 0) {
    correction = -correctionSpeed // Counteract positive tilt
  }

  // Apply changes
  tilt.value += (currentDrift + correction) * dt

  // 4. Win/Lose
  if (Math.abs(tilt.value) >= maxTilt) {
    handleFail()
    return
  }

  if (progress.value >= 1) {
    handleSuccess()
    return
  }

  gameLoop = requestAnimationFrame(update)
}

const handleFail = () => {
  isFallen.value = true
  gameState.value = 'ended'
  cancelAnimationFrame(gameLoop)
  setTimeout(() => {
    emit('complete', { success: false })
  }, 2000) // Show fail screen for 2 seconds
}

const handleSuccess = () => {
  gameState.value = 'ended'
  cancelAnimationFrame(gameLoop)
  emit('complete', { success: true })
}

const handleKeydown = (e) => {
  if (gameState.value !== 'playing') return
  if (e.key === 'ArrowLeft' || e.key.toLowerCase() === 'a') startPush(-1)
  if (e.key === 'ArrowRight' || e.key.toLowerCase() === 'd') startPush(1)
}

const handleKeyup = (e) => {
  if (['ArrowLeft', 'ArrowRight', 'a', 'd'].includes(e.key.toLowerCase()) || ['ArrowLeft', 'ArrowRight'].includes(e.key)) {
    stopPush()
  }
}

onMounted(() => {
  window.addEventListener('keydown', handleKeydown)
  window.addEventListener('keyup', handleKeyup)
})

onUnmounted(() => {
  if (gameLoop) cancelAnimationFrame(gameLoop)
  if (countdownTimer) clearInterval(countdownTimer)
  window.removeEventListener('keydown', handleKeydown)
  window.removeEventListener('keyup', handleKeyup)
})
</script>

<style scoped>
.perspective-container {
  perspective: 1000px;
}
.perspective-view {
  transform-style: preserve-3d;
}

.animate-fade-in {
  animation: fadeIn 0.2s ease-out;
}
@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

.animate-shake {
  animation: shake 0.5s cubic-bezier(.36,.07,.19,.97) both;
}

@keyframes shake {
  10%, 90% { transform: translate3d(-1px, 0, 0); }
  20%, 80% { transform: translate3d(2px, 0, 0); }
  30%, 50%, 70% { transform: translate3d(-4px, 0, 0); }
  40%, 60% { transform: translate3d(4px, 0, 0); }
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.1s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>
