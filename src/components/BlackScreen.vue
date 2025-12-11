<template>
  <div class="black-screen w-full h-full flex items-center justify-center bg-black">
    <h1 class="text-3xl md:text-5xl font-light text-white/80 tracking-widest italic animate-pulse text-center px-4">
      {{ caption }}
    </h1>
  </div>
</template>

<script setup>
import { onMounted, onUnmounted, watch, ref } from 'vue'

const props = defineProps({
  caption: String,
  duration: {
    type: Number,
    default: 5000
  },
  audioSrc: String,
  isActive: Boolean
})

const emit = defineEmits(['complete'])

let timer = null
const audio = ref(null)

const startSequence = () => {
  if (timer) clearTimeout(timer)
  
  if (props.audioSrc) {
    audio.value = new Audio(props.audioSrc)
    audio.value.play().catch(e => console.error("Audio play failed", e))
  }

  timer = setTimeout(() => {
    emit('complete')
  }, props.duration)
}

const stopSequence = () => {
  if (timer) clearTimeout(timer)
  if (audio.value) {
    audio.value.pause()
    audio.value = null
  }
}

watch(() => props.isActive, (active) => {
  if (active) startSequence()
  else stopSequence()
}, { immediate: true })

onUnmounted(() => {
  stopSequence()
})
</script>
