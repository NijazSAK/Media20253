<template>
  <div class="wakeup-game w-full h-full relative overflow-hidden bg-black cursor-pointer" @click="handleClick">
    <!-- Blurred Scene Background -->
    <div 
      class="absolute inset-0 bg-cover bg-center transition-all duration-100"
      style="background-image: url('https://picsum.photos/800/600?grayscale');"
      :style="{ filter: `blur(${10 - (openness / 10)}px) brightness(${openness / 100})` }"
    ></div>

    <!-- Eyelids -->
    <div 
      class="absolute top-0 left-0 w-full bg-black z-10 transition-all duration-100 ease-out"
      :style="{ height: `${50 - (openness / 2)}%` }"
    >
      <div class="absolute bottom-0 w-full h-4 bg-gradient-to-t from-transparent to-black opacity-50"></div>
    </div>
    <div 
      class="absolute bottom-0 left-0 w-full bg-black z-10 transition-all duration-100 ease-out"
      :style="{ height: `${50 - (openness / 2)}%` }"
    >
      <div class="absolute top-0 w-full h-4 bg-gradient-to-b from-transparent to-black opacity-50"></div>
    </div>

    <!-- Instructions -->
    <div class="absolute inset-0 flex items-center justify-center z-20 pointer-events-none">
      <h2 
        v-if="openness < 80" 
        class="text-white text-4xl font-bold opacity-50 animate-pulse"
      >
        TAP TO WAKE UP
      </h2>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'

const emit = defineEmits(['complete'])

const openness = ref(0) // 0 to 100
const decayRate = 0.8
let gameLoop = null

const handleClick = () => {
  openness.value = Math.min(100, openness.value + 15)
}

const update = () => {
  // Constant decay (closing eyes)
  if (openness.value > 0) {
    openness.value = Math.max(0, openness.value - decayRate)
  }

  // Win condition: Eyes fully open for a moment
  if (openness.value >= 98) {
    // Hold it for a split second logic could go here, but for now instant win if fully open
    emit('complete')
    cancelAnimationFrame(gameLoop)
    return
  }

  gameLoop = requestAnimationFrame(update)
}

onMounted(() => {
  gameLoop = requestAnimationFrame(update)
})

onUnmounted(() => {
  cancelAnimationFrame(gameLoop)
})
</script>
