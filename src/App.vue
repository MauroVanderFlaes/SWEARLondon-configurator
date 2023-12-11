<script setup>
import { ref, onMounted, reactive } from 'vue';

import * as THREE from "three";

//import GLFT loader
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader.js";

// Import DRACOLoader
import { DRACOLoader } from "three/examples/jsm/loaders/DRACOLoader.js";

//import orbit controls 
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js';

const loading = ref(true); // Initialize loading state as true

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

const textureLoader = new THREE.TextureLoader();

//leather texture
const colorMap = textureLoader.load('/textures/Leather/FabricLeatherCowhide001_COL_VAR1_2K.jpg');
const normalMap = textureLoader.load('/textures/Leather/FabricLeatherCowhide001_NRM_2K.jpg');
const bumpMap = textureLoader.load('/textures/Leather/FabricLeatherCowhide001_BUMP_2K.jpg');
const glossMap = textureLoader.load('/textures/Leather/FabricLeatherCowhide001_GLOSS_2K.jpg');

//rubber texture
const rubberNormalMap = textureLoader.load('/textures/Rubber/Rubber_Sole_002_normal.jpg');
const rubberBumpMap = textureLoader.load('/textures/Rubber/Rubber_Sole_002_height.png');
const rubberGlossMap = textureLoader.load('/textures/Rubber/Rubber_Sole_002_roughness.jpg');
const rubberColorMap = textureLoader.load('/textures/Rubber/Rubber_Sole_002_basecolor.jpg');
const rubberAmbientOcclusionMap = textureLoader.load('/textures/Rubber/Rubber_Sole_002_ambientOcclusion.jpg');

//Organic texture
const organicNormalMap = textureLoader.load('/textures/Organic/Abstract_Organic_006_normal.jpg');
const organicBumpMap = textureLoader.load('/textures/Organic/Abstract_Organic_006_height.png');
const organicGlossMap = textureLoader.load('/textures/Organic/Abstract_Organic_006_roughness.jpg');
const organicColorMap = textureLoader.load('/textures/Organic/Abstract_Organic_006_basecolor.jpg');
const organicAmbientOcclusionMap = textureLoader.load('/textures/Organic/Abstract_Organic_006_ambientOcclusion.jpg');

//Lava texture
const lavaNormalMap = textureLoader.load('/textures/Lava/Lava_005_NORM.jpg');
const lavaBumpMap = textureLoader.load('/textures/Lava/Lava_005_DISP.png');
const lavaGlossMap = textureLoader.load('/textures/Lava/Lava_005_ROUGH.jpg');
const lavaColorMap = textureLoader.load('/textures/Lava/Lava_005_COLOR.jpg');
const lavaAmbientOcclusionMap = textureLoader.load('/textures/Lava/Lava_005_OCC.jpg');

//add environment map
scene.background = environmentMapTexture;
scene.environment = environmentMapTexture;

//add orbit controls
const controls = new OrbitControls(camera, renderer.domElement);

controls.maxDistance = 10;
controls.minDistance = 1;
controls.maxPolarAngle = Math.PI / 2 - 0.1;

//add GLTF model
const gltfLoader = new GLTFLoader();
gltfLoader.setDRACOLoader(draco); // Attach DRACOLoader to GLTFLoader

const textures = [
  { name: 'leather', displayName: 'Leather', preview: '/textures/Leather/leather_preview.jpg' },
  { name: 'rubber', displayName: 'Rubber', preview: '/textures/Rubber/rubber_preview.jpg' },
  { name: 'organic', displayName: 'Organic', preview: '/textures/Organic/organic_preview.jpg' },
  { name: 'lava', displayName: 'Lava', preview: '/textures/Lava/lava_preview.jpg' },
  // Add more textures as needed
];

const state = reactive({
  selectedTexture: 'leather', // Standaard geselecteerde textuur
});


const changeTexture = (texture, part) => {
  const texturePath = getTexturePath(texture);


  scene.traverse((child) => {
  if (child.isMesh && child.name === part) {
    const textureMap = new THREE.TextureLoader().load(texturePath, (loadedTexture) => {
      updateMaterialTextures(child.material, loadedTexture, part);
    }); // Add the closing parenthesis and semicolon here
  }
});

  state.selectedTexture = texture;
};

