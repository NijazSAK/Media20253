<template>
  <div class="black-screen w-full h-full flex items-center justify-center bg-black">
    <h1 class="text-3xl md:text-5xl font-light text-white/80 tracking-widest italic animate-pulse text-center px-4">
      {{ caption }}
    </h1>
  </div>
</template>

<script setup>
import { onMounted, onUnmounted, watch } from 'vue'

const props = defineProps({
  caption: String,
  duration: {
    type: Number,
    default: 5000
  },
  isActive: Boolean
})

const emit = defineEmits(['complete'])

let timer = null

const startTimer = () => {
  if (timer) clearTimeout(timer)
  timer = setTimeout(() => {
    emit('complete')
  }, props.duration)
}

watch(() => props.isActive, (active) => {
  if (active) startTimer()
  else if (timer) clearTimeout(timer)
}, { immediate: true })

onUnmounted(() => {
  if (timer) clearTimeout(timer)
})
</script>
