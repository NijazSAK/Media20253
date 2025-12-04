<template>
  <div class="language-game w-full h-full flex flex-col items-center justify-center bg-amber-50 p-8">
    <h2 class="text-2xl font-bold mb-12 text-amber-800 animate-fade-in">Form the Sentence</h2>

    <!-- Target Area -->
    <div class="flex flex-wrap gap-2 mb-12 min-h-[60px] p-4 bg-white rounded-xl shadow-inner w-full max-w-md justify-center items-center border-2 border-amber-100">
      <div 
        v-for="(word, index) in formedSentence" 
        :key="`formed-${index}`"
        @click="removeWord(index)"
        class="px-4 py-2 bg-amber-500 text-white rounded-lg cursor-pointer hover:bg-red-500 transition-colors shadow-sm animate-pop"
      >
        {{ word }}
      </div>
      <span v-if="formedSentence.length === 0" class="text-gray-400 italic">Tap words below...</span>
    </div>

    <!-- Word Bank -->
    <div class="flex flex-wrap gap-4 justify-center max-w-md">
      <button 
        v-for="(word, index) in availableWords" 
        :key="`bank-${index}`"
        @click="addWord(word, index)"
        class="px-6 py-3 bg-white text-amber-900 border border-amber-200 rounded-lg shadow-md hover:-translate-y-1 hover:shadow-lg transition-all active:scale-95 font-medium"
      >
        {{ word }}
      </button>
    </div>

    <!-- Feedback -->
    <div class="h-8 mt-8">
      <p v-if="isWrong" class="text-red-500 font-bold animate-shake">Try again!</p>
    </div>
  </div>
</template>

<script setup>
import { ref, watch } from 'vue'

const emit = defineEmits(['complete'])

const targetSentence = ["I", "miss", "you", "so", "much"]
const shuffled = ["much", "I", "so", "miss", "you"].sort(() => Math.random() - 0.5)

const availableWords = ref([...shuffled])
const formedSentence = ref([])
const isWrong = ref(false)

const addWord = (word, index) => {
  formedSentence.value.push(word)
  availableWords.value.splice(index, 1)
  isWrong.value = false
}

const removeWord = (index) => {
  const word = formedSentence.value[index]
  formedSentence.value.splice(index, 1)
  availableWords.value.push(word)
  isWrong.value = false
}

watch(formedSentence, (newVal) => {
  if (newVal.length === targetSentence.length) {
    // Check correctness
    const isCorrect = newVal.every((word, i) => word === targetSentence[i])
    if (isCorrect) {
      setTimeout(() => emit('complete'), 500)
    } else {
      isWrong.value = true
    }
  }
}, { deep: true })
</script>

<style scoped>
.animate-pop {
  animation: pop 0.3s cubic-bezier(0.175, 0.885, 0.32, 1.275);
}
@keyframes pop {
  0% { transform: scale(0); opacity: 0; }
  100% { transform: scale(1); opacity: 1; }
}
.animate-shake {
  animation: shake 0.5s cubic-bezier(.36,.07,.19,.97) both;
}
@keyframes shake {
  10%, 90% { transform: translate3d(-1px, 0, 0); }
  20%, 80% { transform: translate3d(2px, 0, 0); }
  30%, 50%, 70% { transform: translate3d(-4px, 0, 0); }
  40%, 60% { transform: translate3d(4px, 0, 0); }
}
</style>
