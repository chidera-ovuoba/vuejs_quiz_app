<script setup>
import { ref, onMounted, computed} from 'vue';
import Timer from './Timer.vue';
import ProgressIndicator from './ProgressIndicator.vue';
import DragDropQuestion from './DragDropQuestion .vue';
import QuestionCard from './QuestionCard.vue';
import Button from './Button.vue';
import { ChevronLeftIcon } from '@heroicons/vue/24/solid';


const title = ref('Aptitude Test');
const currentIndex = ref(0);
const transitionEffect = ref('');
const timeLeft = ref(120);
const submitted = ref(false);
const answers = ref({multipleChoiceAnswers:[],draggableAnswers:{}});
const questions = ref([
  { id: 1, text: 'If a car travels 200 miles in 4 hours, what is its average speed?', options: ['40 Mph', '50 Mph', '60 Mph', '70 Mph'], answer: '40 Mph' },
  { id: 2, text: 'If it takes 5 hours for 5 men to dig 5 holes, how long does it take for 100 men to dig 100 holes?', options: ['1 Hour', '2 Hour', '5 Hour', '10 Hour'], answer: '1 Hour' },
  { id: 3, text: 'Match the following time durations correctly', 
  options: [{value: '24 Hours', pair:2}, {value: '7 days', pair:3}, {value: '4 weeks', pair:5}, {value: '12 months', pair:1}], 
  draggableOptions: [{value: '1 day', pair:2}, {value: '1 week', pair:3}, {value: '1 month', pair:5}, {value: '1 year', pair:1}],
  draggableAnswers:[{value:'24 Hours', match:'1 day'},{value:'7 days', match:'1 week'},{value:'4 weeks', match:'1 month'},{value:'12 months', match:'1 year'},]
 }
]);
let timerInterval = null;
let animationTimeout = null;

onMounted(() => {
  startTimer();
  return () => clearInterval(timerInterval);
});

const startTimer = () => {
  clearInterval(timerInterval);
  timerInterval = setInterval(() => {
        if (timeLeft.value > 0) {
            timeLeft.value--;
        } else {
          submitQuiz();
          clearInterval(timerInterval);
        }
  }, 1000);
};

const handleAnswerSelection = ({selected,isDroppedOptions,questionId}) => {
    if(isDroppedOptions){
        answers.value.draggableAnswers[questionId] = selected;
    }
    answers.value.multipleChoiceAnswers[currentIndex.value] = selected;
};

const nextQuestion = () => {
  if (animationTimeout) clearTimeout(animationTimeout);
  transitionEffect.value = '';
  animationTimeout = setTimeout(() => {
    transitionEffect.value = 'fade-slide-next';
    if (currentIndex.value < questions.value.length - 1) {
      currentIndex.value++;
    }
  }, 10);
};

const prevQuestion = () => {
  if (animationTimeout) clearTimeout(animationTimeout);
  transitionEffect.value = '';
  animationTimeout = setTimeout(() => {
    transitionEffect.value = 'fade-slide-prev';
    if (currentIndex.value > 0) {
      currentIndex.value--;
    }
  }, 10);
};


const submitQuiz = () => {
  submitted.value = true;
};

const resetQuiz = () => {
  clearInterval(timerInterval);
  submitted.value = false;
  currentIndex.value = 0;
  answers.value = {multipleChoiceAnswers:[],draggableAnswers:{}};
  timeLeft.value = 120;
  startTimer();
};

const incorrectOrMissingAnswers = computed(() => {
  return questions.value
    .filter((q) => q.draggableOptions) 
    .map((question) => {
      const userAnswers = answers.value.draggableAnswers[question.id] || [];
      
      return {
        questionId: question.id,
        incorrectPairs: question.draggableAnswers.filter((pair) => {
          const foundAnswer = userAnswers.find((answer) => answer.value === pair.match);
          return !foundAnswer || !foundAnswer.isCorrect;
        }),
      };
    })
    .filter((q) => q.incorrectPairs.length > 0);
});

</script>

