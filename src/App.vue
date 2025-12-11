<template>
  <div class="app-container w-full h-screen overflow-hidden bg-slate-900 text-slate-200 font-sans">
    <!-- Main Scroll Container -->
    <div 
      ref="container"
      class="w-full h-full relative transition-transform duration-700 ease-in-out"
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
          class="absolute top-4 right-4 z-50 px-4 py-2 bg-slate-800/80 text-white rounded-lg hover:bg-slate-700 transition-colors border border-slate-600 font-mono text-sm backdrop-blur-sm"
        >
          SKIP [S]
        </button>
        
        <!-- Navigation Hint -->
        <div 
          v-if="scene.completed && index < scenes.length - 1"
          class="absolute bottom-8 left-1/2 transform -translate-x-1/2 animate-bounce flex flex-col items-center gap-2 opacity-80"
        >
          <span class="text-white text-sm font-mono tracking-widest uppercase bg-black/50 px-3 py-1 rounded-full backdrop-blur-sm">Press ↓</span>
          <svg class="w-6 h-6 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 14l-7 7m0 0l-7-7m7 7V3"></path></svg>
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
import EndScene from './components/EndScene.vue'

import IntroScene from './components/IntroScene.vue'
import introImage from './assets/intro-scene.jpg'

// Scene Configuration
const scenes = ref([
  {
    component: shallowRef(IntroScene),
    props: {
      src: introImage,
      caption: 'Again... I lie...'
    },
    isGame: false,
    completed: false
  },
  { 
    component: shallowRef(Scene), 
    // Quick shots of city life
    props: { 
      src: [
        'https://picsum.photos/id/43/1920/1080', // City
        'https://picsum.photos/id/48/1920/1080', // Architecture
        'https://picsum.photos/id/60/1920/1080', // Office
        'https://picsum.photos/id/180/1920/1080' // Laptop
      ], 
      caption: 'It started like any other day... fast paced.',
      animationSpeed: 400 
    },
    isGame: false,
    completed: false
  },
  { 
    component: shallowRef(Scene), 
    props: { src: 'https://picsum.photos/id/11/1920/1080', caption: 'But something felt off balance.' },
    isGame: false,
    completed: false
  },
  { 
    component: shallowRef(BalanceGame), 
    props: {}, 
    isGame: true,
    completed: false 
  },
  { 
    component: shallowRef(Scene), 
    // Disoriented shots
    props: { 
      src: [
        'https://picsum.photos/id/12/1920/1080',
        'https://picsum.photos/id/16/1920/1080',
        'https://picsum.photos/id/20/1920/1080'
      ],
      caption: 'Words couldn\'t express the feeling.',
      animationSpeed: 800
    },
    isGame: false,
    completed: false
  },
  { 
    component: shallowRef(LanguageGame), 
    props: {}, 
    isGame: true,
    completed: false 
  },
  { 
    component: shallowRef(Scene), 
    // Darker, slower shots
    props: { 
      src: [
        'https://picsum.photos/id/13/1920/1080', // Beach/Calm?
        'https://picsum.photos/id/19/1920/1080', // Darker
        'https://picsum.photos/id/26/1920/1080'  // Objects
      ],
      caption: 'Exhaustion took over.',
      animationSpeed: 1500
    },
    isGame: false,
    completed: false
  },
  { 
    component: shallowRef(Scene), 
    props: { src: 'https://picsum.photos/id/14/1920/1080', caption: 'Trying to stay awake...' },
    isGame: false,
    completed: false
  },
  { 
    component: shallowRef(WakeUpGame), 
    props: {}, 
    isGame: true,
    completed: false 
  },
  { 
    component: shallowRef(Scene), 
    props: { src: 'https://picsum.photos/id/64/1920/1080', caption: '"You weren\'t cut out for this," he scoffed, walking past me.' },
    isGame: false,
    completed: false
  },
  { 
    component: shallowRef(Scene), 
    props: { src: 'https://picsum.photos/id/55/1920/1080', caption: '"WAIT! HOW DID YOU DO THAT!?" I screamed, desperate.' },
    isGame: false,
    completed: false
  },
  { 
    component: shallowRef(Scene), 
    props: { src: 'https://picsum.photos/id/225/1920/1080', caption: 'He turned, holding up a package. "10-minute ready meal."' },
    isGame: false,
    completed: false
  },
  { 
    component: shallowRef(EndScene), 
    props: { 
      // Flashback Sequence (Ad Montage)
      src: [
        'https://picsum.photos/id/43/1920/1080',  // City
        'https://picsum.photos/id/11/1920/1080',  // Balance
        'https://picsum.photos/id/12/1920/1080',  // Confusion
        'https://picsum.photos/id/13/1920/1080',  // Exhaustion
        'https://picsum.photos/id/14/1920/1080',  // Struggle
        'https://picsum.photos/id/15/1920/1080',  // Clarity
        'https://picsum.photos/id/292/1920/1080', // Food/Meal
        'https://picsum.photos/id/88/1920/1080'   // Slogan/Logo Background
      ],
      // Place your audio file in public/music/ending.mp3
      audioSrc: '/music/ending.mp3', 
      script: [
        "The city rush, a blur of grey,",
        "Stumbling through the alleyway.",
        "Words get lost, the mind goes weak,",
        "Energy is the thing you seek.",
        "Why fight the tired, heavy eyes?",
        "When the secret to winning lies...",
        "...in a flavor that's quick and real.",
        "StickBurger. The 10-Minute Ready Meal."
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
