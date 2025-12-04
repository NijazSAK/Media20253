<template>
  <div class="wakeup-game w-full h-full relative overflow-hidden bg-slate-900 select-none">
    
    <!-- Custom Background -->
    <div v-if="backgroundImage" class="absolute inset-0 z-0">
      <img :src="backgroundImage" class="w-full h-full object-cover opacity-50" />
    </div>

    <!-- Split Screen Background -->
    <div v-else class="absolute inset-0 flex">
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
          <div class="absolute bottom-4 left-1/2 transform -translate-x-1/2 text-white/50 font-bold text-2xl">
            {{ ['←', '↓', '↑', '→'][col-1] }}
          </div>
        </div>

        <!-- Falling Tiles -->
        <div 
          v-for="tile in tiles" 
          :key="tile.id"
          class="absolute w-[25%] h-32 bg-black rounded-md transition-colors border-2 border-white/20 z-10 flex items-center justify-center"
          :style="{ 
            left: `${(tile.col - 1) * 25}%`, 
            top: `${tile.y}%`,
            opacity: tile.hit ? 0 : 1,
            borderColor: tile.hit ? '#4ade80' : 'rgba(255,255,255,0.2)'
          }"
        >
          <!-- Arrow Icon on Tile -->
          <span class="text-white/50 text-2xl font-bold">{{ ['←', '↓', '↑', '→'][tile.col-1] }}</span>
        </div>
      </div>
    </div>

    <!-- UI Overlay -->
    <div class="absolute top-4 left-1/2 transform -translate-x-1/2 z-30 flex flex-col items-center">
      <div class="text-4xl font-bold text-slate-800 drop-shadow-md">{{ timeLeft.toFixed(1) }}s</div>
      <div class="text-sm font-bold text-slate-800 uppercase tracking-widest bg-white/80 px-4 py-1 rounded-full backdrop-blur-sm shadow-sm mt-2">
        Use Arrow Keys!
      </div>
      <!-- Feedback Text -->
      <div 
        v-if="feedbackText"
        class="absolute top-20 text-4xl font-black italic tracking-tighter text-transparent bg-clip-text bg-gradient-to-b from-yellow-300 to-orange-500 drop-shadow-lg animate-pop-out"
        :key="feedbackKey"
      >
        {{ feedbackText }}
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
  isActive: Boolean,
  backgroundImage: String
})

const emit = defineEmits(['complete'])

// Game State
const tiles = ref([])
const isPlaying = ref(false)
const timeLeft = ref(10)
const playerBlinked = ref(false)
const opponentBlinked = ref(false)
const activeKey = ref(null)
const feedbackText = ref('')
const feedbackKey = ref(0)

// Config
const spawnRate = 300 // ms
const fallSpeed = 1.2 // % per frame
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
  feedbackText.value = ''
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
    // Feedback
    const dist = Math.abs(target.y - hitLineY)
    if (dist < 5) showFeedback("PERFECT!")
    else showFeedback("GOOD!")
  } else {
    // Miss penalty?
    // showFeedback("MISS!")
  }
}

const showFeedback = (text) => {
  feedbackText.value = text
  feedbackKey.value++
  setTimeout(() => {
    if (feedbackText.value === text) feedbackText.value = ''
  }, 500)
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
  const keyMap = { 
    'arrowleft': 1, 
    'arrowdown': 2, 
    'arrowup': 3, 
    'arrowright': 4 
  }
  const col = keyMap[e.key.toLowerCase()]
  if (col) {
    e.preventDefault() // Prevent scroll
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

.animate-pop-out {
  animation: popOut 0.5s cubic-bezier(0.175, 0.885, 0.32, 1.275) forwards;
}

@keyframes popOut {
  0% { transform: scale(0.5); opacity: 0; }
  50% { transform: scale(1.2); opacity: 1; }
  100% { transform: scale(1); opacity: 0; }
}
</style>