<template>
<div class="flex sm:items-center items-start justify-start sm:justify-center min-h-screen max-h-[90vh]">
  <button 
      v-if="currentIndex > 0 && !submitted"
      @click="prevQuestion" 
      class="absolute left-4 top-4 p-2 rounded-full bg-purple-100 shadow-md sm:hidden"
    >
      <ChevronLeftIcon class="w-6 h-6 text-purple-700" />
    </button>
    <div v-if="!submitted" class="w-screen sm:w-124 bg-none sm:bg-white p-6 sm:rounded-3xl rounded-none shadow-none sm:shadow-xl">
      <div class="flex justify-between items-center sm:my-4 my-8 ">
        <h2 class="text-lg font-semibold">{{ title }}</h2>
        <Timer :timeLeft="timeLeft" />
      </div>
      <ProgressIndicator :current="currentIndex" :total="questions.length" />
      <h6 class="text-purple-800 font-semibold">Question {{ currentIndex + 1}} of {{questions.length}}</h6>
      <QuestionCard 
          v-if="!questions[currentIndex].draggableOptions"
          :question="questions[currentIndex]" 
          :selectedAnswer="answers.multipleChoiceAnswers[currentIndex]"
          :correctAnswer="questions[currentIndex].answer"
          @answerSelected="(val) => answers.multipleChoiceAnswers[currentIndex] = val"
          :class="transitionEffect"
        />
        <DragDropQuestion
        v-else 
          :question="questions[currentIndex]" 
          @answerSelected="handleAnswerSelection"
          :class="transitionEffect" 
        />

      <div class="flex gap-2 mt-4">
        <Button v-if="currentIndex > 0" text="← Back" @click="prevQuestion" class="w-full sm:block hidden" />
        <Button v-if="currentIndex < questions.length - 1" text="Next →" @click="nextQuestion" class="w-full sm:mt-0 mt-[5vh]" />
        <Button v-else text="Submit" @click="submitQuiz" class="w-full" />
      </div>
    </div>

    <div v-else class="w-full max-w-lg sm:w-96 bg-white p-6 sm:mx-none mx-6 self-center rounded-2xl shadow-xl max-h-[90vh]">
      <h2 class="text-lg font-semibold mb-4">Quiz Results</h2>
      <ul class="max-h-[60vh] overflow-y-scroll">
        <li v-for="(question, index) in questions" :key="index" class="mb-3">
          <p class="font-medium">{{ question.text }}</p>
          <p v-if="!question.draggableOptions" :class="{'text-green-600': answers.multipleChoiceAnswers[index] === question.answer, 'text-red-600': answers.multipleChoiceAnswers[index] !== question.answer}">
              Your answer: 
              <span>{{ answers.multipleChoiceAnswers[index] || 'No answer' }}</span>
            </p>
            <p v-else :class="{'text-red-600': !answers.draggableAnswers[question.id]}">
              Your answer: 
              <div v-if="answers.draggableAnswers[question.id]" v-for="(pair, i) in answers.draggableAnswers[question.id]" :key="i" class="grid grid-cols-3 gap-4 mb-2">
                <span class="font-semibold place-self-start" :class="{'text-green-600':pair.isCorrect, 'text-red-600': !pair.isCorrect}">{{ pair.match }}</span>
                <span class="text-gray-500 place-self-center">→</span>
                <span class="font-semibold place-self-end" :class="{'text-green-600':pair.isCorrect, 'text-red-600': !pair.isCorrect}">{{ pair.value }}</span>
              </div>
              <span v-else>No answer</span>
          </p>
          <p v-if="answers.multipleChoiceAnswers[index] !== question.answer && !question.draggableOptions" class="text-gray-600">Correct answer: {{ question.answer }}</p>
          <p v-if="question.draggableOptions" class="text-gray-600">Correct answer: 
            <div  v-for="(pair, i) in incorrectOrMissingAnswers.find(q => q.questionId === question.id).incorrectPairs || []"
            :key="i" class="grid grid-cols-3 gap-4 mb-2">
              <span class="font-semibold place-self-start">{{ pair.value }}</span>
              <span class="text-gray-500 place-self-center">→</span>
              <span class="font-semibold place-self-end">{{ pair.match }}</span>
            </div>
          </p>

        </li>
      </ul>
      <Button text="Restart Quiz" @click="resetQuiz" class="w-full mt-4" />
    </div>
  </div>
  </template>

<style scoped>
.fade-slide-next {
  animation: fade-slide-next-enter-from  0.4s ease-in-out;
}
.fade-slide-prev{
  animation: fade-slide-prev-enter-from 0.4s ease-in-out;
}
@keyframes fade-slide-next-enter-from  {
  0%,50%{
    opacity: 0;
    transform: translateX(20px);
  }
  100%{
    opacity: 1;
    transform: translateX(0px);
  }
}
@keyframes fade-slide-prev-enter-from  {
  0%,50%{
    opacity: 0;
    transform: translateX(20px);
  }
  100%{
    opacity: 1;
    transform: translateX(0px);
  }
}

</style>
   