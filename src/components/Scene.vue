<template>
  <div class="scene-image w-full h-full flex items-center justify-center bg-gray-900 relative overflow-hidden">
    <img 
      :src="src" 
      class="w-full h-full object-cover transition-transform duration-[2000ms] ease-in-out hover:scale-105"
      alt="Scene"
    />
    <div class="absolute inset-0 bg-black/20"></div>
    <div class="absolute bottom-12 left-8 max-w-md text-white">
      <p class="text-lg font-light tracking-wide drop-shadow-md">{{ caption }}</p>
    </div>
  </div>
</template>

<script setup>
defineProps({
  src: String,
  caption: String
})
</script>
