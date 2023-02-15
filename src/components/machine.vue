<template>
  <div class="container" ref="containerRef"></div>
</template>
<script setup>
import { onMounted, onUnmounted, ref } from "vue";
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";
// import gsap from "gsap";
// import { DRACOLoader } from "three/examples/jsm/loaders/DRACOLoader";
// import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader";
const containerRef = ref();

const scene = new THREE.Scene();

const camera = new THREE.PerspectiveCamera(
  75,
  window.innerWidth / window.innerHeight,
  0.1,
  1000
);
camera.position.set(0, 0, 15);

let controls;

const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.setSize(window.innerWidth, window.innerHeight);

const render = () => {
  renderer.render(scene, camera);
  controls && controls.update();
  requestAnimationFrame(render);
};
// 眼眶
const curve = new THREE.EllipseCurve(
  0,
  0, // ax, aY
  8,
  3, // xRadius, yRadius
  0,
  2 * Math.PI, // aStartAngle, aEndAngle
  false, // aClockwise
  0 // aRotation
);
const points = curve.getPoints(50);
console.log(points, "points=====");
// const geometry = new THREE.BufferGeometry().setFromPoints(points);

// const material = new THREE.LineBasicMaterial({ color: 0xff0000 });

// Create the final object to add to the scene
// const ellipse = new THREE.Line(geometry, material);
// scene.add(ellipse);
// 眼珠
const cicleCurve = new THREE.EllipseCurve(
  0,
  0, // ax, aY
  3,
  3, // xRadius, yRadius
  0,
  2 * Math.PI, // aStartAngle, aEndAngle
  false, // aClockwise
  0 // aRotation
);
const circlePoints = cicleCurve.getPoints(50);
// const circleGeometry = new THREE.BufferGeometry().setFromPoints(circlePoints);

// const circleMaterial = new THREE.LineBasicMaterial({ color: 0xff0000 });

// Create the final object to add to the scene
// const circle = new THREE.Line(circleGeometry, circleMaterial);
// scene.add(circle);
// 粒子图片
const ctx = document.createElement("canvas").getContext("2d");
ctx.canvas.width = ctx.canvas.height = 32;

ctx.fillStyle = "#000";
ctx.fillRect(0, 0, 32, 32);

let grd = ctx.createRadialGradient(16, 16, 0, 16, 16, 16);
grd.addColorStop(0.0, "#fff");
grd.addColorStop(1.0, "#000");
ctx.fillStyle = grd;
ctx.beginPath();
ctx.rect(15, 0, 2, 32);
ctx.fill();
ctx.beginPath();
ctx.rect(0, 15, 32, 2);
ctx.fill();

grd = ctx.createRadialGradient(16, 16, 0, 16, 16, 16);
grd.addColorStop(0.1, "#ffff");
grd.addColorStop(0.6, "#0000");
ctx.fillStyle = grd;
ctx.fillRect(0, 0, 32, 32);

// const alphaMap = new THREE.CanvasTexture(ctx.canvas);

const sprite1 = new THREE.TextureLoader().load("assets/snowflake1.png");
const createSprite = (pointArr = [], num = 10, map = sprite1) => {
  const spriteGeometry = new THREE.BufferGeometry();
  let vertices = [];
  for (let i = 0; i < pointArr.length; i++) {
    for (let j = 0; j < num; j++) {
      const x =
        Math.random() * (pointArr[i].x + 2 - pointArr[i].x - 1) +
        pointArr[i].x -
        1;
      const y =
        Math.random() * (pointArr[i].y + 2 - pointArr[i].y - 1) +
        pointArr[i].y -
        1;
      const z = Math.random();
      vertices.push(x, y, z);
    }
  }
  spriteGeometry.setAttribute(
    "position",
    new THREE.Float32BufferAttribute(vertices, 3)
  );
  const pointMaterial = new THREE.PointsMaterial({
    size: 0.2,
    sizeAttenuation: true,
    map: map,
    alphaTest: 0.5,
    transparent: true,
    blending: THREE.AdditiveBlending,
    depthTest: false,
  });
  pointMaterial.color.setHSL(1.0, 0.3, 0.7);
  const particles = new THREE.Points(spriteGeometry, pointMaterial);
  scene.add(particles);
};
createSprite(points, 50);
createSprite(circlePoints, 20);
let universe = new THREE.Group();
// const createUniverse = (count = 100, map = sprite1) => {
//   for (let i = 0; i < count / 2; i++) {
//     let x = Math.random() * 100 - 50;
//     let y = Math.random() * 100 - 50;
//     let z = Math.random() * 100 - 50;
//     const SpriteMaterial = new THREE.SpriteMaterial({
//       map: map,
//       side: THREE.DoubleSide,
//       sizeAttenuation: true,
//       alphaTest: 0.5,
//       transparent: true,
//       blending: THREE.AdditiveBlending,
//       // depthTest: false,
//     });
//     const sprite = new THREE.Sprite(SpriteMaterial);
//     sprite.position.set(x, y, z);