const getTexturePath = (textureName) => {
  switch (textureName) {
    case 'leather':
      return '/textures/Leather/FabricLeatherCowhide001_COL_VAR1_2K.jpg';
    case 'rubber':
      return '/textures/Rubber/Rubber_Sole_002_basecolor.jpg';
    case 'organic':
      return '/textures/Organic/Abstract_Organic_006_basecolor.jpg';
    case 'lava':
      return '/textures/Lava/Lava_005_COLOR.jpg';
    // Voeg meer cases toe indien nodig
    default:
      return '';
  }
};

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

        let partMaterial; // Declare a variable to hold the material for each part

        if (child.name == "inside") {
          partMaterial = new THREE.MeshStandardMaterial({
            color: "#ffffff",
            map: colorMap,
            normalMap: normalMap,
            bumpMap: bumpMap,
            roughnessMap: glossMap
          });
        }
        if (child.name == "outside_1" || child.name == "outside_2" || child.name == "outside_3") {
          partMaterial = new THREE.MeshStandardMaterial({
            color: "#00ff00",
            map: colorMap,
            normalMap: normalMap,
            bumpMap: bumpMap,
            roughnessMap: glossMap
          });
        }
        if (child.name == "laces") {
          partMaterial = new THREE.MeshStandardMaterial({
            color: "#000000",
            map: colorMap,
            normalMap: normalMap,
            bumpMap: bumpMap,
            roughnessMap: glossMap
          });
        }
        if (child.name == "sole_bottom" || child.name == "sole_top") {



          partMaterial = new THREE.MeshStandardMaterial({
            color: "#ffffff",

            //Leather texture
            map: colorMap,
            normalMap: normalMap,
            bumpMap: bumpMap,
            roughnessMap: glossMap,

          });
        }

        // Set up casting shadows for meshes
        child.castShadow = true;

        // Apply the specific material for each part
        child.material = partMaterial || shoeMaterial; // Use shoeMaterial as a fallback
      }
    });


    // Add the loaded model to the scene
    scene.add(gltf.scene);

    // Set loading to false when the model is loaded
    loading.value = false;
  });

  //table
  const geometry = new THREE.CylinderGeometry(2, 1, 5, 128);
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

  camera.position.z = 2.5;
  camera.position.y = 0.5;

  //add ambient
  const ambientLight = new THREE.AmbientLight(0xffffff, 10);
  ambientLight.position.set(0, 10, 0);
  scene.add(ambientLight);

  //add clock
  const clock = new THREE.Clock();

  function animate() {
    const elapsedTime = clock.getElapsedTime();
    requestAnimationFrame(animate);

    //rotate shoe
    scene.rotation.y = elapsedTime * 0.5;

    controls.update();

    renderer.render(scene, camera);
  }

  animate();
});

const colors = [
  { color: 'White', hex: '#ffffff' },
  { color: 'Black', hex: '#000000' },
  { color: 'Red', hex: '#ff0000' },
  { color: 'Blue', hex: '#0000ff' },
  { color: 'Gold', hex: '#ffd700' },
  { color: 'Navy', hex: '#000080' },
  { color: 'Teal', hex: '#008080' },
  { color: 'Lime', hex: '#00ff00' },
];

const parts = [
  { name: "inside", display: "Inside Lining" },
  { name: "outside_1", display: "Frontstrays" },
  { name: "outside_2", display: "Midstrays" },
  { name: "outside_3", display: "Backstrays" },
  { name: "laces", display: "Laces" },
  { name: "sole_bottom", display: "Outsole" },
  { name: "sole_top", display: "Midsole" }
];

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



