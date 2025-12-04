<template>
  <div class="wakeup-game w-full h-full relative overflow-hidden bg-slate-900 select-none">
    
    <!-- Split Screen Background -->
    <div class="absolute inset-0 flex">
      <!-- Opponent Side (Top-Left) -->
      <div class="w-full h-full relative bg-amber-200 overflow-hidden clip-diagonal-top">
        <div class="absolute bottom-10 right-20 flex flex-col items-center">
          <!-- Opponent Face -->
          <div class="w-40 h-40 bg-white rounded-full border-4 border-black relative">
            <div class="absolute top-12 left-8 w-4 h-4 bg-black rounded-full" :class="{ 'scale-y-0': opponentBlinked }"></div>
            <div class="absolute top-12 right-8 w-4 h-4 bg-black rounded-full" :class="{ 'scale-y-0': opponentBlinked }"></div>
            <div class="absolute bottom-10 left-1/2 transform -translate-x-1/2 w-12 h-1 bg-black rounded-full"></div>
            <!-- Sweat drop if losing? -->
          </div>
          <div class="w-1 h-20 bg-black"></div>
        </div>
      </div>

      <!-- Player Side (Bottom-Right) -->
      <div class="absolute inset-0 w-full h-full bg-blue-100 clip-diagonal-bottom pointer-events-none">
        <div class="absolute top-10 left-20 flex flex-col items-center">
           <!-- Player Face -->
           <div class="w-40 h-40 bg-white rounded-full border-4 border-black relative">
            <div class="absolute top-12 left-8 w-4 h-4 bg-black rounded-full transition-transform duration-100" :class="{ 'scale-y-0': playerBlinked }"></div>
            <div class="absolute top-12 right-8 w-4 h-4 bg-black rounded-full transition-transform duration-100" :class="{ 'scale-y-0': playerBlinked }"></div>
            <div class="absolute bottom-10 left-1/2 transform -translate-x-1/2 w-12 h-1 bg-black rounded-full"></div>
          </div>
          <div class="w-1 h-20 bg-black"></div>
        </div>
      </div>
    </div>

    <!-- Diagonal Divider Line -->
    <div class="absolute inset-0 pointer-events-none">
      <svg width="100%" height="100%">
        <line x1="100%" y1="0" x2="0" y2="100%" stroke="black" stroke-width="4" />
      </svg>
    </div>

    <!-- Piano Tiles Layer -->
    <div class="absolute inset-0 flex justify-center z-20">
      <div class="w-full max-w-md h-full relative flex">
        <!-- 4 Columns -->
        <div v-for="col in 4" :key="col" class="flex-1 h-full border-r border-white/10 relative group">
          <!-- Hit Zone Indicator -->
          <div class="absolute bottom-0 w-full h-32 bg-gradient-to-t from-green-500/20 to-transparent opacity-0 transition-opacity duration-100" :class="{ 'opacity-100': activeKey === col }"></div>
          
          <!-- Key Hint -->
          <div class="absolute bottom-4 left-1/2 transform -translate-x-1/2 text-white/50 font-bold text-xl">
            {{ ['D', 'F', 'J', 'K'][col-1] }}
          </div>
        </div>

        <!-- Falling Tiles -->
        <div 
          v-for="tile in tiles" 
          :key="tile.id"
          class="absolute w-[25%] h-32 bg-black rounded-md cursor-pointer hover:bg-slate-800 active:bg-slate-700 transition-colors border-2 border-white/20"
          :style="{ 
            left: `${(tile.col - 1) * 25}%`, 
            top: `${tile.y}%`,
            opacity: tile.hit ? 0 : 1
          }"
          @mousedown="hitTile(tile)"
          @touchstart.prevent="hitTile(tile)"
        ></div>
      </div>
    </div>

    <!-- UI Overlay -->
    <div class="absolute top-4 left-1/2 transform -translate-x-1/2 z-30 flex flex-col items-center">
      <div class="text-4xl font-bold text-slate-800 drop-shadow-md">{{ timeLeft.toFixed(1) }}s</div>
      <div class="text-sm font-bold text-slate-600 uppercase tracking-widest">Don't Blink!</div>
    </div>

    <!-- Fail Overlay -->
    <div v-if="playerBlinked" class="absolute inset-0 bg-black/80 z-50 flex items-center justify-center">
      <h2 class="text-red-500 text-5xl font-bold animate-pulse">BLINKED!</h2>
    </div>

  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted, watch } from 'vue'

