<template>
  <div class="balance-game relative w-full h-full flex flex-col items-center justify-center bg-blue-50 overflow-hidden">
    <h2 class="text-2xl font-bold mb-8 text-slate-700 animate-fade-in">Keep Balance!</h2>
    
    <!-- Balance Area -->
    <div class="relative w-64 h-64 flex items-end justify-center mb-12">
      <!-- Pivot -->
      <div class="absolute bottom-0 w-0 h-0 border-l-[20px] border-l-transparent border-r-[20px] border-r-transparent border-b-[40px] border-b-slate-800"></div>
      
      <!-- Beam/Character -->
      <div 
        class="w-48 h-12 bg-indigo-500 rounded-lg shadow-lg transition-transform duration-100 ease-linear origin-bottom"
        :style="{ transform: `rotate(${tilt}deg)` }"
      >
        <!-- Eyes to make it look like a character -->
        <div class="absolute top-2 left-4 w-3 h-3 bg-white rounded-full"></div>
        <div class="absolute top-2 right-4 w-3 h-3 bg-white rounded-full"></div>
        <div class="absolute bottom-3 left-1/2 transform -translate-x-1/2 w-8 h-1 bg-white rounded-full opacity-50"></div>
      </div>
    </div>

    <!-- Controls -->
    <div class="flex gap-8">
      <button 
        @mousedown="startPush(-1)" 
        @mouseup="stopPush" 
        @mouseleave="stopPush"
        @touchstart.prevent="startPush(-1)" 
        @touchend.prevent="stopPush"
        class="w-20 h-20 rounded-full bg-indigo-600 text-white text-2xl font-bold shadow-lg active:scale-95 transition-transform flex items-center justify-center"
      >
        L
      </button>
      <button 
        @mousedown="startPush(1)" 
        @mouseup="stopPush" 
        @mouseleave="stopPush"
        @touchstart.prevent="startPush(1)" 
        @touchend.prevent="stopPush"
        class="w-20 h-20 rounded-full bg-indigo-600 text-white text-2xl font-bold shadow-lg active:scale-95 transition-transform flex items-center justify-center"
      >
        R
      </button>
    </div>

    <!-- Progress Bar -->
    <div class="absolute bottom-10 w-64 h-2 bg-gray-200 rounded-full overflow-hidden">
      <div 
        class="h-full bg-green-500 transition-all duration-300"
        :style="{ width: `${(timeBalanced / requiredTime) * 100}%` }"
      ></div>
    </div>
    
    <p class="absolute bottom-4 text-slate-500 text-sm">Hold buttons to balance</p>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'

const emit = defineEmits(['complete'])

const tilt = ref(0)
const velocity = ref(0)
const pushForce = ref(0)
const timeBalanced = ref(0)
const requiredTime = 5000 // 5 seconds to win
const maxTilt = 45
let gameLoop = null
let lastTime = 0

const startPush = (direction) => {
  pushForce.value = direction * 0.8 // Force applied
}

const stopPush = () => {
  pushForce.value = 0
}

const update = (timestamp) => {
  if (!lastTime) lastTime = timestamp
  const dt = (timestamp - lastTime) / 16 // Normalize to ~60fps
  lastTime = timestamp

  // Physics
  // Gravity pulls tilt further away from 0
  const gravity = Math.sin(tilt.value * Math.PI / 180) * 0.15
  
  velocity.value += (gravity + pushForce.value) * dt
  velocity.value *= 0.95 // Friction
  
  tilt.value += velocity.value * dt

  // Bounds
  if (tilt.value > maxTilt) tilt.value = maxTilt
  if (tilt.value < -maxTilt) tilt.value = -maxTilt

  // Check Balance
  if (Math.abs(tilt.value) < 10) {
    timeBalanced.value += 16.6 * dt
  } else {
    timeBalanced.value = Math.max(0, timeBalanced.value - 50) // Penalty for losing balance
  }

  // Win Condition
  if (timeBalanced.value >= requiredTime) {
    emit('complete')
    cancelAnimationFrame(gameLoop)
    return
  }

  gameLoop = requestAnimationFrame(update)
}

onMounted(() => {
  // Initial random tilt
  tilt.value = (Math.random() - 0.5) * 30
  gameLoop = requestAnimationFrame(update)
})

onUnmounted(() => {
  cancelAnimationFrame(gameLoop)
})
</script>

<style scoped>
.animate-fade-in {
  animation: fadeIn 1s ease-out;
}
@keyframes fadeIn {
  from { opacity: 0; transform: translateY(-20px); }
  to { opacity: 1; transform: translateY(0); }
}
</style>
