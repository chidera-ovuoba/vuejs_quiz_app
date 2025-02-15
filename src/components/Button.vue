  <script setup>
  defineProps({ text: String });
  </script>
<template>
    <button class="bg-black text-white px-4 py-2 rounded-lg hover:bg-gray-800 transition">{{ text }}</button>
  </template>
  