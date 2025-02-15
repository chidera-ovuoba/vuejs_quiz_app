   <script setup>
  import { computed } from 'vue';
  const props = defineProps({ current: Number, total: Number });
  const progress = computed(() => (props.current / props.total) * 100);
  </script>
  
<template>
    <div class="w-full bg-gray-200 rounded-full h-2.5 mb-4">
      <div class="bg-purple-600 h-2.5 rounded-full transition-all duration-500 ease-in-out" :style="{ width: progress + '%' }"></div>
    </div>
  </template>
  