//     sprite.name = `编号${i}`;
//     universe.add(sprite);
//   }
//   scene.add(universe);
// };
const createUniverse = (count = 100, map = sprite1) => {
  const universeGeometry = new THREE.BufferGeometry();
  const universeMaterial = new THREE.PointsMaterial({
    size: 0.5,
    sizeAttenuation: true,
    map: map,
    alphaTest: 0.5,
    transparent: true,
    blending: THREE.AdditiveBlending,
    depthTest: false,
  });
  const universePosition = new Float32Array((count * 3) / 2);
  const universeSeed = new Float32Array((count * 3) / 2);
  const universeSize = new Float32Array(count / 2);
  const universeName = new Float32Array(count / 2);

  for (let i = 0; i < count / 2; i++) {
    universePosition[i * 3 + 0] = Math.random() * 100 - 50;
    universePosition[i * 3 + 1] = Math.random() * 100 - 50;
    universePosition[i * 3 + 2] = Math.random() * 100 - 50;
    universeSeed[i * 3 + 0] = Math.random();
    universeSeed[i * 3 + 1] = Math.random();
    universeSeed[i * 3 + 2] = Math.random();
    universeSize[i] = Math.random() * 2 + 0.5;
    universeName[i] = i;
  }

  universeGeometry.setAttribute(
    "position",
    new THREE.BufferAttribute(universePosition, 3)
  );
  universeGeometry.setAttribute(
    "seed",
    new THREE.BufferAttribute(universeSeed, 3)
  );
  universeGeometry.setAttribute(
    "size",
    new THREE.BufferAttribute(universeSize, 1)
  );
  universeGeometry.setAttribute(
    "name",
    new THREE.BufferAttribute(universeName, 1)
  );
  console.log(universeGeometry, "universeGeometry");
  const points = new THREE.Points(universeGeometry, universeMaterial);
  // points.geometry("name", universeName);
  universe.add(points);
  console.log(points, "universe====");
  scene.add(universe);
};
createUniverse(10000);
// 创建辅助坐标
// const axes = new THREE.AxesHelper(5);
// scene.add(axes);
let point = new THREE.Vector2();
const onPointerMove = (e) => {
  point.x = (e.clientX / window.innerWidth) * 2 - 1;
  point.y = -(e.clientX / window.innerWidth) * 2 + 1;
  // console.log(point, "point=====");
};

// 创建投射光线
const raycaster = new THREE.Raycaster();
const handelClick = (e) => {
  e.preventDefault();
  // 通过摄像机和鼠标位置更新射线
  raycaster.setFromCamera(point, camera);
  // 计算物体与射线的交点
  console.log(universe, "arrow=====");
  const intersects = raycaster.intersectObjects(universe.children);
  console.log(intersects, "intersects=====");
  if (intersects.length > 0) {
    console.log("点击星星");
    alert(`星星编号${intersects[0].index}`);
  }
};
document.addEventListener("pointermove", onPointerMove);
document.addEventListener("click", handelClick, false);
onMounted(() => {
  controls = new OrbitControls(camera, containerRef.value);
  controls.enableDamping = true;
  containerRef.value.appendChild(renderer.domElement);
  render();
  // gsap.to(camera.position, {
  //   duration: 5,
  //   rotate: Math.PI * 2,
  //   repeat: -1,
  //   ease: "linear",
  // });
});
onUnmounted(() => {
  renderer.dispose();
});
</script>
