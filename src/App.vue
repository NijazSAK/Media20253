<template>
  <div class="app-container w-full h-screen overflow-hidden bg-slate-900 text-slate-200 font-sans">
    <!-- Main Scene Container -->
    <div class="w-full h-full relative">
      
      <!-- Global Header -->
      <header class="absolute top-0 left-0 w-full z-[60] p-6 flex items-start pointer-events-none">
        <a 
          href="https://new-test-one-tau.vercel.app/" 
          class="pointer-events-auto group flex items-center gap-3 text-white/60 hover:text-white transition-all duration-300 ease-out"
        >
          <span class="w-8 h-8 rounded-full border border-white/20 flex items-center justify-center group-hover:border-white/80 group-hover:bg-white/10 transition-all">
            <svg xmlns="http://www.w3.org/2000/svg" class="w-4 h-4 transform group-hover:-translate-x-0.5 transition-transform" fill="none" viewBox="0 0 24 24" stroke="currentColor">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10 19l-7-7m0 0l7-7m-7 7h18" />
            </svg>
          </span>
          <span class="font-mono text-xs uppercase tracking-[0.2em] opacity-0 -translate-x-2 group-hover:opacity-100 group-hover:translate-x-0 transition-all duration-300">
            Return to Hub
          </span>
        </a>
      </header>

      <!-- Current Scene -->
      <transition name="fade" mode="out-in">
        <component 
          :is="scenes[currentSceneIndex].component" 
          v-bind="scenes[currentSceneIndex].props" 
          :isActive="true"
          @complete="unlockScene(currentSceneIndex, $event)"
        />
      </transition>

      <!-- Skip Button (Global Overlay) -->
      <button 
        v-if="scenes[currentSceneIndex].isGame && !scenes[currentSceneIndex].completed" 
        @click="unlockScene(currentSceneIndex, { success: true })"
        class="absolute top-4 right-4 z-50 px-4 py-2 bg-slate-800/80 text-white rounded-lg hover:bg-slate-700 transition-colors border border-slate-600 font-mono text-sm backdrop-blur-sm"
      >
        SKIP [S]
      </button>
      
      <!-- Navigation Hint (Global Overlay) -->
      <div 
        v-if="scenes[currentSceneIndex].completed && currentSceneIndex < scenes.length - 1"
        class="absolute bottom-8 left-1/2 transform -translate-x-1/2 animate-bounce flex flex-col items-center gap-2 opacity-80 z-50"
      >
        <span class="text-white text-sm font-mono tracking-widest uppercase bg-black/50 px-3 py-1 rounded-full backdrop-blur-sm">Press ↓</span>
        <svg class="w-6 h-6 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 14l-7 7m0 0l-7-7m7 7V3"></path></svg>
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
import EndScene from './components/EndScene.vue'

import IntroScene from './components/IntroScene.vue'
import introImage1 from './assets/intro-1.png'
import introImage2 from './assets/intro-2.png'
import introImage3 from './assets/intro-3.png'

import balanceBg1 from './assets/balance-bg-1.jpg'
import balanceBg2 from './assets/balance-bg-2.jpg'
import balanceBg3 from './assets/balance-bg-3.jpg'

import BlackScreen from './components/BlackScreen.vue'
import orderingFoodClerk from './assets/ordering-food-clerk.png'
import restaurantEntranceImage from './assets/restaurant-entrance.jpg'
import exhaustedImage from './assets/exhaustion.jpg'
import finalAdImage from './assets/final-ad.jpg'

// Scene Configuration
const scenes = ref([
  {
    component: shallowRef(IntroScene),
    props: {
      slides: [
        { src: introImage3, caption: 'Again... I lie...', duration: 3000 },
        { src: introImage2, caption: 'When can I stop...', duration: 3000 },
        { src: introImage1, caption: '...', duration: 3000 }
      ]
    },
    isGame: false,
    completed: false
  },
  { 
    component: shallowRef(BalanceGame), 
    props: {
      backgrounds: [balanceBg1, balanceBg2, balanceBg3]
    }, 
    isGame: true,
    completed: false,
    autoAdvance: true
  },
  {
    component: shallowRef(Scene),
    props: {
      src: restaurantEntranceImage,
      caption: '...',
      duration: 3000
    },
    isGame: false,
    completed: false,
    autoAdvance: true
  },
  {
    component: shallowRef(BlackScreen),
    props: {
      caption: 'Homecooked would have been better...',
      duration: 5000,
      audioSrc: '/music/bell.mp3'
    },
    isGame: false,
    completed: false,
    autoAdvance: true
  },
  { 
    component: shallowRef(LanguageGame), 
    props: {
      backgroundImage: orderingFoodClerk
    }, 
    isGame: true,
    completed: false 
  },
  { 
    component: shallowRef(Scene), 
    // Darker, slower shots
    props: { 
      src: exhaustionImage,
      caption: 'Is that him?',
      duration: 3000
    },
    isGame: false,
    completed: false
  },

  { 
    component: shallowRef(EndScene), 
    props: { 
      // Same image repeated for each line of the poem
      src: Array(6).fill(finalAdImage),
      audioSrc: '/music/ending.mp3', 
      script: [
        "In shadowed streets where lonely footsteps tread,",
        "We seek a warmth that's more than just the bread.",
        "For hunger deep isn't cured by rush and haste,",
        "But by the memory of a shared taste.",
        "So take this meal, a simple friend in need,",
        "Ten minutes' grace, a soul and body feed."
      ]
    },
    isGame: false,
    completed: false
  }
])

const currentSceneIndex = ref(0)
const isScrolling = ref(false)

// Unlock Logic
const unlockScene = (index, payload) => {
  scenes.value[index].completed = true
  
  // Handle Balance Game Outcome (Index 1)
  if (index === 1 && scenes.value[2]) {
    if (payload && payload.success === false) {
      // scenes.value[2].props.caption = "I stumbled and fell, but I had to keep moving."
    } else {
      // scenes.value[2].props.caption = "I made it through the alley, steady on my feet."
    }
  }

  // Auto-advance if configured
  if (scenes.value[index].autoAdvance && index < scenes.value.length - 1) {
    currentSceneIndex.value++
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

/* Global Transitions */
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>
