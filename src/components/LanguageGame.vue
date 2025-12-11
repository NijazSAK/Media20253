<template>
  <div class="language-game w-full h-full flex flex-col items-center justify-between p-8 relative overflow-hidden">
    
    <!-- Custom Background -->
    <div v-if="backgroundImage" class="absolute inset-0 z-0">
      <img :src="backgroundImage" class="w-full h-full object-cover" />
      <div class="absolute inset-0 bg-black/20"></div>
    </div>

    <!-- Conversation Area -->
    <div class="w-full max-w-2xl flex flex-col gap-8 z-10 mt-32">
      
      <!-- Clerk Bubble (Left) -->
      <div class="self-start bg-white p-6 rounded-2xl rounded-tl-none shadow-md border-2 border-gray-100 max-w-xs animate-fade-in-left transition-all duration-100">
        <p class="text-lg font-medium text-gray-800">{{ currentStageData.clerkText }}</p>
      </div>

      <!-- Player Bubble (Right - Target) -->
      <div class="self-end flex flex-col items-end w-full">
        <div 
          class="bg-blue-500 p-6 rounded-2xl rounded-tr-none shadow-md min-w-[200px] min-h-[80px] flex flex-wrap gap-2 justify-end items-center transition-colors duration-100"
          :class="{ 'bg-red-500': isWrong, 'bg-green-500': isCorrect }"
        >
          <div 
            v-for="(word, index) in formedSentence" 
            :key="`formed-${index}`"
            @click="removeWord(index)"
            class="px-3 py-1 bg-white/20 text-white rounded cursor-pointer hover:bg-white/30 transition-colors duration-100"
          >
            {{ word }}
          </div>
          <span v-if="formedSentence.length === 0" class="text-white/50 italic text-sm">Tap words to reply...</span>
        </div>
        <p v-if="isWrong" class="text-red-500 font-bold mt-2 animate-shake">Incorrect grammar!</p>
      </div>

    </div>

    <!-- Word Bank (Bottom) -->
    <div class="w-full max-w-2xl z-10 mb-12">
      <div class="flex flex-wrap gap-3 justify-center">
        <button 
          v-for="(word, index) in availableWords" 
          :key="`bank-${index}`"
          @click="addWord(word, index)"
          class="px-6 py-3 bg-white text-slate-800 border-b-4 border-slate-200 rounded-lg shadow-sm hover:-translate-y-1 hover:border-b-6 active:translate-y-0 active:border-b-2 transition-all duration-100 font-bold text-lg"
        >
          {{ word }}
        </button>
      </div>
    </div>

  </div>
</template>

<script setup>
import { ref, computed, watch } from 'vue'

const props = defineProps({
  backgroundImage: String
})

const emit = defineEmits(['complete'])

const stage = ref(0)
const isWrong = ref(false)
const isCorrect = ref(false)

const stages = [
  {
    clerkText: "Hello! Welcome to StickBurger. What can I get for you today?",
    targetSentence: ["I", "would", "like", "a", "burger"],
    wordBank: ["burger", "I", "a", "would", "like", "want", "give"]
  },
  {
    clerkText: "Great choice! Would you like fries with that?",
    targetSentence: ["Yes", "please", "add", "fries"],
    wordBank: ["fries", "No", "Yes", "add", "please", "don't"]
  },
  {
    clerkText: "Anything to drink?",
    targetSentence: ["Just", "water", "for", "me"],
    wordBank: ["me", "water", "Just", "coke", "for", "please"]
  }
]

const currentStageData = computed(() => stages[stage.value])

// State
const availableWords = ref([...stages[0].wordBank].sort(() => Math.random() - 0.5))
const formedSentence = ref([])

const addWord = (word, index) => {
  if (isCorrect.value) return
  formedSentence.value.push(word)
  availableWords.value.splice(index, 1)
  isWrong.value = false
  checkSentence()
}

const removeWord = (index) => {
  if (isCorrect.value) return
  const word = formedSentence.value[index]
  formedSentence.value.splice(index, 1)
  availableWords.value.push(word)
  isWrong.value = false
}

const checkSentence = () => {
  const target = currentStageData.value.targetSentence
  
  // Only check if length matches to avoid premature errors, 
  // unless you want strict ordering check on every click.
  // Let's check only when length matches for simplicity.
  if (formedSentence.value.length === target.length) {
    const isMatch = formedSentence.value.every((w, i) => w === target[i])
    
    if (isMatch) {
      handleSuccess()
    } else {
      isWrong.value = true
    }
  }
}

const handleSuccess = () => {
  isCorrect.value = true
  setTimeout(() => {
    if (stage.value < stages.length - 1) {
      nextStage()
    } else {
      emit('complete')
    }
  }, 1000)
}

const nextStage = () => {
  stage.value++
  isCorrect.value = false
  formedSentence.value = []
  availableWords.value = [...currentStageData.value.wordBank].sort(() => Math.random() - 0.5)
}

</script>

<style scoped>
.animate-fade-in-left {
  animation: fadeInLeft 0.5s ease-out;
}
@keyframes fadeInLeft {
  from { opacity: 0; transform: translateX(-20px); }
  to { opacity: 1; transform: translateX(0); }
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
