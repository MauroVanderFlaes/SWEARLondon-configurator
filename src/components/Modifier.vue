<script setup>
import { ref } from 'vue';

const colors = [
    { color: 'white', hex: '#ffffff' },
    { color: 'black', hex: '#000000' },
    { color: 'red', hex: '#ff0000' },
    { color: 'blue', hex: '#0000ff' },
    { color: 'gold', hex: '#ffd700' },
    { color: 'navy', hex: '#000080' },
    { color: 'teal', hex: '#008080' },
    { color: 'lime', hex: '#00ff00' },
    // Add other colors as needed
];

const parts = ["inside", "outside_1", "outside_2", "outside_3", "laces", "sole_bottom", "sole_top"];

const selectedColor = ref(colors[0]); // Initialize with the first color
const currentPartIndex = ref(0); // Initialize with the first part index

const changeColor = (color) => {
    selectedColor.value = { ...color, part: parts[currentPartIndex.value] };
    console.log(selectedColor.value);
    // Emit an event or call a method to notify the configurator component about the color change
};

const navigatePrev = () => {
    currentPartIndex.value = (currentPartIndex.value - 1 + parts.length) % parts.length;
};

const navigateNext = () => {
    currentPartIndex.value = (currentPartIndex.value + 1) % parts.length;
};
</script>

<template>
    <div class="fixed bottom-0 left-0 right-0 bg-stone-200 py-7">
        <!-- Previous and Next buttons -->
        <div class="flex justify-center">
            <a href="#" @click="navigatePrev">prev</a>
            <h1 class="text-center text-[2em] font-bold pb-7">{{ parts[currentPartIndex] }}</h1>
            <a href="#" @click="navigateNext">next</a>
        </div>

        <!-- Color options -->
        <div class="flex justify-center gap-8 items-center flex-wrap">
            <div v-for="color in colors" :key="color.color" class="text-center">
                <div :style="{ backgroundColor: color.hex }" @click="changeColor(color)"
                    class="w-[42px] h-[42px] rounded-full mx-auto cursor-pointer"></div>
                <h2 class="mt-2">{{ color.color }}</h2>
            </div>
        </div>
    </div>
</template>