const updateMaterialTextures = (material, textureMap, part) => {
  switch (part) {
    case 'inside':
    case 'outside_1':
    case 'outside_2':
    case 'outside_3':
    case 'laces':
    case 'sole_bottom':
    case 'sole_top':
      if (state.selectedTexture === 'leather') {
        material.map = textureMap;
        material.normalMap = textureLoader.load('/textures/Leather/FabricLeatherCowhide001_NRM_2K.jpg');
        material.bumpMap = textureLoader.load('/textures/Leather/FabricLeatherCowhide001_BUMP_2K.jpg');
        material.roughnessMap = textureLoader.load('/textures/Leather/FabricLeatherCowhide001_GLOSS_2K.jpg');
      } else if (state.selectedTexture === 'rubber') {
        material.normalMap = textureLoader.load('/textures/Rubber/Rubber_Sole_002_normal.jpg');
        material.bumpMap = textureLoader.load('/textures/Rubber/Rubber_Sole_002_height.png');
        material.roughnessMap = textureLoader.load('/textures/Rubber/Rubber_Sole_002_roughness.jpg');
        material.map = textureMap;
        material.ambientOcclusionMap = textureLoader.load('/textures/Rubber/Rubber_Sole_002_ambientOcclusion.jpg');
      }
      else if (state.selectedTexture === 'organic') {
        material.normalMap = textureLoader.load('/textures/Organic/Abstract_Organic_006_normal.jpg');
        material.bumpMap = textureLoader.load('/textures/Organic/Abstract_Organic_006_height.png');
        material.roughnessMap = textureLoader.load('/textures/Organic/Abstract_Organic_006_roughness.jpg');
        material.map = textureMap;
        material.ambientOcclusionMap = textureLoader.load('/textures/Organic/Abstract_Organic_006_ambientOcclusion.jpg');
      }
      else if (state.selectedTexture === 'lava') {
        material.normalMap = textureLoader.load('/textures/Lava/Lava_005_NORM.jpg');
        material.bumpMap = textureLoader.load('/textures/Lava/Lava_005_DISP.png');
        material.roughnessMap = textureLoader.load('/textures/Lava/Lava_005_ROUGH.jpg');
        material.map = textureMap;
        material.ambientOcclusionMap = textureLoader.load('/textures/Lava/Lava_005_OCC.jpg');
      }
      break;
    // Voeg meer cases toe voor andere onderdelen indien nodig
  }
};


</script>

<template>
  <div>
    <!-- Preloader -->
    <div v-if="loading"
      class="preloader fixed w-full h-full flex justify-center items-center flex-col bg-black text-white">
      <!-- Add your preloader animation or message here -->
      <img src="/images/logo.png" alt="" class="mt-10 mb-5">
      <img src="/images/preloader.jpg" alt="Loading..." class="h-85 bg-gradient-to-b from-transparent to-black opacity-60 rounded-full"/>
      <p class="absolute bottom-20">RUNNING TO THE SHOE LAB...</p>
    </div>
    <div v-else>
      <div class="fixed bottom-0 left-0 right-0 bg-stone-200 py-4">
        <!-- Previous and Next buttons -->
        <div class="flex justify-center">
          <a href="#" @click="navigatePrev"><i class="fa-solid fa-arrow-left fa-xl relative top-3.5 right-5"></i></a>
          <h1 class="text-center text-[2em] font-bold pb-7">{{ parts[currentPartIndex].display }}</h1>
          <a href="#" @click="navigateNext"><i class="fa-solid fa-arrow-right fa-xl relative top-3.5 left-5"></i></a>
        </div>

        <!-- Color options -->
        <div class="flex justify-center gap-8 items-center flex-wrap">
          <div v-for="color in colors" :key="color.color" class="text-center">
            <div :style="{ backgroundColor: color.hex }" @click="changeColor(color.hex, parts[currentPartIndex].name)"
              class="w-[42px] h-[42px] rounded-full mx-auto cursor-pointer"></div>
            <h2 class="mt-2">{{ color.color }}</h2>
          </div>
        </div>

        <!-- Texture options -->
<div class="flex justify-center gap-8 items-center flex-wrap mt-5">
  <div v-for="texture in textures" :key="texture.name" class="text-center">
    <div @click="changeTexture(texture.name, parts[currentPartIndex].name)" class="w-[42px] h-[42px] mx-auto cursor-pointer">
      <!-- Display texture preview or icon here -->
      <img :src="texture.preview" alt="Texture Preview" class="w-full h-full rounded-full">
    </div>
    <h2 class="mt-2">{{ texture.displayName }}</h2>
  </div>
</div>



      </div>
    </div>
  </div>
</template>

<style scoped></style>