const props = defineProps({
  isActive: Boolean
})

const emit = defineEmits(['complete'])

// Game State
const tiles = ref([])
const isPlaying = ref(false)
const timeLeft = ref(10)
const playerBlinked = ref(false)
const opponentBlinked = ref(false)
const activeKey = ref(null)

// Config
const spawnRate = 600 // ms
const fallSpeed = 0.6 // % per frame
let gameLoop = null
let spawner = null
let tileIdCounter = 0

// Lifecycle
watch(() => props.isActive, (active) => {
  if (active) {
    resetGame()
    startGame()
  } else {
    stopGame()
  }
})

const resetGame = () => {
  tiles.value = []
  timeLeft.value = 10
  playerBlinked.value = false
  opponentBlinked.value = false
  isPlaying.value = false
}

const startGame = () => {
  isPlaying.value = true
  
  // Spawn Loop
  spawner = setInterval(() => {
    if (!isPlaying.value) return
    const col = Math.floor(Math.random() * 4) + 1
    tiles.value.push({
      id: tileIdCounter++,
      col,
      y: -20, // Start above screen
      hit: false
    })
  }, spawnRate)

  // Game Loop
  gameLoop = requestAnimationFrame(update)
}

const stopGame = () => {
  isPlaying.value = false
  clearInterval(spawner)
  cancelAnimationFrame(gameLoop)
}

const update = () => {
  if (!isPlaying.value) return

  // Time
  timeLeft.value -= 1/60
  if (timeLeft.value <= 0) {
    handleWin()
    return
  }

  // Move Tiles
  tiles.value.forEach(tile => {
    if (tile.hit) return
    tile.y += fallSpeed
    
    // Miss Condition
    if (tile.y > 100) {
      handleFail()
    }
  })

  // Cleanup
  tiles.value = tiles.value.filter(t => t.y < 120 && !(t.hit && t.y > 100)) // Keep hit tiles briefly? No just hide them

  gameLoop = requestAnimationFrame(update)
}

const hitTile = (tile) => {
  if (!isPlaying.value || tile.hit) return
  
  // Check if tile is in hit zone (e.g., bottom 30%)
  // Actually standard piano tiles allows hitting anywhere usually, but let's say it has to be on screen
  tile.hit = true
  
  // Visual feedback
  activeKey.value = tile.col
  setTimeout(() => activeKey.value = null, 100)
}

const handleInput = (col) => {
  if (!isPlaying.value) return
  
  activeKey.value = col
  setTimeout(() => activeKey.value = null, 100)

  // Find lowest unhit tile in column
  const target = tiles.value
    .filter(t => t.col === col && !t.hit)
    .sort((a, b) => b.y - a.y)[0] // Lowest first

  if (target) {
    // Check accuracy? For now just hit it
    target.hit = true
  } else {
    // Penalty for tapping empty lane? 
    // Maybe slight blink flicker?
  }
}

const handleFail = () => {
  playerBlinked.value = true
  stopGame()
  setTimeout(() => {
    // Retry logic? Or just fail scene?
    // Let's restart for now or emit fail
    resetGame()
    startGame() 
  }, 1500)
}

const handleWin = () => {
  opponentBlinked.value = true
  stopGame()
  setTimeout(() => {
    emit('complete')
  }, 1500)
}

const handleKeydown = (e) => {
  const keyMap = { 'd': 1, 'f': 2, 'j': 3, 'k': 4 }
  const col = keyMap[e.key.toLowerCase()]
  if (col) {
    handleInput(col)
  }
}

onMounted(() => {
  window.addEventListener('keydown', handleKeydown)
})

onUnmounted(() => {
  stopGame()
  window.removeEventListener('keydown', handleKeydown)
})
</script>

<style scoped>
.clip-diagonal-top {
  clip-path: polygon(0 0, 100% 0, 0 100%);
}
.clip-diagonal-bottom {
  clip-path: polygon(100% 0, 100% 100%, 0 100%);
}
</style>
