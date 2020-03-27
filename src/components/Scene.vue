<template>
  <div class="container">
    <canvas id="canvas"></canvas>
    <div class="bottomContainer">
      <button type="button" class="button primitive" @click="showObject(0)">Primitive</button>
      <button type="button" class="button model1" @click="showObject(1)">Model1</button>
      <button type="button" class="button model2" @click="showObject(2)">Model2</button>
      <button type="button" class="button texture" @click="showObject(3)">Texture</button>
    </div>
  </div>
</template>

<script>
import * as THREE from "three";
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";

export default {
  name: "Scene",
  data() {
    const scene = new THREE.Scene();
    const renderer = null;

    const fov = 45;
    const aspect = 2;
    const near = 0.1;
    const far = 100;
    const camera = new THREE.PerspectiveCamera(fov, aspect, near, far);
    
    const directLight = new THREE.DirectionalLight(0xffffff);
    const hemisphereLight = null;
    const spotLight1 = null;
    const spotLight2 = null;
    const pointLight1 = null;
    const pointLight2 = null;

    const geometry = new THREE.BoxGeometry(10, 10, 10);
    const material = new THREE.MeshBasicMaterial();
    const cube = new THREE.Mesh(geometry, material);
    const textureLoader = new THREE.TextureLoader();
    const imageTexture = null;
    const imageCube = null;
    const gltfLoader = new GLTFLoader();
    const cityGLTF = null;
    const zebraGLTF = null;
    const controls = null;
    let objectType = 0;

    return { 
      scene, 
      renderer, 
      camera, 
      directLight, 
      hemisphereLight, 
      spotLight1,
      spotLight2,
      pointLight1,
      pointLight2,
      geometry, 
      material, 
      cube, 
      gltfLoader,
      cityGLTF, 
      zebraGLTF,
      controls,
      objectType,
      textureLoader,
      imageTexture,
      imageCube
    };
  },
  mounted() {
    const $canvas = document.getElementById("canvas");
    this.renderer = new THREE.WebGLRenderer({
      antialias: true,
      canvas: $canvas
    });
    this.renderer.setSize(window.innerWidth, window.innerHeight);

    this.camera.position.set(0, 20, 40);
    
    this.controls = new OrbitControls(this.camera, $canvas);
    this.controls.target.set(0, 5, 0);
    this.controls.update();

    this.imageTexture = new THREE.MeshBasicMaterial({
      map: this.textureLoader.load("https://threejsfundamentals.org/threejs/resources/images/wall.jpg"),
    });
    this.imageCube = new THREE.Mesh(this.geometry, this.imageTexture);

    const skyColor = 0xB1E1FF;
    const groundColor = 0xB97A20;
    const intensity = 1;
    this.hemisphereLight = new THREE.HemisphereLight(skyColor, groundColor, intensity);
    this.scene.add(this.hemisphereLight);

    this.directLight.position.set(5, 10, 2);
    this.scene.add(this.directLight);
    this.scene.add(this.directLight.target);

    this.spotLight1 = new THREE.SpotLight(0xB1E1FF);
    this.spotLight1.position.set(150, 100, 100);
    this.spotLight1.castShadow = true;
    this.spotLight1.shadow.mapSize.width = 1024;
    this.spotLight1.shadow.mapSize.height = 1024;
    this.spotLight1.shadow.camera.near = 500;
    this.spotLight1.shadow.camera.far = 4000;
    this.spotLight1.shadow.camera.fov = 30;
    this.scene.add(this.spotLight1);

    this.spotLight2 = new THREE.SpotLight(0xB1E1FF);
    this.spotLight2.position.set(-150, 100, 100);
    this.spotLight2.castShadow = true;
    this.spotLight2.shadow.mapSize.width = 1024;
    this.spotLight2.shadow.mapSize.height = 1024;
    this.spotLight2.shadow.camera.near = 500;
    this.spotLight2.shadow.camera.far = 4000;
    this.spotLight2.shadow.camera.fov = 30;
    this.scene.add(this.spotLight2);

    this.pointLight1 = new THREE.PointLight(0xB1E1FF, 1, 100);
    this.pointLight1.position.set(50, 100, -50);
    this.scene.add(this.pointLight1);

    this.pointLight2 = new THREE.PointLight(0xB1E1FF, 1, 100);
    this.pointLight2.position.set(50, 100, 50);
    this.scene.add(this.pointLight2);

    this.showObject(0);
  },
  methods: {
    animate() {
      requestAnimationFrame(this.animate);
      this.renderer.render(this.scene, this.camera);
    },
    frameArea(sizeToFitOnScreen, boxSize, boxCenter, camera) {
      const halfSizeToFitOnScreen = sizeToFitOnScreen * 0.5;
      const halfFovY = THREE.MathUtils.degToRad(camera.fov * .5);
      const distance = halfSizeToFitOnScreen / Math.tan(halfFovY);
      const direction = (new THREE.Vector3())
          .subVectors(camera.position, boxCenter)
          .multiply(new THREE.Vector3(1, 0, 1))
          .normalize();
      camera.position.copy(direction.multiplyScalar(distance).add(boxCenter));
      camera.near = boxSize / 100;
      camera.far = boxSize * 100;
      camera.updateProjectionMatrix();
      camera.lookAt(boxCenter.x, boxCenter.y, boxCenter.z);
    },
    showObject(index) {
      this.objectType = index;
      this.removeAllObjects();

      switch(this.objectType) {
        case 0:
          this.renderCube();
          break;
        case 1:
          this.renderModel1();
          break;
        case 2:
          this.renderModel2();
          break;
        case 3:
          this.renderTexture();
          break;
      }
    },
    renderCube() {
      this.camera.position.set(0, 20, 40);
      this.controls.target.set(0, 5, 0);
      this.controls.update();
      const randomColor = Math.floor(Math.random()*16777215);
      this.cube.material = new THREE.MeshBasicMaterial({ color: randomColor })
      this.scene.add(this.cube);
      this.animate();
    },
    renderModel1() {
      this.gltfLoader.load('https://threejsfundamentals.org/threejs/resources/models/cartoon_lowpoly_small_city_free_pack/scene.gltf', (gltf) => {
        this.cityGLTF = gltf.scene;
        this.scene.add(this.cityGLTF);
        const box = new THREE.Box3().setFromObject(this.cityGLTF);
        const boxSize = box.getSize(new THREE.Vector3()).length();
        const boxCenter = box.getCenter(new THREE.Vector3());
        this.frameArea(boxSize * 0.5, boxSize, boxCenter, this.camera);
        this.controls.maxDistance = boxSize * 10;
        this.controls.target.copy(boxCenter);
        this.controls.update();
      });
    },
    renderModel2() {
      this.gltfLoader.load('https://threejsfundamentals.org/threejs/resources/models/animals/Zebra.gltf', (gltf) => {
        this.zebraGLTF = gltf.scene;
        this.scene.add(this.zebraGLTF);
        const box = new THREE.Box3().setFromObject(this.zebraGLTF);
        const boxSize = box.getSize(new THREE.Vector3()).length();
        const boxCenter = box.getCenter(new THREE.Vector3());
        this.frameArea(boxSize * 0.9, boxSize, boxCenter, this.camera);
        this.controls.maxDistance = boxSize * 10;
        this.controls.target.copy(boxCenter);
        this.controls.update();
      });
    },
    renderTexture() {
      this.camera.position.set(0, 20, 40);
      this.controls.target.set(0, 5, 0);
      this.controls.update();
      this.scene.add(this.imageCube);
      this.animate();
    },
    removeAllObjects() {
      if (this.cube) { this.scene.remove(this.cube); }
      if (this.cityGLTF) { this.scene.remove(this.cityGLTF); }
      if (this.zebraGLTF) { this.scene.remove(this.zebraGLTF); }
      if (this.imageCube) { this.scene.remove(this.imageCube); }
    }
  }
};
</script>

<style lang="scss">
.container {
  width: 100vw;
  height: 100vh;
}

.bottomContainer {
  position: absolute;
  bottom: 30px;
	width: 100%;
	text-align: center;
	z-index: 100;
	display:block;
}

.button {
  width: 200px;
  padding: 20px 0;
  margin: 0 20px 0 20px;
  border-width: 0;
  border-radius: 50px;
  color: #000000;
  font-size: 20px;
  font-weight: bold;
  outline: 0;

  &:hover {
    opacity: 0.8;
  }

  &.primitive {
    background-color: #6495ED;
  }

  &.model1 {
    background-color: #5F9EA0;
  }

  &.model2 {
    background-color: #FF69B4;
  }

  &.texture {
    background-color: #FFA500;
  }
}
</style>