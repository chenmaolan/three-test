<template>
  <div class="container" ref="containerRef">
    <!-- <img
      style="position: fixed; top: 0; left: 0; width: 100px; height: 100px"
      :src="url"
    /> -->
  </div>
</template>
<script setup>
import * as THREE from "three";
import { onMounted, ref } from "vue";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";
// import { TrackballControls } from "three/examples/jsm/controls/TrackballControls";
// import { RGBELoader } from "three/examples/jsm/loaders/RGBELoader";
const containerRef = ref();
// 初始化场景
const scene = new THREE.Scene();
// 初始化相机
const camera = new THREE.PerspectiveCamera(
  75,
  window.innerWidth / window.innerHeight,
  0.1,
  100
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
  // if (renderer.domElement) {
  //   url.value = renderer.domElement.toDataURL();
  //   // console.log(renderer.domElement, "url====", url);
  // }
};

// 房间贴纸数组
const homeDirectionArr = ["r", "l", "u", "d", "f", "b"];
const createRoomFace = (roonNumber, folder) => {
  const roomFaceArr = [];
  homeDirectionArr.forEach((item) => {
    // 纹理加载
    let texture = new THREE.TextureLoader().load(
      `./homeImgs/${folder}/${roonNumber}_${item}.jpg`
    );
    //   创建材质
    roomFaceArr.push(new THREE.MeshBasicMaterial({ map: texture }));
  });
  return roomFaceArr;
};
let point = new THREE.Vector2();
const onPointerMove = (e) => {
  point.x = (e.clientX / window.innerWidth) * 2 - 1;
  point.y = -(e.clientX / window.innerWidth) * 2 + 1;
  // console.log(point, "point=====");
};

// 创建一个箭头
const SpriteMaterial = new THREE.SpriteMaterial({
  map: new THREE.TextureLoader().load("./assets/arrow.png"),
  side: THREE.DoubleSide,
});
const arrow = new THREE.Sprite(SpriteMaterial);
arrow.position.set(3.5, -3.5, 3.8);
arrow.scale.set(0.5, 0.5, 0.5);
arrow.rotateY(Math.PI / 2);
arrow.rotateX(Math.PI / 3);
scene.add(arrow);

// 创建投射光线
const raycaster = new THREE.Raycaster();
// const arrowHelper = new THREE.ArrowHelper(
//   raycaster.direction,
//   raycaster.origin,
//   15,
//   0xff0000,
//   1,
//   0.5
// );
// scene.add(arrowHelper);
// const raycasterhelp = new THREE.ray();
// scene.add(raycasterhelp);

// 材质
let boxMaterial = createRoomFace(19, "childroom");
// createRoomFace(4, "living", boxMaterial);
// 初始化立方体
const geometry = new THREE.BoxGeometry(10, 10, 10);
// const metarial = new THREE.MeshBasicMaterial({ color: 0x00ff00 });
const cube = new THREE.Mesh(geometry, boxMaterial);
cube.geometry.scale(1, 1, -1);
scene.add(cube);

// 辅助坐标
// const axes = new THREE.AxesHelper(20);
// scene.add(axes);

// 房间球体
// const geometry = new THREE.SphereGeometry(5, 32, 32);
// const loader = new RGBELoader();
// loader.load(
//   "./homeImgs/hdr/ersisd-Merksem_Appartment_Living_4k.hdr",
//   (texture) => {
//     const material = new THREE.MeshBasicMaterial({ map: texture });
//     const sphere = new THREE.Mesh(geometry, material);
//     sphere.geometry.scale(1, 1, -1);
//     scene.add(sphere);
//   }
// );

const handelClick = (e) => {
  e.preventDefault();
  // 通过摄像机和鼠标位置更新射线
  raycaster.setFromCamera(point, camera);
  // 计算物体与射线的交点
  console.log(arrow, "arrow=====");
  const intersects = raycaster.intersectObjects([arrow]);
  console.log(intersects, "intersects=====");
  if (intersects.length > 0) {
    console.log("点击进入房间");
    const timer = setTimeout(() => {
      camera.fov -= 1;
      if (camera.fov === 20) {
        clearTimeout(timer);
        camera.fov = 50;
        camera.updateProjectionMatrix();
        for (let i = 0; i < homeDirectionArr.length; i++) {
          scene.remove(boxMaterial[i]);
        }
        const newFace = createRoomFace(4, "living", boxMaterial);
        boxMaterial = newFace;
      }
    }, 50);
  }
};
document.addEventListener("pointermove", onPointerMove);
document.addEventListener("click", handelClick, false);
// let url = ref();
onMounted(() => {
  controls = new OrbitControls(camera, containerRef.value);
  // 阻尼
  controls.enableDamping = true;

  containerRef.value.appendChild(renderer.domElement);
  render();
});
</script>
