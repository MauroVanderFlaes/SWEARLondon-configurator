<script setup>
import { ref, onMounted } from "vue";
import * as THREE from "three";

//import GLFT loader
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader.js";

// Import DRACOLoader
import { DRACOLoader } from "three/examples/jsm/loaders/DRACOLoader.js";

//import orbit controls 
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js';

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
                    child.material.color.set("#000000");
                }
                if (child.name == "outside_2") {
                    child.material.color.set("#000000");
                }
                if (child.name == "outside_3") {
                    child.material.color.set("#000000");
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

    camera.position.z = 2;

    //add ambient
    const ambientLight = new THREE.AmbientLight(0xffffff, 0.5);
    scene.add(ambientLight);

    //add clock
    const clock = new THREE.Clock();

    function animate() {
        const elapsedTime = clock.getElapsedTime();
        requestAnimationFrame(animate);

        renderer.render(scene, camera);
    }

    animate();
});

</script>

<template>
    <div>
    </div>
</template>