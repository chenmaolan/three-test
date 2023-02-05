<template>
  <div class="container" ref="containerRef"></div>
</template>
<script setup>
import { onMounted, ref } from "vue";
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";

const containerRef = ref();

const scene = new THREE.Scene();

const camera = new THREE.PerspectiveCamera(
  75,
  window.innerWidth / window.innerHeight,
  0.1,
  1000
);
camera.position.set(0, 1.5, 6);

let controls;

const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.setSize(window.innerWidth, window.innerHeight);

const render = () => {
  renderer.render(scene, camera);
  controls && controls.update();
  requestAnimationFrame(render);
};

// 创建辅助坐标
const axes = new THREE.AxesHelper(5);
scene.add(axes);

onMounted(() => {
  controls = new OrbitControls(camera, containerRef.value);
  controls.enableDamping = true;
  containerRef.value.appendChild(renderer.domElement);
  render();
});
</script>
