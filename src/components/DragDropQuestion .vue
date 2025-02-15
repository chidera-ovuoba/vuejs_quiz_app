<script setup>
import { ref, computed } from 'vue';

const props = defineProps({
  question: Object,
});
const emits = defineEmits(['answerSelected']);

const droppedOptions = ref(Array(props.question.options.length).fill(null));
const draggedItem = ref(null);

const remainingOptions = computed(() => {
  return props.question.draggableOptions.filter(option => !droppedOptions.value.includes(option));
});

const dragStart = (option, index) => {
  draggedItem.value = { option, index };
};

const drop = (targetOption, index) => {
  if (draggedItem.value) {
    droppedOptions.value[index] = draggedItem.value.option;
    draggedItem.value = null;
    checkAnswers();
  }
};

const removeDropped = (index) => {
  droppedOptions.value[index] = null;
};

const isCorrect = (index) => {
  return droppedOptions.value[index] && droppedOptions.value[index].pair === props.question.options[index].pair;
};

const isWrong = (index) => {
  return droppedOptions.value[index] && droppedOptions.value[index].pair !== props.question.options[index].pair;
};

const checkAnswers = () => {
  const selected = droppedOptions.value.map((opt, index) =>{
      if(opt){
         opt.isCorrect = opt.pair === props.question.options[index].pair
         opt.match = props.question.options[index].value
         return opt 
    }
});

const filteredselected = selected.filter(item => item !== undefined)
  emits('answerSelected', {selected:filteredselected,isDroppedOptions:true,questionId:props.question.id});
};
</script>

<template>
    <div class="p-4">
      <h3 class="text-lg font-semibold mb-4">{{ question.text }}</h3>
      <div class="grid grid-cols-2 gap-4">
        <div 
          v-for="(option, index) in question.options" 
          :key="index" 
          class="border-dashed border-purple-600 border-2 p-4 rounded-lg shadow-md bg-purple-200 min-h-[50px]"
          @dragover.prevent 
          @drop="drop(option, index)"
        >
          <p class="text-gray-600 mb-2 text-center">{{ option.value }}</p>
          <div
            v-if="droppedOptions[index]"
            :class="{'bg-green-200': isCorrect(index), 'bg-red-200': isWrong(index)}"
            class="p-2 rounded-md text-center cursor-pointer"
            @click="removeDropped(index)"
          >
            {{ droppedOptions[index].value }}
          </div>
        </div>
        <div class="col-span-2 border border-purple-600 p-4 rounded-lg shadow-md bg-white min-h-[100px]">
          <p class="text-gray-600 mb-2">Drag from here:</p>
          <div class="grid grid-cols-2 gap-4">
          <div
            v-for="(option, index) in remainingOptions"
            :key="index"
            draggable="true"
            @dragstart="dragStart(option, index)"
            class="p-2 bg-purple-500 text-white rounded-md shadow-sm text-center cursor-pointer"
          >
            {{ option.value }}
          </div>
          </div>
        </div>
      </div>
    </div>
  </template>
  
  <style scoped>
  div[draggable="true"] {
    transition: transform 0.2s;
  }
  
  div[draggable="true"]:active {
    transform: scale(1.1);
  }
  </style>