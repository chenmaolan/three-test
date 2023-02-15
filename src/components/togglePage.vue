<template>
  <div>
    <div class="container" ref="containerRef"></div>
  </div>
</template>
<script setup>
import { ref } from "@vue/reactivity";
import { onMounted } from "@vue/runtime-core";
import * as THREE from "three";
// import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";
import { DRACOLoader } from "three/examples/jsm/loaders/DRACOLoader";
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader";
import gsap from "gsap";
const containerRef = ref();
const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(
  45,
  window.innerWidth / window.innerHeight,
  0.1,
  1000
);
camera.position.set(0, 0, 10);
// scene.add(camera);
const texture = new THREE.TextureLoader().load("./assets/25s.jpg");
texture.mapping = THREE.EquirectangularRefractionMapping;
scene.background = texture;
scene.environment = texture;
const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.clearColor(0xffffff);
// let controls;
const render = () => {
  requestAnimationFrame(render);
  renderer.render(scene, camera);
  // controls && controls.update();
};

const initModel = () => {
  const dracoloader = new DRACOLoader();
  dracoloader.setDecoderPath("./draco/gltf/");
  const loader = new GLTFLoader();
  loader.setDRACOLoader(dracoloader);
  loader.load("./model/gr75.glb", (gltf) => {
    console.log(gltf, "gltf====");
    gltf.scene.position.set(3, 0, 0);
    listenMouse(gltf.scene.rotation);
    scene.add(gltf.scene);
  });
  loader.load("./model/xq6.glb", (gltf) => {
    console.log(gltf, "gltf====");
    gltf.scene.scale.set(0.08, 0.08, 0.08);
    gltf.scene.position.set(3, -8, 0);
    listenMouse(gltf.scene.rotation);
    scene.add(gltf.scene);
  });
  loader.load("./model/xz.glb", (gltf) => {
    console.log(gltf, "gltf====");
    gltf.scene.scale.set(0.08, 0.08, 0.08);
    gltf.scene.position.set(3, -16, 0);
    listenMouse(gltf.scene.rotation);
    scene.add(gltf.scene);
  });
  loader.load("./model/moon.glb", (gltf) => {
    console.log(gltf, "loder=====model");
    const moon = gltf.scene.children[0];
    console.log(moon, "moon====");
    for (let y = 0; y < 10; y++) {
      let moonInstand = new THREE.InstancedMesh(
        moon.geometry,
        moon.material,
        50
      );
      for (let i = 0; i < 50; i++) {
        const matrix4 = new THREE.Matrix4();
        const size = Math.random() * 10 - 8;
        matrix4.makeScale(size, size, size);
        matrix4.makeTranslation(
          Math.random() * 1000 - 500,
          Math.random() * 1000 - 500,
          Math.random() * 1000 - 500
        );
        moonInstand.setMatrixAt(i, matrix4);
      }
      scene.add(moonInstand);

      gsap.to(moonInstand.position, {
        duration: Math.random() * 10 + 2,
        z: -1000,
        repeat: -1,
        ease: "linear",
      });
    }
  });
};
const listenMouse = (animationType) => {
  console.log(animationType, "animationType-=====");
  window.addEventListener("mousemove", (e) => {
    // console.log(e.clientY, "clienty----", e.clientX);
    const x = (e.clientX / window.innerWidth) * 2 - 1; // 范围值是 -1 至 1
    const y = -(e.clientY / window.innerHeight) * 2 + 1;
    // console.log(-(Math.PI / 8) * y, "client----", (Math.PI / 8) * x);
    let timeline2 = gsap.timeline();
    timeline2.to(animationType, {
      duration: 1,
      x: -(Math.PI / 8) * y,
      y: (Math.PI / 8) * x,
    });
  });
};
let page = 0;
const timeline = gsap.timeline();
window.addEventListener("mousewheel", (e) => {
  if (timeline.isActive()) return;
  console.log(e.wheelDelta);
  if (e.wheelDelta < 0) {
    page++;
    if (page >= 2) {
      page = 2;
    }
  }
  if (e.wheelDelta > 0) {
    page--;
    if (page <= 0) {
      page = 0;
    }
  }
  timeline.to(camera.position, {
    duration: 1,
    y: -page * 8,
    ease: "easeInOut",
  });
});
const initLight = () => {
  // 添加直线光
  let light1 = new THREE.DirectionalLight(0xffffff, 1);
  light1.position.set(0, 10, 10);
  let light2 = new THREE.DirectionalLight(0xffffff, 1);
  light1.position.set(0, 10, -10);
  let light3 = new THREE.DirectionalLight(0xffffff, 1);
  light1.position.set(10, 10, 10);
  scene.add(light1, light2, light3);
};
onMounted(() => {
  // controls = new OrbitControls(camera, renderer.domElement);
  // controls.enableDamping = true;
  containerRef.value.appendChild(renderer.domElement);
  initModel();
  initLight();
  render();
});

window.onresize = () => {
  // 更新摄像头长宽比
  camera.aspect = window.innerWidth / window.innerHeight;
  // 更新摄像头的矩阵
  camera.updateProjectionMatrix();
  // 更新屏幕宽高
  renderer.setSize(window.innerWidth, window.innerHeight);
  // 更新屏幕像素比
  renderer.setPixelRatio(window.devicePixelRatio);
};
</script>