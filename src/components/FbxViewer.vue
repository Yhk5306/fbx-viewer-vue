<template>
  <div ref="fbxContainer" class="fbx-viewer"></div>
</template>

<script setup lang="ts">
import {ref, onMounted, computed, toRef} from 'vue';
import * as THREE from 'three';
import { FBXLoader } from 'three/examples/jsm/loaders/FBXLoader';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';

const fbxContainer = ref(null);
let scene: THREE.Scene, camera: THREE.Camera, renderer: THREE.WebGLRenderer, fbxModel, controls: OrbitControls;

const props = defineProps({
  fbx:{
    type:String,
  },
  ToplightPosition: {
    type: Object as () => { x: number; y: number; z: number },
    default: () => ({ x: 0, y: 2, z: 0 })
  },
  TopPointLight:{
    type: Object as () => { color: string; intensity: number; distance: number },
    default: () => ({color:'#ffffff',intensity:1,distance:8})
  },
  BottomLightPosition:{
    type: Object as () => { x: number; y: number; z: number },
    default: () => ({ x: 0, y: -2, z: 0 })
  },
  BottomPointLight:{
    type: Object as () => { color: string; intensity: number; distance: number },
    default: () => ({color:'#421630',intensity:1,distance:10})
  },
  LeftLightPosition:{
    type: Object as () => { x: number; y: number; z: number },
    default: () => ({ x: 0, y: -2, z: 0 })
  },
  LeftPointLight:{
    type: Object as () => { color: string; intensity: number; },
    default: () => ({color:'#ffffff',intensity:1,distance:10})
  },
  RightLightPosition:{
    type: Object as () => { x: number; y: number; z: number },
    default: () => ({ x: 0, y: -2, z: 0 })
  },
  RightPointLight:{
    type: Object as () => { color: string; intensity: number; },
    default: () => ({color:'#ffffff',intensity:1,distance:10})
  },
  ObjColor:{
    type:String,
    default:'#fefefe'
  },
  SceneBg:{
    type: Object as () => {color:string,alpha:number},
    default: () => ({color:'#041839',alpha:0})
  },
  animate:{
    type:Object as () => {animate:boolean,duration:number},
    default: () => ({animate:true,duration:1000})
  },
  distance:{
    type:Object as () => {minDistance:number,maxDistance:number},
    default: () => ({minDistance:0.8,maxDistance:1})
  },
  autoRotate:{
    type:Object as () => {value:boolean,rotateSpeed:number},
    default:() => ({value:true,rotateSpeed:2})
  },
  cameraPosition:{
    type: Object as () => {x: number; y: number; z: number },
    default:() => ({x: 1,y: 1,z: 1})
  }

});


onMounted(() => {
  if (fbxContainer.value) {
    init();
    animate();
  }
});

