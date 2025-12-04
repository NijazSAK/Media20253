<template>
  <div class="app-container w-full h-screen overflow-hidden bg-slate-900 text-slate-200 font-sans">
    <!-- Main Scroll Container -->
    <!-- Progress Indicators -->
    <div class="fixed right-6 top-1/2 transform -translate-y-1/2 z-50 flex flex-col gap-3">
      <div 
        v-for="(scene, index) in scenes" 
        :key="index"
        class="w-2 h-2 rounded-full transition-all duration-500 border border-white/50"
        :class="{ 
          'bg-white scale-125': index === currentSceneIndex,
          'bg-white/20': index !== currentSceneIndex,
          'bg-green-400 border-green-400': scene.completed
        }"
      ></div>
    </div>

    <div 
      ref="container"
      class="w-full h-full relative transition-transform duration-1000 cubic-bezier(0.22, 1, 0.36, 1)"
      :style="{ transform: `translateY(-${currentSceneIndex * 100}%)` }"
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
          class="absolute top-6 right-6 z-40 px-4 py-2 bg-black/40 text-white/70 rounded-full hover:bg-white/10 hover:text-white transition-all border border-white/10 font-mono text-xs backdrop-blur-md uppercase tracking-wider"
        >
          Skip Level
        </button>
        
        <!-- Navigation Hint -->
        <div 
          v-if="scene.completed && index < scenes.length - 1"
          class="absolute bottom-12 left-1/2 transform -translate-x-1/2 animate-bounce flex flex-col items-center gap-2 z-40 cursor-pointer"
          @click="currentSceneIndex++"
        >
          <span class="text-white/80 text-xs font-light tracking-[0.2em] uppercase bg-black/20 px-4 py-2 rounded-full backdrop-blur-sm border border-white/10 hover:bg-white/10 transition-colors">
            Scroll Down
          </span>
          <svg class="w-5 h-5 text-white/50" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.5" d="M19 14l-7 7m0 0l-7-7m7 7V3"></path></svg>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, shallowRef, onMounted, onUnmounted, watch } from 'vue'
import Scene from './components/Scene.vue'
import BalanceGame from './components/BalanceGame.vue'
import LanguageGame from './components/LanguageGame.vue'
import WakeUpGame from './components/WakeUpGame.vue'
import EndScene from './components/EndScene.vue'

// ... (scenes array remains the same, no changes needed there) ...

const currentSceneIndex = ref(0)
const container = ref(null)

// Preload Next Scene Images
watch(currentSceneIndex, (newIndex) => {
  const nextScene = scenes.value[newIndex + 1]
  if (nextScene && nextScene.props.src) {
    const sources = Array.isArray(nextScene.props.src) ? nextScene.props.src : [nextScene.props.src]
    sources.forEach(src => {
      const img = new Image()
      img.src = src
    })
  }
})

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
}

const handleKeydown = (e) => {
  const currentScene = scenes.value[currentSceneIndex.value]

  // Navigation (Down Arrow)
  if (e.key === 'ArrowDown') {
    if (currentScene.completed && currentSceneIndex.value < scenes.value.length - 1) {
      currentSceneIndex.value++
    }
  }

  // Skip Game Cheat
  if (e.key.toLowerCase() === 's') {
    if (currentScene.isGame && !currentScene.completed) {
      unlockScene(currentSceneIndex.value, { success: true })
    }
  }
}

onMounted(() => {
  window.addEventListener('keydown', handleKeydown)
  // Preload first scene
  if (scenes.value[0].props.src) {
     const sources = Array.isArray(scenes.value[0].props.src) ? scenes.value[0].props.src : [scenes.value[0].props.src]
     sources.forEach(src => { const img = new Image(); img.src = src })
  }
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
