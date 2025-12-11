<template>
  <div class="balance-game relative w-full h-full flex flex-col items-center justify-center bg-black overflow-hidden perspective-container">
    
    <!-- Dynamic Backgrounds -->
    <div class="absolute inset-0 z-0">
      <transition-group name="fade">
        <img 
          v-if="currentBackgroundIndex === 0" 
          key="bg1"
          :src="backgrounds[0]" 
          class="absolute inset-0 w-full h-full object-cover transition-opacity duration-1000"
        />
        <img 
          v-if="currentBackgroundIndex === 1" 
          key="bg2"
          :src="backgrounds[1]" 
          class="absolute inset-0 w-full h-full object-cover transition-opacity duration-1000"
        />
        <img 
          v-if="currentBackgroundIndex === 2" 
          key="bg3"
          :src="backgrounds[2]" 
          class="absolute inset-0 w-full h-full object-cover transition-opacity duration-1000"
        />
      </transition-group>
      <div class="absolute inset-0 bg-black/30"></div>
    </div>

    <!-- Game Layer -->
    <div class="relative w-full h-full max-w-4xl mx-auto flex items-center justify-center perspective-view transition-opacity duration-1000" :class="{ 'opacity-100': isActive, 'opacity-0': !isActive }">
      
      <!-- Character/Table Container -->
      <div 
        class="character-rig absolute bottom-10 transition-transform duration-100 ease-linear"
        :style="{ 
          transform: `scale(${1 - (progress * 0.3)}) translateY(${-progress * 50}px) rotate(${tilt}deg)`,
          opacity: isFallen ? 0 : 1
        }"
      >
        <!-- Table Sprite -->
        <div class="relative w-48 h-48 flex flex-col items-center justify-end">
           <img :src="tableSprite" class="w-full h-full object-contain drop-shadow-2xl" alt="Balance Object" />
        </div>
      </div>

      <!-- Balance Bar Indicator -->
      <div v-if="gameState === 'playing'" class="absolute top-24 w-64 h-4 bg-slate-800/80 rounded-full border border-slate-600 overflow-hidden backdrop-blur-sm">
        <!-- Center Marker -->
        <div class="absolute left-1/2 top-0 bottom-0 w-0.5 bg-white/50 transform -translate-x-1/2 z-10"></div>
        
        <!-- Bar Fill -->
        <div 
          class="h-full transition-all duration-75 ease-linear"
          :class="tilt > 0 ? 'bg-red-500/80' : 'bg-blue-500/80'"
          :style="{ 
            width: `${Math.min(Math.abs(tilt) / maxTilt * 50, 50)}%`,
            marginLeft: tilt > 0 ? '50%' : `calc(50% - ${Math.min(Math.abs(tilt) / maxTilt * 50, 50)}%)`
          }"
        ></div>
      </div>

      <!-- Countdown Overlay -->
      <div v-if="gameState === 'countdown'" class="absolute inset-0 flex items-center justify-center z-50">
        <div class="text-9xl font-bold text-white animate-pulse drop-shadow-[0_0_20px_rgba(255,255,255,0.8)]">
          {{ countdown }}
        </div>
        <div class="absolute bottom-1/4 text-white/70 text-xl tracking-widest uppercase">Keep it steady...</div>
      </div>

      <!-- Fall Message -->
      <div v-if="isFallen" class="absolute inset-0 flex items-center justify-center bg-black/80 z-50 animate-fade-in">
        <h2 class="text-4xl font-bold text-red-500 tracking-widest uppercase">Spilled...</h2>
      </div>

    </div>

    <!-- UI / Controls -->
    <div class="absolute bottom-8 w-full flex flex-col items-center gap-4 z-50 transition-opacity duration-500" :class="{ 'opacity-100': gameState === 'playing', 'opacity-0': gameState !== 'playing' }">
      <div class="w-64 h-2 bg-slate-700 rounded-full overflow-hidden border border-slate-600">
        <div class="h-full bg-green-500 transition-all duration-300" :style="{ width: `${progress * 100}%` }"></div>
      </div>
      
      <div class="flex gap-12 text-white/50 font-mono text-sm">
        <span :class="{ 'text-white font-bold': pushForce < 0 }">&larr; LEFT</span>
        <span :class="{ 'text-white font-bold': pushForce > 0 }">RIGHT &rarr;</span>
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
import { ref, onMounted, onUnmounted, watch, computed } from 'vue'
import tableSprite from '../assets/table-sprite.png'

const props = defineProps({
  isActive: Boolean,
  backgrounds: {
    type: Array,
    default: () => []
  }
})

const emit = defineEmits(['complete'])

const tilt = ref(0)
const velocity = ref(0)
const pushForce = ref(0)
const progress = ref(0) // 0 to 1
const isFallen = ref(false)
const gameState = ref('idle') // idle, countdown, playing, ended
const countdown = ref(3)

// Game Config
const walkSpeed = 0.0010 // Slower progress for tension
const maxTilt = 45
const drunkNoise = 0.2 
const correctionForce = 1.5 // Constant force applied when key is held
const friction = 0.90 // High friction for "direct" control feel

let gameLoop = null
let lastTime = 0
let countdownTimer = null

const currentBackgroundIndex = computed(() => {
  if (progress.value < 0.33) return 0
  if (progress.value < 0.66) return 1
  return 2
})

const startGameSequence = () => {
  // Reset
  tilt.value = 0
  velocity.value = 0
  pushForce.value = 0
  progress.value = 0
  isFallen.value = false
  gameState.value = 'countdown'
  countdown.value = 3
  
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
  // Constant force, not additive
  pushForce.value = direction * correctionForce
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

  // 2. Physics
  const noise = (Math.random() - 0.5) * drunkNoise
  const gravity = Math.sin(tilt.value * Math.PI / 180) * 0.3

  // Velocity update with high friction
  velocity.value += (gravity + pushForce.value + noise) * dt
  velocity.value *= friction

  tilt.value += velocity.value * dt

  // 3. Win/Lose Conditions
  if (Math.abs(tilt.value) > maxTilt) {
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

.animate-fade-in {
  animation: fadeIn 0.5s ease-out;
}
@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 1s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>
