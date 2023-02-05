<template>
  <div class="container" ref="containerRef"></div>
</template>
<script setup>
import * as THREE from "three";
import { onMounted, ref } from "vue";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";
import { RGBELoader } from "three/examples/jsm/loaders/RGBELoader";
const containerRef = ref();
// 初始化场景
const scene = new THREE.Scene();
// 初始化相机
const camera = new THREE.PerspectiveCamera(
  75,
  window.innerWidth / window.innerHeight,
  0.1,
  1000
);
camera.position.z = 5;
// 创建轨道控制器
let controls;

// 初始化渲染器
const renderer = new THREE.WebGLRenderer();
renderer.setSize(window.innerWidth, window.innerHeight);

const render = () => {
  renderer.render(scene, camera);
  requestAnimationFrame(render);
  controls && controls.update();
};

// // 房间贴纸数组
// const homeArr = ["4_r", "4_l", "4_u", "4_d", "4_f", "4_b"];
// // 材质
// const boxMaterial = [];
// homeArr.forEach((item) => {
//   // 纹理加载
//   let texture = new THREE.TextureLoader().load(`./homeImgs/living/${item}.jpg`);
//   //   创建材质
//   boxMaterial.push(new THREE.MeshBasicMaterial({ map: texture }));
// });

// // 初始化立方体
// const geometry = new THREE.BoxGeometry(10, 10, 10);
// // const metarial = new THREE.MeshBasicMaterial({ color: 0x00ff00 });
// const cube = new THREE.Mesh(geometry, boxMaterial);
// cube.geometry.scale(1, 1, -1);
// scene.add(cube);

// 球体
const geometry = new THREE.SphereGeometry(5, 32, 32);
const loader = new RGBELoader();
loader.load(
  "./homeImgs/hdr/ersisd-Merksem_Appartment_Living_4k.hdr",
  (texture) => {
    const material = new THREE.MeshBasicMaterial({ map: texture });
    const sphere = new THREE.Mesh(geometry, material);
    sphere.geometry.scale(1, 1, -1);
    scene.add(sphere);
  }
);

onMounted(() => {
  controls = new OrbitControls(camera, containerRef.value);
  // 阻尼
  controls.enableDamping = true;

  containerRef.value.appendChild(renderer.domElement);
  render();
});
</script>
