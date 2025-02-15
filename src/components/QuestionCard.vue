<script setup>
import QuestionCorrectionState from './QuestionCorrectionState.vue';
import QuestionOptions from './QuestionOptions.vue';

const props = defineProps({
  question: Object,
  selectedAnswer: String,
  correctAnswer: String
});

const emit = defineEmits(['answerSelected']);
</script>

<template>
  <div class="w-full sm:mt-0 mt-8">
    <h3 class="text-lg font-medium sm:mb-3 mb-6">{{ question.text }}</h3>
    <div v-for="option in question.options" :key="option" class="sm:mb-2 mb-4">
      <QuestionOptions 
        :option="option" 
        :selectedAnswer="selectedAnswer"
        @answerSelected="emit('answerSelected', $event)" 
      />
    </div>
    <p v-if="selectedAnswer && selectedAnswer !== correctAnswer" class="mt-2 text-red-600">
      <QuestionCorrectionState :correctAnswer="correctAnswer"/>
    </p>
  </div>
</template>