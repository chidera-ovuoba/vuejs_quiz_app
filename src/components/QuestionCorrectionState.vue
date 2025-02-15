<script setup>
const props = defineProps({
  correctAnswer: String
});
</script>
<template>
    <div class="px-4 py-3 bg-red-100 border-red-500 border-1 rounded-3xl">
    <h5>Wrong !</h5>
    <span>{{ correctAnswer }}</span>
    </div>
</template>   