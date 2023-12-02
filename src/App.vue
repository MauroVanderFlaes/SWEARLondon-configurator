<script setup>
import Configurator from './components/Configurator.vue';
import Modifier from './components/Modifier.vue';

import { ref } from 'vue';

let newColor = ref({name: 'white', hex: '#ffffff', part: 'inside'});

const updateColor = (color) => {
  newColor.value = color;
};
</script>

<template>
  <Configurator :color="newColor" />
  <Modifier @update:colorChange="updateColor" />
</template>

<style scoped></style>
