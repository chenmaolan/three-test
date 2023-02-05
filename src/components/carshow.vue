<template>
  <div class="container">
    <div class="canves-container" ref="canvesRef"></div>
    <div class="car-content">
      <div class="title"><h1>汽车展示与选配</h1></div>
      <h2>选择车身颜色</h2>
      <div class="select">
        <template v-for="item in colors" :key="item">
          <div
            class="select-item"
            :style="{ backgroundColor: item }"
            @click="handleSelectColor(item)"
          ></div>
        </template>
      </div>
      <h2>选择车轱辘颜色</h2>
      <div class="select">
        <template v-for="item in colors" :key="item">
          <div
            class="select-item"
            :style="{ backgroundColor: item }"
            @click="handleSelectWheelsColor(item)"
          ></div>
        </template>
      </div>
      <h2>选择车身贴膜材质</h2>
      <div class="select">
        <template v-for="item in metarial" :key="item.name">
          <div class="select-item" @click="handleSelectCarMetarial(item)">
            {{ item.name }}
          </div>
        </template>
      </div>
    </div>
  </div>
</template>
<script setup>
import * as THEREE from "three";
import { onMounted, ref } from "vue";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader";
import { DRACOLoader } from "three/examples/jsm/loaders/DRACOLoader";
const canvesRef = ref();
const colors = ["red", "pink", "green", "gray", "purple", "#4077ca"];
const metarial = [
  { name: "磨砂", value: 1 },
  { name: "冰晶", value: 0 },
];
// 常见场景
const scene = new THEREE.Scene();
// 创建摄像头
const camera = new THEREE.PerspectiveCamera(
  75,
  window.innerWidth / window.innerHeight,
  0.1,
  1000
);
camera.position.set(0, 2, 6);
// 创建渲染器
const renderer = new THEREE.WebGLRenderer({
  // 抗锯齿
  antialias: true,
});
renderer.setSize(window.innerWidth, window.innerHeight);
// 创建控制器
let controls;

const render = () => {
  renderer.render(scene, camera);
  controls && controls.update();
  requestAnimationFrame(render);
};
// 轮毂
let wheels = [];
let carBody, fontCar, hoodCar, glassCar;
// console.log(carBody, fontCar, hoodCar, glassCar, "1111");
const bodyMaterial = new THEREE.MeshPhysicalMaterial({
  color: 0xff0000,
  metalness: 1, //金属度
  roughbess: 0.5,
  clearcoat: 1,
  clearcoatRoughness: 0,
});
const fontMaterial = new THEREE.MeshPhysicalMaterial({
  color: 0xff0000,
  metalness: 1, //金属度
  roughbess: 0.5,
  clearcoat: 1,
  clearcoatRoughness: 0,
});
const hoodMaterial = new THEREE.MeshPhysicalMaterial({
  color: 0xff0000,
  metalness: 1, //金属度
  roughbess: 0.5,
  clearcoat: 1,
  clearcoatRoughness: 0,
});
const wheelsMaterial = new THEREE.MeshPhysicalMaterial({
  color: 0xff0000,
  metalness: 1, //金属度
  roughbess: 0.1,
});
const glassMaterial = new THEREE.MeshPhysicalMaterial({
  color: 0xffffff,
  metalness: 0, //金属度
  roughness: 0.1,
  transmission: 1,
  transparent: true,
});

const handleSelectColor = (color) => {
  bodyMaterial.color.set(color);
  fontMaterial.color.set(color);
  hoodMaterial.color.set(color);
  glassMaterial.color.set(color);
};
const handleSelectWheelsColor = (color) => {
  wheelsMaterial.color.set(color);
};
const handleSelectCarMetarial = (item) => {
  bodyMaterial.clearcoatRoughness = item.value;
  fontMaterial.clearcoatRoughness = item.value;
  hoodMaterial.clearcoatRoughness = item.value;
};
onMounted(() => {
  // 将渲染器插入dom
  canvesRef.value.appendChild(renderer.domElement);
  renderer.setClearColor("#000");
  scene.background = new THEREE.Color("#ccc");
  scene.environment = new THEREE.Color("#ccc");
  render();

  // 添加网格地面
  const GridHelper = new THEREE.GridHelper(10, 10);
  GridHelper.material.opacity = 0.2;
  GridHelper.material.transparent = true;
  scene.add(GridHelper);

  controls = new OrbitControls(camera, renderer.domElement);
  controls.update();

  // 添加汽车模型
  const loader = new GLTFLoader();
  const dracoloader = new DRACOLoader();
  dracoloader.setDecoderPath("./draco/gltf/");
  loader.setDRACOLoader(dracoloader);
  loader.load("./model/bmw01.glb", (gltf) => {
    const bmw = gltf.scene;
    console.log(bmw, "gltf=====");
    bmw.traverse((child) => {
      if (child.isMesh) {
        console.log(child.name, "child---name");
        if (child.name.includes("前脸")) {
          child.material = fontMaterial;
          fontCar = child;
        }
        if (child.name.includes("轮毂")) {
          child.material = wheelsMaterial;
          wheels.push(child);
        }
        if (child.name.includes("Mesh002")) {
          child.material = bodyMaterial;
          carBody = child;
        }
        if (child.name.includes("引擎盖_1")) {
          child.material = hoodMaterial;
          hoodCar = child;
        }
        if (child.name.includes("挡风玻璃")) {
          child.material = glassMaterial;
          glassCar = child;
        }
      }
    });

    console.log(carBody, fontCar, hoodCar, glassCar, 2222);
    scene.add(gltf.scene);
  });
  // 正前方
  const light1 = new THEREE.DirectionalLight(0xffffff, 1);
  light1.position.set(0, 0, 10);
  scene.add(light1);
  // 正后方
  const light2 = new THEREE.DirectionalLight(0xffffff, 1);
  light2.position.set(0, 0, -10);
  scene.add(light2);
  // 右
  const light3 = new THEREE.DirectionalLight(0xffffff, 1);
  light3.position.set(10, 0, 0);
  scene.add(light3);
  // 左
  const light4 = new THEREE.DirectionalLight(0xffffff, 1);
  light4.position.set(-10, 0, 0);
  scene.add(light4);
  //
  const light5 = new THEREE.DirectionalLight(0xffffff, 1);
  light5.position.set(0, 10, 0);
  scene.add(light5);
  const light6 = new THEREE.DirectionalLight(0xffffff, 1);
  light6.position.set(5, -10, 0);
  scene.add(light6);
  const light7 = new THEREE.DirectionalLight(0xffffff, 1);
  light7.position.set(0, 10, 5);
  scene.add(light7);
  const light8 = new THEREE.DirectionalLight(0xffffff, 1);
  light8.position.set(0, 10, -5);
  scene.add(light8);
  const light9 = new THEREE.DirectionalLight(0xffffff, 1);
  light9.position.set(-5, 10, 0);
  scene.add(light9);
});
</script>

<style>
.car-content {
  position: fixed;
  top: 0;
  right: 0;
}
.select {
  display: flex;
  padding: 10px;
}
.select-item {
  width: 50px;
  height: 50px;
  border-radius: 10px;
  margin: 0 5px;
  /* pointer-events: none; */
}
</style>
