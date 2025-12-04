<template>
  <div class="balance-game relative w-full h-full flex flex-col items-center justify-center bg-black overflow-hidden perspective-container">
    
    <!-- Alleyway Background -->
    <div class="absolute inset-0 overflow-hidden transition-opacity duration-1000" :class="{ 'opacity-100': isActive, 'opacity-0': !isActive }">
      <!-- Sky/Night -->
      <div class="absolute top-0 w-full h-1/2 bg-gradient-to-b from-slate-900 to-slate-800"></div>
      <!-- Ground -->
      <div class="absolute bottom-0 w-full h-1/2 bg-gradient-to-b from-slate-800 to-slate-900"></div>
      
      <!-- Walls (Perspective) -->
      <div class="absolute top-0 left-0 w-full h-full perspective-walls">
        <div class="absolute top-0 left-0 h-full w-[30%] bg-gradient-to-r from-black to-slate-900 transform origin-left skew-y-12 opacity-80"></div>
        <div class="absolute top-0 right-0 h-full w-[30%] bg-gradient-to-l from-black to-slate-900 transform origin-right -skew-y-12 opacity-80"></div>
      </div>

      <!-- Street Lights -->
      <div class="absolute top-[20%] left-[20%] w-2 h-32 bg-yellow-900/50 blur-sm"></div>
      <div class="absolute top-[20%] right-[20%] w-2 h-32 bg-yellow-900/50 blur-sm"></div>
    </div>

    <!-- Game Layer -->
    <div class="relative w-full h-full max-w-4xl mx-auto flex items-center justify-center perspective-view transition-opacity duration-1000" :class="{ 'opacity-100': isActive, 'opacity-0': !isActive }">
      
      <!-- Character Container (Scales as they walk away) -->
      <div 
        class="character-rig absolute bottom-10 transition-transform duration-100 ease-linear"
        :style="{ 
          transform: `scale(${1 - (progress * 0.7)}) translateY(${-progress * 20}px) rotate(${tilt}deg)`,
          opacity: isFallen ? 0 : 1
        }"
      >
        <!-- Stick Figure -->
        <div class="relative w-20 h-48 flex flex-col items-center">
          <!-- Head -->
          <div class="w-12 h-12 bg-white rounded-full shadow-[0_0_15px_rgba(255,255,255,0.5)] relative z-10"></div>
          
          <!-- Body -->
          <div class="w-2 h-24 bg-white rounded-full -mt-1 relative z-0"></div>
          
          <!-- Arms (Flailing) -->
          <div 
            class="absolute top-14 w-32 h-2 bg-white rounded-full transition-transform duration-300"
            :style="{ transform: `rotate(${-tilt * 2}deg)` }"
          ></div>

          <!-- Legs (Walking Animation) -->
          <div class="absolute bottom-0 w-full h-24">
            <div class="absolute left-1/2 top-0 w-2 h-24 bg-white origin-top animate-walk-left" :class="{ 'paused': gameState !== 'playing' }"></div>
            <div class="absolute left-1/2 top-0 w-2 h-24 bg-white origin-top animate-walk-right" :class="{ 'paused': gameState !== 'playing' }"></div>
          </div>
        </div>
      </div>

      <!-- Countdown Overlay -->
      <div v-if="gameState === 'countdown'" class="absolute inset-0 flex items-center justify-center z-50">
        <div class="text-9xl font-bold text-white animate-pulse drop-shadow-[0_0_20px_rgba(255,255,255,0.8)]">
          {{ countdown }}
        </div>
        <div class="absolute bottom-1/4 text-white/70 text-xl tracking-widest uppercase">Get Ready...</div>
      </div>

      <!-- Fall Message -->
      <div v-if="isFallen" class="absolute inset-0 flex items-center justify-center bg-black/80 z-50 animate-fade-in">
        <h2 class="text-4xl font-bold text-red-500 tracking-widest uppercase">Oof...</h2>
      </div>

    </div>

    <!-- UI / Controls -->
    <div class="absolute bottom-8 w-full flex flex-col items-center gap-4 z-50 transition-opacity duration-500" :class="{ 'opacity-100': gameState === 'playing', 'opacity-0': gameState !== 'playing' }">
      <div class="w-64 h-2 bg-slate-700 rounded-full overflow-hidden border border-slate-600">
        <div class="h-full bg-green-500 transition-all duration-300" :style="{ width: `${progress * 100}%` }"></div>
      </div>
      
      <div class="flex gap-12 text-white/50 font-mono text-sm">
        <span>&larr; LEFT</span>
        <span>RIGHT &rarr;</span>
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

  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted, watch } from 'vue'

const props = defineProps({
  isActive: Boolean
})

const emit = defineEmits(['complete'])

const tilt = ref(0)
const velocity = ref(0)
const pushForce = ref(0)
const progress = ref(0) // 0 to 1
const isFallen = ref(false)
const gameState = ref('idle') // idle, countdown, playing, ended
const countdown = ref(5)

// Game Config
const walkSpeed = 0.0015 // Progress per frame
const maxTilt = 60
const drunkNoise = 0.3 // Random force magnitude

let gameLoop = null
let lastTime = 0
let countdownTimer = null

const startGameSequence = () => {
  // Reset
  tilt.value = 0
  velocity.value = 0
  pushForce.value = 0
  progress.value = 0
  isFallen.value = false
  gameState.value = 'countdown'
  countdown.value = 5
  
  // Countdown Loop
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
    // Pause/Reset if navigated away
    gameState.value = 'idle'
    if (gameLoop) cancelAnimationFrame(gameLoop)
    if (countdownTimer) clearInterval(countdownTimer)
  }
}, { immediate: true })

const startPush = (direction) => {
  if (gameState.value !== 'playing') return
  pushForce.value = direction * 1.5
}

const stopPush = () => {
  pushForce.value = 0
}

const update = (timestamp) => {
  if (gameState.value !== 'playing') return
  
  if (!lastTime) lastTime = timestamp
  const dt = (timestamp - lastTime) / 16
  lastTime = timestamp

  // 1. Progress (Walking forward)
  progress.value += walkSpeed * dt

  // 2. Physics
  // Drunk noise (random perturbations)
  const noise = (Math.random() - 0.5) * drunkNoise
  
  // Gravity (instability increases as you tilt)
  const gravity = Math.sin(tilt.value * Math.PI / 180) * 0.25

  velocity.value += (gravity + pushForce.value + noise) * dt
  velocity.value *= 0.96 // Friction

  tilt.value += velocity.value * dt

  // 3. Win/Lose Conditions
  // Fall?
  if (Math.abs(tilt.value) > maxTilt) {
    handleFail()
    return
  }

  // Finished?
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
  }, 1500)
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

/* Walking Animation */
@keyframes walkLeft {
  0%, 100% { transform: rotate(15deg); }
  50% { transform: rotate(-15deg); }
}
@keyframes walkRight {
  0%, 100% { transform: rotate(-15deg); }
  50% { transform: rotate(15deg); }
}

.animate-walk-left {
  animation: walkLeft 1s infinite ease-in-out;
}
.animate-walk-right {
  animation: walkRight 1s infinite ease-in-out;
  animation-delay: 0.5s;
}

.paused {
  animation-play-state: paused;
}

.animate-fade-in {
  animation: fadeIn 0.5s ease-out;
}
@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}
</style>