function init() {
  scene = new THREE.Scene();
  THREE.ColorManagement.enabled = true;

  const containerWidth = fbxContainer.value.clientWidth;
  const containerHeight = fbxContainer.value.clientHeight;
  const aspectRatio = containerWidth / containerHeight;

  camera = new THREE.PerspectiveCamera(75, aspectRatio, 0.1, 1000);
  camera.position.set(props.cameraPosition?.x, props.cameraPosition?.y, props.cameraPosition?.z);
  camera.lookAt(0, 0, 0);

  renderer = new THREE.WebGLRenderer({ antialias: true });
  renderer.setSize(containerWidth, containerHeight);
  fbxContainer.value.appendChild(renderer.domElement);

  const loader = new FBXLoader();
  loader.load(props.fbx, (object) => {
    fbxModel = object;
    fbxModel.scale.set(0.01, 0.01, 0.01);
    centerObject(fbxModel);
    setObjectColor(fbxModel, props.ObjColor);
    setSceneBackgroundColor(props.SceneBg?.color, props.SceneBg?.alpha); // Sets a semi-transparent green background

    const animationMixer = new THREE.AnimationMixer(fbxModel);
    const rotationAnimation = new THREE.AnimationClip('rotation', 1, [
      new THREE.VectorKeyframeTrack('.rotation[y]', [0, 2], [0, Math.PI * 2]),
    ]);
    animationMixer.clipAction(rotationAnimation).setDuration(2000).play();

    scene.add(fbxModel);

    if (props.animate?.animate){
      performZoomOutAnimation(props.animate.duration);
    }
  });

  controls = new OrbitControls(camera, renderer.domElement);
  controls.enableDamping = true;
  controls.dampingFactor = 0.1;
  controls.autoRotate = props.autoRotate?.value; // Enable auto-rotation
  controls.autoRotateSpeed = props.autoRotate?.rotateSpeed; // Adjust auto-rotation speed as needed
  controls.minDistance = props.distance?.minDistance;
  controls.maxDistance = props.distance?.maxDistance;

  controls.mouseButtons = {
    LEFT: THREE.MOUSE.ROTATE,
    MIDDLE: THREE.MOUSE.DOLLY,
    RIGHT: THREE.MOUSE.NONE,
  };

  controls.update();

  // Ambient Light
  const ambientLight = new THREE.AmbientLight('#848789');
  scene.add(ambientLight);

  // right light
  const directionalLightRight = new THREE.DirectionalLight(props.RightPointLight?.color,props.RightPointLight?.intensity);
  directionalLightRight.position.set(props.RightLightPosition?.x,props.RightLightPosition?.y,props.RightLightPosition?.z);
  scene.add(directionalLightRight);

  // left light
  const directionalLightLeft = new THREE.DirectionalLight(props.LeftPointLight?.color,props.LeftPointLight?.intensity);
  directionalLightLeft.position.set(props.LeftLightPosition?.x,props.LeftLightPosition?.y,props.LeftLightPosition?.z);
  scene.add(directionalLightLeft);

  // bottom light
  const pointLight = new THREE.PointLight(props.BottomPointLight?.color,props.BottomPointLight?.intensity,props.BottomPointLight?.distance);
  pointLight.position.set(props.BottomLightPosition?.x,props.BottomLightPosition?.y,props.BottomLightPosition?.z);
  scene.add(pointLight);

  // top light
  const topLight = new THREE.PointLight(props.TopPointLight?.color,props.TopPointLight?.intensity,props.TopPointLight?.distance);
  topLight.position.set(props.ToplightPosition?.x,props.ToplightPosition?.y,props.ToplightPosition?.z);

  scene.add(topLight);
}

function centerObject(object: THREE.Object3D<THREE.Object3DEventMap>) {
  const box = new THREE.Box3().setFromObject(object);
  const center = box.getCenter(new THREE.Vector3());
  object.position.sub(center);
}

// Set object color
function setObjectColor(object: THREE.Group<THREE.Object3DEventMap>, hexColor: string) {
  const material = new THREE.MeshStandardMaterial({ color: hexColor });
  object.traverse((child) => {
    if (child instanceof THREE.Mesh) {
      child.material = material;
    }
  });
}

// Set background color to transparent
function setSceneBackgroundColor(hexColor: THREE.ColorRepresentation | undefined, alpha: number | undefined) {
  const clearColor = new THREE.Color(hexColor);
  renderer.setClearColor(clearColor, alpha);
}

function performZoomOutAnimation(duration: number) {
  const zoomOutDuration = duration; // Duration of the zoom-out animation in milliseconds

  const startPosition = new THREE.Vector3(0, 1,0);
  const targetPosition = new THREE.Vector3(1, 1,1);

  const startTime = Date.now();
  const endTime = startTime + zoomOutDuration;

  function updateCameraPosition() {
    const now = Date.now();
    if (now < endTime) {
      const progress = (now - startTime) / zoomOutDuration;
      const newPosition = new THREE.Vector3().lerpVectors(startPosition, targetPosition, progress);
      camera.position.copy(newPosition);
      controls.update(); // Update OrbitControls to match the camera position
      requestAnimationFrame(updateCameraPosition);
    }
  }

  updateCameraPosition();
}

function animate() {
  requestAnimationFrame(animate);
  controls.update();
  renderer.render(scene, camera);
}
</script>

<style scoped>
.fbx-viewer{
  width: 100%;
  height: 100%;
}
</style>
