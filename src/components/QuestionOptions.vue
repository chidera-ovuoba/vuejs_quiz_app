<script setup>
const props = defineProps({
  option: String,
  selectedAnswer: String
});

const emit = defineEmits(['answerSelected']);
</script>

<template>
  <label class="flex items-center gap-2 cursor-pointer w-full justify-between px-3 py-1 hover:bg-purple-100 hover:border-purple-500 rounded-3xl" :class="{'bg-purple-100 border-purple-500 border-1': selectedAnswer === option}">
    <input
      type="radio"
      :value="option"
      :checked="selectedAnswer === option"
      class="hidden"
      @change="emit('answerSelected', option)"
    />
    <span class="text-gray-700">{{ option }}</span>
    <div class="w-5 h-5 border border-purple-400 rounded-full flex items-center justify-center"
      :class="{ 'bg-purple-600 border-purple-600': selectedAnswer === option }">
      <div v-if="selectedAnswer === option" class="w-2.5 h-2.5 bg-white rounded-full"></div>
    </div>
  </label>
</template>