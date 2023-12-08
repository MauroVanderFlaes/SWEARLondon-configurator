<script setup>
import { ref, onMounted } from 'vue';

import * as THREE from "three";

//import GLFT loader
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader.js";

// Import DRACOLoader
import { DRACOLoader } from "three/examples/jsm/loaders/DRACOLoader.js";

//import orbit controls 
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js';

const props = defineProps(['color']);

const draco = new DRACOLoader();
draco.setDecoderConfig({ type: 'js' });
draco.setDecoderPath('https://www.gstatic.com/draco/versioned/decoders/1.5.6/');

const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(
  75,
  window.innerWidth / window.innerHeight,
  0.1,
  1000
);

const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.setSize(window.innerWidth, window.innerHeight);
document.body.appendChild(renderer.domElement);

//load cubeTextureLoader from /CubeMap
const cubeTextureLoader = new THREE.CubeTextureLoader();
const environmentMapTexture = cubeTextureLoader.load([
  "/CubeMap/px.png",
  "/CubeMap/nx.png",
  "/CubeMap/py.png",
  "/CubeMap/ny.png",
  "/CubeMap/pz.png",
  "/CubeMap/nz.png",
]);

//add environment map
scene.background = environmentMapTexture;
scene.environment = environmentMapTexture;

//add orbit controls
const controls = new OrbitControls(camera, renderer.domElement);

//add GLTF model
const gltfLoader = new GLTFLoader();
gltfLoader.setDRACOLoader(draco); // Attach DRACOLoader to GLTFLoader

onMounted(() => {
  // Initialize the renderer and add it to the DOM
  renderer.setSize(window.innerWidth, window.innerHeight);
  document.querySelector("#app").appendChild(renderer.domElement);

  // Set up the GLTF model
  gltfLoader.load("/models/shoe.glb", (gltf) => {
    gltf.scene.position.set(0, 0, 0);
    gltf.scene.rotation.y = 1;
    gltf.scene.rotation.x = 0;
    gltf.scene.rotation.z = 0;
    gltf.scene.scale.set(6, 6, 6);

    let shoe = gltf.scene.children[0];

    // Traverse the children of the loaded model
    shoe.traverse((child) => {
      if (child.isMesh) {
        console.log(child.name);
        if (child.name == "inside") {
          child.material.color.set("#ffffff");
        }
        if (child.name == "outside_1") {
          child.material.color.set("#00ff00");
        }
        if (child.name == "outside_2") {
          child.material.color.set("#00ff00");
        }
        if (child.name == "outside_3") {
          child.material.color.set("#00ff00");
        }
        if (child.name == "laces") {
          child.material.color.set("#000000");
        }
        if (child.name == "sole_bottom") {
          child.material.color.set("#ffffff");
        }
        if (child.name == "sole_top") {
          child.material.color.set("#ffffff");
        }
        // Set up casting shadows for meshes
        child.castShadow = true;
      }
    });
    // Add the loaded model to the scene
    scene.add(gltf.scene);
  });

  //table
  const geometry = new THREE.CylinderGeometry(2, 1, 5, 32);
  const material = new THREE.MeshStandardMaterial({
    roughness: 0,
    metalness: 0.5,
    color: 0xAA00,
    envMap: environmentMapTexture,
  });
  const octagon = new THREE.Mesh(geometry, material);
  //table position
  octagon.position.set(0, -3, 0);
  octagon.material.side = THREE.DoubleSide;
  scene.add(octagon);

  camera.position.z = 3;

  //add ambient
  const ambientLight = new THREE.AmbientLight(0xffffff, 2);
  ambientLight.position.set(0, 10, 0);
  scene.add(ambientLight);

  //add clock
  const clock = new THREE.Clock();

  function animate() {
    const elapsedTime = clock.getElapsedTime();
    requestAnimationFrame(animate);

    //rotate shoe
    scene.rotation.y = elapsedTime * 0.5;

    renderer.render(scene, camera);
  }

  animate();
});

const colors = [
  { color: 'white', hex: '#ffffff' },
  { color: 'black', hex: '#000000' },
  { color: 'red', hex: '#ff0000' },
  { color: 'blue', hex: '#0000ff' },
  { color: 'gold', hex: '#ffd700' },
  { color: 'navy', hex: '#000080' },
  { color: 'teal', hex: '#008080' },
  { color: 'lime', hex: '#00ff00' },
];

const parts = ["inside", "outside_1", "outside_2", "outside_3", "laces", "sole_bottom", "sole_top"];

const selectedColor = ref(colors[0]); // Initialize with the first color
const currentPartIndex = ref(0); // Initialize with the first part index

const navigatePrev = () => {
  currentPartIndex.value = (currentPartIndex.value - 1 + parts.length) % parts.length;
};

const navigateNext = () => {
  currentPartIndex.value = (currentPartIndex.value + 1) % parts.length;
};

const changeColor = (color, part) => {
  scene.traverse((child) => {
    console.log(child.name, color, part);
    if (child.isMesh && child.name === part) {
      child.material.color.setStyle(color);
    }
  });
};
</script>

<template>
  <div class="fixed bottom-0 left-0 right-0 bg-stone-200 py-7">
    <!-- Previous and Next buttons -->
    <div class="flex justify-center">
      <a href="#" @click="navigatePrev"><i class="fa-solid fa-arrow-left fa-xl relative top-3.5 right-5"></i></a>
      <h1 class="text-center text-[2em] font-bold pb-7">{{ parts[currentPartIndex] }}</h1>
      <a href="#" @click="navigateNext"><i class="fa-solid fa-arrow-right fa-xl relative top-3.5 left-5"></i></a>
    </div>

    <!-- Color options -->
    <div class="flex justify-center gap-8 items-center flex-wrap">
      <div v-for="color in colors" :key="color.color" class="text-center">
        <div :style="{ backgroundColor: color.hex }" @click="changeColor(color.hex, parts[currentPartIndex])"
          class="w-[42px] h-[42px] rounded-full mx-auto cursor-pointer"></div>
        <h2 class="mt-2">{{ color.color }}</h2>
      </div>
    </div>
  </div>
</template>

<style scoped></style>