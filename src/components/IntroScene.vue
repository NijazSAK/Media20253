<template>
  <div class="intro-scene w-full h-full flex items-center justify-center bg-black relative overflow-hidden">
    <img 
      :src="src" 
      class="w-full h-full object-cover opacity-80"
      alt="Intro Scene"
    />
    <div class="absolute inset-0 bg-black/40"></div>
    <div class="absolute inset-0 flex items-center justify-center">
      <h1 class="text-4xl md:text-6xl font-bold text-white tracking-widest uppercase drop-shadow-2xl text-center px-4 animate-pulse">
        {{ caption }}
      </h1>
    </div>
  </div>
</template>

<script setup>
import { onMounted, onUnmounted } from 'vue'

const props = defineProps({
  src: String,
  caption: String,
  isActive: Boolean
})

const emit = defineEmits(['complete'])

let completionTimeout = null

const startScene = () => {
  // Auto-complete after 3 seconds, or we could wait for user input.
  // Given "user cannot avoid it", maybe a slightly longer delay or just let the user scroll/navigate manually if the app handles that.
  // The App.vue handles navigation with ArrowDown.
  // We should just emit complete so the "Press Down" hint appears.
  completionTimeout = setTimeout(() => {
    emit('complete')
  }, 3000)
}

const stopScene = () => {
  if (completionTimeout) clearTimeout(completionTimeout)
}

// Watch for activation
import { watch } from 'vue'
watch(() => props.isActive, (active) => {
  if (active) startScene()
  else stopScene()
}, { immediate: true })

onUnmounted(stopScene)
</script>

<style scoped>
/* Add any specific styles if needed */
</style>
