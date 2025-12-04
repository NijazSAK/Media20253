<template>
  <div class="app-container w-full h-screen overflow-hidden bg-slate-900 text-slate-200 font-sans">
    <!-- Main Scroll Container -->
    <div 
      ref="container"
      class="w-full h-full relative transition-transform duration-700 ease-in-out"
      :style="{ transform: `translateY(-${currentSceneIndex * 100}%)` }"
      @wheel="handleScroll"
      @touchstart="handleTouchStart"
      @touchend="handleTouchEnd"
    >
      <!-- Render Scenes -->
      <div 
        v-for="(scene, index) in scenes" 
        :key="index"
        class="w-full h-full relative"
      >
        <!-- Dynamic Component Rendering -->
        <component 
          :is="scene.component" 
          v-bind="scene.props" 
          :isActive="index === currentSceneIndex"
          @complete="unlockScene(index, $event)"
        />

        <!-- Skip Button -->
        <button 
          v-if="scene.isGame && !scene.completed" 
          @click="unlockScene(index, { success: true })"
          class="absolute top-4 right-4 z-50 px-4 py-2 bg-slate-800/80 text-white rounded-lg hover:bg-slate-700 transition-colors border border-slate-600 font-mono text-sm backdrop-blur-sm"
        >
          SKIP [S]
        </button>
        
        <!-- Navigation Hint -->
        <div 
          v-if="!scene.isGame || scene.completed"
          class="absolute bottom-4 left-1/2 transform -translate-x-1/2 animate-bounce opacity-50"
        >
          <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 14l-7 7m0 0l-7-7m7 7V3"></path></svg>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, shallowRef, onMounted, onUnmounted } from 'vue'
import Scene from './components/Scene.vue'
import BalanceGame from './components/BalanceGame.vue'
import LanguageGame from './components/LanguageGame.vue'
import WakeUpGame from './components/WakeUpGame.vue'

// Scene Configuration
const scenes = ref([
  { 
    component: shallowRef(Scene), 
    props: { src: 'https://picsum.photos/id/10/1920/1080', caption: 'It started like any other day...' },
    isGame: false 
  },
  { 
    component: shallowRef(Scene), 
    props: { src: 'https://picsum.photos/id/11/1920/1080', caption: 'But something felt off balance.' },
    isGame: false 
  },
  { 
    component: shallowRef(BalanceGame), 
    props: {}, 
    isGame: true,
    completed: false 
  },
  { 
    component: shallowRef(Scene), 
    props: { src: 'https://picsum.photos/id/12/1920/1080', caption: 'Words couldn\'t express the feeling.' },
    isGame: false 
  },
  { 
    component: shallowRef(LanguageGame), 
    props: {}, 
    isGame: true,
    completed: false 
  },
  { 
    component: shallowRef(Scene), 
    props: { src: 'https://picsum.photos/id/13/1920/1080', caption: 'Exhaustion took over.' },
    isGame: false 
  },
  { 
    component: shallowRef(Scene), 
    props: { src: 'https://picsum.photos/id/14/1920/1080', caption: 'Trying to stay awake...' },
    isGame: false 
  },
  { 
    component: shallowRef(WakeUpGame), 
    props: {}, 
    isGame: true,
    completed: false 
  },
  { 
    component: shallowRef(Scene), 
    props: { src: 'https://picsum.photos/id/15/1920/1080', caption: 'Finally, clarity.' },
    isGame: false 
  }
])

const currentSceneIndex = ref(0)
const isScrolling = ref(false)

// Unlock Logic
const unlockScene = (index, payload) => {
  scenes.value[index].completed = true
  
  // Handle Balance Game Outcome (Index 2)
  if (index === 2 && scenes.value[3]) {
    if (payload && payload.success === false) {
      scenes.value[3].props.caption = "I stumbled and fell, but I had to keep moving."
    } else {
      scenes.value[3].props.caption = "I made it through the alley, steady on my feet."
    }
  }

  // Auto advance after a brief delay for satisfaction
  setTimeout(() => {
    if (currentSceneIndex.value < scenes.value.length - 1) {
      currentSceneIndex.value++
    }
  }, 1000)
}

// Scroll Handling
const handleScroll = (e) => {
  if (isScrolling.value) return
  
  // Debounce
  isScrolling.value = true
  setTimeout(() => isScrolling.value = false, 800)

  const direction = e.deltaY > 0 ? 1 : -1
  attemptNavigation(direction)
}

// Touch Handling
let touchStartY = 0
const handleTouchStart = (e) => {
  touchStartY = e.touches[0].clientY
}
const handleTouchEnd = (e) => {
  const touchEndY = e.changedTouches[0].clientY
  const diff = touchStartY - touchEndY
  
  if (Math.abs(diff) > 50) { // Threshold
    const direction = diff > 0 ? 1 : -1
    attemptNavigation(direction)
  }
}

const attemptNavigation = (direction) => {
  const currentScene = scenes.value[currentSceneIndex.value]
  
  // Prevent moving forward if current scene is an incomplete game
  if (direction > 0 && currentScene.isGame && !currentScene.completed) {
    // Shake effect or visual feedback could go here
    return
  }

  const newIndex = currentSceneIndex.value + direction
  if (newIndex >= 0 && newIndex < scenes.value.length) {
    currentSceneIndex.value = newIndex
  }
}

const handleKeydown = (e) => {
  // Skip Game Cheat
  if (e.key.toLowerCase() === 's') {
    const currentScene = scenes.value[currentSceneIndex.value]
    if (currentScene.isGame && !currentScene.completed) {
      unlockScene(currentSceneIndex.value)
    }
  }
}

onMounted(() => {
  window.addEventListener('keydown', handleKeydown)
})

onUnmounted(() => {
  window.removeEventListener('keydown', handleKeydown)
})
</script>

<style>
/* Global Reset */
body, html {
  margin: 0;
  padding: 0;
  overflow: hidden; /* Prevent native scroll */
}
</style>
