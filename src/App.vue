<template>
  <div id="container" ref="container"></div>
</template>

<script setup>
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'
import { RGBELoader } from "three/examples/jsm/loaders/RGBELoader"
import { CSS2DRenderer, CSS2DObject } from "three/examples/jsm/renderers/CSS2DRenderer"
import { TWEEN } from "three/examples/jsm/libs/tween.module.min.js"
import { ref, onMounted } from "vue"

let pointLabel = null;
// 初始化场景
const scene = new THREE.Scene();

// 初始化相机
const camera = new THREE.PerspectiveCamera(75,window.innerWidth / window.innerHeight,0.1,1000);

// 相机位置
camera.position.set(0, 0, 0.1);

// 创建坐标辅助轴
const axes = new THREE.AxesHelper(5);
scene.add(axes);

// 初始化渲染器
const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.setSize(window.innerWidth, window.innerHeight);

const container = ref(null);

// 创建css2d标签
const labelRenderer = new CSS2DRenderer();
labelRenderer.setSize(window.innerWidth, window.innerHeight);
labelRenderer.domElement.style.position = 'absolute';
labelRenderer.domElement.style.top = 0;
labelRenderer.domElement.style.pointerEvents = 'none';

// 添加球体
const geometry = new THREE.SphereGeometry(10, 40, 40);
const loader = new RGBELoader();
loader.load('11hdr.hdr',(texture) => {
  const material = new THREE.MeshBasicMaterial({ map: texture });
  const sphere = new THREE.Mesh(geometry, material);
  sphere.geometry.scale(1, 1, -1);
  sphere.name = '物体';
  sphere.position.set(0, 0, 0);
  scene.add(sphere);
});

// 添加球体2
const geometry2 = new THREE.SphereGeometry(10, 40, 40);
const loader2 = new THREE.TextureLoader();
loader2.load('1.jpg', (texture) => {
  const material = new THREE.MeshBasicMaterial({ map: texture });
  const sphere = new THREE.Mesh(geometry2, material);
  sphere.geometry.scale(1,1,-1);
  sphere.name = '物体2';
  sphere.position.set(-20, 0, 0);
  scene.add(sphere);
});

// 创建控制器
const controls = new OrbitControls(camera, renderer.domElement);
controls.enableDamping = true;
controls.target.set(0,0,0);

createClickButton(10, 10, 0.01, 0.5, 4.681439945350212, -0.5272706600360021, 1.607439997249297,"台灯");
createClickButton(10, 10, 0.01, 0.45, -5.002445361352734, 0.7609046789933294, 6.444562725897698,"电视");

const render = () => {
  requestAnimationFrame(render);
  TWEEN.update();
  controls.update();
  labelRenderer.render(scene,camera);
  renderer.render(scene,camera);
};

// 创建css2d标签Div
// eslint-disable-next-line no-undef
function createLabelObj (text, vector) {
  const laberDiv = document.createElement('div');// 创建div容器
  laberDiv.className = 'label_name';
  laberDiv.id = 'labelId';
  laberDiv.innerText = text + '\n' + '参数:测试';
  const pointLabel = new CSS2DObject(laberDiv);
  pointLabel.position.set(vector.x, vector.y, vector.z);
  return pointLabel;
};

// 监听鼠标点击事件
function onDocumentMouseMove (e) {
  const raycaster = new THREE.Raycaster();
  const vector = new THREE.Vector2((e.clientX / window.innerWidth) * 2 - 1, -(e.clientY / window.innerHeight) * 2 + 1);
  raycaster.setFromCamera(vector, camera);
  const intersects = raycaster.intersectObjects(scene.children);
  if (intersects.length > 0) {
    // 判断参数[boxMesh]中模型对象是否与射线相交
    // console.log(intersects[0].point, '===');
    intersects.forEach(function (e) {
      const obj = e.object;
      // 判断相交的是否是精灵对象并且对应标签的名称，如果是鼠标变小手
      if (obj instanceof THREE.Mesh && obj.name.indexOf("台灯") > -1) {
        console.log("point:",intersects[0].point);
        console.log("obj:",obj);
        // 先移除旧的label,避免重复创建造成重叠
        scene.remove(pointLabel);
        pointLabel = createLabelObj(obj.name, obj.position);
        scene.add(pointLabel);
      }
      if (obj instanceof THREE.Mesh && obj.name.indexOf("电视") > -1) {
        // 相机当前位置
        let current1 = new THREE.Vector3();
        current1 = camera.position;
        // 相机的目标位置
        const target1 = new THREE.Vector3(-18,0,0.2);
        // 新的controls的target
        const target2 = new THREE.Vector3(-18,0,0.1);

        const tween = new TWEEN.Tween(current1).to(target1,2000).easing(TWEEN.Easing.Quadratic.InOut);
        tween.onUpdate((object) => {
          // 修改相机位置
          camera.lookAt(0, 0, 0);
          controls.target.set(target2.x, target2.y, target2.z);
          controls.update();
          camera.updateProjectionMatrix();
        });
        tween.onComplete((object) => {});
        tween.start();
      }
    });
  }
};

function createClickButton (width, height, depth, angle, x, y, z, name) {
  const loader = new THREE.TextureLoader();
  loader.load('tanhao.png', (texture) => {
    const windowGeometry = new THREE.BoxGeometry(width, height, depth);
    const windowMaterial = new THREE.MeshBasicMaterial({ map: texture,transparent: true });
    windowMaterial.opacity = 1.0;
    windowGeometry.transparent = true;
    windowGeometry.scale(0.04, 0.04, 0.04);
    const window = new THREE.Mesh(windowGeometry, windowMaterial);
    window.position.set(x, y, z);
    window.rotation.y += angle * Math.PI; // - 逆时针旋转，+ 顺时针旋转
    window.name = name;
    scene.add(window);
  });
}

onMounted(() => {
  window.addEventListener('click',onDocumentMouseMove);
  container.value.appendChild(labelRenderer.domElement);
  container.value.appendChild(renderer.domElement);
  render();
});
</script>

<style>
#container {
  margin: 0;
  padding: 0;
  width: 100vw;
  height: 100vh;
  overflow: hidden;
}
.label_name{
  color: rgb(0, 255, 255);
  width: 100px;
  height: 30px;
  font-size: 14px;
  padding: 15px;
  background-color: rgba(0, 0, 0, 0.5);
  border-radius: 4%;
}
.label_name::after{
  content: '';
  width: calc(100% + 22px);
  height: 39px;
  background-position: 0 0;
  position: absolute;
  left: -22px;
  bottom: -40px;
}
</style>
