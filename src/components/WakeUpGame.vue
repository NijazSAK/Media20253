<template>
  <div class="wakeup-game w-full h-full relative overflow-hidden bg-slate-900 select-none">
    
    <!-- Split Screen Background -->
    <div class="absolute inset-0 flex">
      <!-- Opponent Side (Top-Left) -->
      <div class="w-full h-full relative bg-amber-200 overflow-hidden clip-diagonal-top">
        <div class="absolute bottom-20 right-32 flex flex-col items-center transform scale-125">
          <!-- Opponent Face -->
          <div class="w-40 h-40 bg-white rounded-full border-4 border-black relative overflow-hidden">
            <!-- Eyes -->
            <div class="absolute top-12 left-8 w-4 h-4 bg-black rounded-full"></div>
            <div class="absolute top-12 right-8 w-4 h-4 bg-black rounded-full"></div>
            <!-- Eyelids (Struggle Animation) -->
            <div class="absolute top-0 left-0 w-full bg-amber-200 z-10 transition-all duration-100" 
                 :class="{ 'animate-struggle': isPlaying && !opponentBlinked, 'h-1/2': opponentBlinked, 'h-0': !opponentBlinked && !isPlaying }"></div>
            
            <!-- Mouth -->
            <div class="absolute bottom-10 left-1/2 transform -translate-x-1/2 w-12 h-1 bg-black rounded-full"></div>
            
            <!-- Sweat -->
            <div v-if="isPlaying" class="absolute top-4 right-8 text-blue-400 text-2xl animate-bounce">💧</div>
          </div>
          <div class="w-1 h-20 bg-black"></div>
        </div>
      </div>

      <!-- Player Side (Bottom-Right) -->
      <div class="absolute inset-0 w-full h-full bg-blue-100 clip-diagonal-bottom pointer-events-none">
        <div class="absolute top-20 left-32 flex flex-col items-center transform scale-125">
           <!-- Player Face -->
           <div class="w-40 h-40 bg-white rounded-full border-4 border-black relative overflow-hidden">
            <!-- Eyes -->
            <div class="absolute top-12 left-8 w-4 h-4 bg-black rounded-full"></div>
            <div class="absolute top-12 right-8 w-4 h-4 bg-black rounded-full"></div>
            <!-- Eyelids (Struggle Animation) -->
            <div class="absolute top-0 left-0 w-full bg-blue-100 z-10 transition-all duration-100"
                 :class="{ 'animate-struggle': isPlaying && !playerBlinked, 'h-1/2': playerBlinked, 'h-0': !playerBlinked && !isPlaying }"></div>

            <!-- Mouth -->
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
        
        <!-- Hit Line -->
        <div class="absolute top-[85%] w-full h-1 bg-red-500 shadow-[0_0_10px_rgba(255,0,0,0.8)] z-0"></div>
        <div class="absolute top-[85%] right-full mr-4 text-white font-bold text-sm whitespace-nowrap">HIT HERE &rarr;</div>

        <!-- 4 Columns -->
        <div v-for="col in 4" :key="col" class="flex-1 h-full border-r border-white/10 relative group">
          <!-- Hit Zone Indicator -->
          <div class="absolute top-[75%] bottom-0 w-full bg-gradient-to-b from-transparent via-white/10 to-transparent opacity-0 transition-opacity duration-100" :class="{ 'opacity-100': activeKey === col }"></div>
          
          <!-- Key Hint -->
          <div class="absolute bottom-4 left-1/2 transform -translate-x-1/2 text-white/50 font-bold text-xl">
            {{ ['D', 'F', 'J', 'K'][col-1] }}
          </div>
        </div>

        <!-- Falling Tiles -->
        <div 
          v-for="tile in tiles" 
          :key="tile.id"
          class="absolute w-[25%] h-32 bg-black rounded-md cursor-pointer hover:bg-slate-800 active:bg-slate-700 transition-colors border-2 border-white/20 z-10"
          :style="{ 
            left: `${(tile.col - 1) * 25}%`, 
            top: `${tile.y}%`,
            opacity: tile.hit ? 0 : 1,
            borderColor: tile.hit ? '#4ade80' : 'rgba(255,255,255,0.2)'
          }"
          @mousedown="hitTile(tile)"
          @touchstart.prevent="hitTile(tile)"
        ></div>
      </div>
    </div>

    <!-- UI Overlay -->
    <div class="absolute top-4 left-1/2 transform -translate-x-1/2 z-30 flex flex-col items-center">
      <div class="text-4xl font-bold text-slate-800 drop-shadow-md">{{ timeLeft.toFixed(1) }}s</div>
      <div class="text-sm font-bold text-slate-800 uppercase tracking-widest bg-white/80 px-4 py-1 rounded-full backdrop-blur-sm shadow-sm">
        Hit tiles on the line!
      </div>
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
const spawnRate = 500 // ms
const fallSpeed = 0.8 // % per frame
const hitLineY = 85 // %
const hitTolerance = 12 // % +/-
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
    
    // Miss Condition (Passed the line + tolerance)
    if (tile.y > hitLineY + hitTolerance && !tile.hit) {
      handleFail()
    }
  })

  // Cleanup
  tiles.value = tiles.value.filter(t => t.y < 120)

  gameLoop = requestAnimationFrame(update)
}

const hitTile = (tile) => {
  if (!isPlaying.value || tile.hit) return
  
  // Check Hit Zone
  const dist = Math.abs(tile.y + 10 - hitLineY) // +10 to center hit on tile roughly (tile is 32px height, % is tricky, let's approx)
  // Actually tile.y is top. Tile height is 32px (fixed) or %? 
  // In template: h-32 (8rem). Screen height varies.
  // Let's use simple logic: if tile top is within range [hitLineY - 20, hitLineY + 5]
  
  // Let's be generous: if tile overlaps line.
  // Tile top: tile.y. Tile bottom: tile.y + ~15 (approx 15% height).
  // Line: 85.
  // Hit if tile.y > 70 && tile.y < 90
  
  if (tile.y > (hitLineY - 20) && tile.y < (hitLineY + 5)) {
    tile.hit = true
    activeKey.value = tile.col
    setTimeout(() => activeKey.value = null, 100)
  } else {
    // Missed click (too early/late)
    // Optional: Penalty?
  }
}

const handleInput = (col) => {
  if (!isPlaying.value) return
  
  activeKey.value = col
  setTimeout(() => activeKey.value = null, 100)

  // Find lowest unhit tile in column that is in range
  const target = tiles.value
    .filter(t => t.col === col && !t.hit && t.y > (hitLineY - 20) && t.y < (hitLineY + 5))
    .sort((a, b) => b.y - a.y)[0] // Lowest first

  if (target) {
    target.hit = true
  }
}

const handleFail = () => {
  playerBlinked.value = true
  stopGame()
  setTimeout(() => {
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

.animate-struggle {
  animation: struggle 2s infinite;
}

@keyframes struggle {
  0%, 100% { height: 0%; }
  5% { height: 40%; }
  10% { height: 0%; }
  40% { height: 0%; }
  45% { height: 30%; }
  50% { height: 0%; }
  80% { height: 0%; }
  82% { height: 20%; }
  84% { height: 0%; }
}
</style>
