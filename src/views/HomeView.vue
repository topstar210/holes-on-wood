<template>
  <div>
    <div ref="container" />
    <div class="control-card">
      <button @click="clearBox">Clear</button>
      <button @click="addCylinder">Make Holes</button>
    </div>
    <div class="texture-group">
      <div class="texture1" @click="selectTexture('/textures/1.png')"></div>
      <div class="texture2" @click="selectTexture('/textures/2.png')"></div>
    </div>
    <div class="box-size-container">
      <div style="position: relative; left: -50%; display: flex;">
        <div class="mx-1">
          <label for="">Length</label><input type="number" v-model="boxL" :disabled="inputDisabled" />
        </div>
        <div class="mx-1">
          <label for="">Broad</label><input type="number" v-model="boxB" :disabled="inputDisabled" />
        </div>
        <div class="mx-1">
          <label for="">Strength</label><input type="number" v-model="boxS" :disabled="inputDisabled" />
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import * as THREE from 'three';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'
import { TransformControls } from 'three/addons/controls/TransformControls';
import { CSG } from 'three-csg-ts'

export default {
  data() {
    return {
      texture: "/textures/2.png",
      cPos: [8, 5, 0],
      cParams: [0.5, 0.5, 5, 30],

      boxL: 8,
      boxB: 15,
      boxS: 1,

      block: null,
      subRes: null,
      box: null,
      cylinder: null,
      scene: null,

      inputDisabled: false,
    };
  },
  mounted() {
    const _self = this;
    // create a Three.js scene
    const scene = new THREE.Scene();

    // create a camera
    const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 1, 1000);
    camera.position.set(0, 0, 13);

    // create a renderer
    const renderer = new THREE.WebGLRenderer();
    renderer.setSize(window.innerWidth, window.innerHeight);
    renderer.setClearColor( 0xffffff, 0);

    // add the renderer to the component's container element
    this.$refs.container.appendChild(renderer.domElement);

    // render the scene
    const animate = () => {
      requestAnimationFrame(animate);
      renderer.render(scene, camera);
    };

    const orbit = new OrbitControls(camera, renderer.domElement)
    orbit.enableDamping = true;
    orbit.update();
    orbit.addEventListener('change', animate);

    const control = new TransformControls(camera, renderer.domElement);
    control.addEventListener('change', animate);

    // Make a wood mesh
    const texture = new THREE.TextureLoader().load(this.texture);
    const box = new THREE.Mesh(
      new THREE.BoxGeometry(this.boxL, this.boxB, this.boxS),
      new THREE.MeshBasicMaterial({ map: texture })
    );
    this.box = box;

    // Make a cylinder mesh 
    const cylinder = new THREE.Mesh(
      new THREE.CylinderGeometry(this.cParams[0], this.cParams[1], this.cParams[2], this.cParams[3]),
      new THREE.MeshNormalMaterial()
    );
    cylinder.geometry.rotateX(Math.PI / 2);
    cylinder.position.set(this.cPos[0], this.cPos[1], this.cPos[2]);
    this.cylinder = cylinder;

    control.attach(cylinder);
    this.control = control;
    // scene.add(control);

    // 
    control.addEventListener('dragging-changed', function (event) {
      if (event.value == false) {
        while (scene.children.length > 0) {
          scene.remove(scene.children[0]);
        }
        if(_self.subRes == null) this.subRes = null;

        const hole = new THREE.Mesh(new THREE.CylinderGeometry(_self.cParams[0], _self.cParams[1], _self.cParams[2], _self.cParams[3]));
        hole.geometry.rotateX(Math.PI / 2);
        hole.geometry.translate(cylinder.position.x, cylinder.position.y, cylinder.position.z);

        this.subRes = this.subRes ? CSG.subtract(this.subRes, hole) : CSG.subtract(_self.box, hole);
        _self.subRes = this.subRes;

        scene.add(this.subRes, cylinder);
        scene.add(control);

        animate();
      }
      orbit.enabled = !event.value;
    });

    // add the meshes to the scene
    scene.add(box);
    this.scene = scene;
    
    animate();
  },
  methods: {
    clearBox() {
      this.subRes = null;
      while (this.scene.children.length > 0) {
        this.scene.remove(this.scene.children[0]);
      }
      this.scene.add(this.box);
      this.inputDisabled = false;
    },
    addCylinder() {
      this.cylinder.position.set(this.cPos[0], this.cPos[1], this.cPos[2]);
      this.scene.add(this.cylinder);
      this.scene.add(this.control);
      this.inputDisabled = true;
    },
    selectTexture(textureUrl) {
      this.texture = textureUrl;
      this.box.material.map = new THREE.TextureLoader().load(this.texture);
      this.box.material.needsUpdate = true;
    },
    changeBoxSize() {
      const scaleFactorX = this.boxL / this.box.geometry.parameters.width;
      const scaleFactorY = this.boxB / this.box.geometry.parameters.height;
      const scaleFactorZ = this.boxS / this.box.geometry.parameters.depth;
      this.box.scale.set( scaleFactorX, scaleFactorY, scaleFactorZ );
    }
  },
  watch: {
    boxL(newVal) {
      this.changeBoxSize();
    },
    boxB(newVal) {
      this.changeBoxSize();
    },
    boxS(newVal) {
      this.changeBoxSize();
    },
  }
};
</script>

<style scoped>
.control-card {
  position: absolute;
  top: 5px;
  right: 10px;
  display: grid;
}
.control-card button {
  border-width: 0;
  background-color: #3EB2FD;
  background-image: linear-gradient(1deg, #4F58FD, #149BF3 99%);
  border-radius: 100px;
  color: #FFFFFF;
  cursor: pointer;
  font-size: 1rem;
  line-height: 1.5;
  padding: 6px 20px;
  text-align: center;
  transition: background-color .2s,background-position .2s;
  margin-bottom: 5px;
}
.texture-group{
  position: absolute;
  top: 5px;
}
.texture-group .texture1{
  width: 80px;
  height: 80px;
  margin: 5px;
  background: url('/textures/1.png');
  cursor: pointer;
}
.texture-group .texture2{
  width: 80px;
  height: 80px;
  margin: 5px;
  background: url('/textures/2.png');
  cursor: pointer;
}

.box-size-container{
  position: absolute;
  bottom: 10px;
  left: 50%; 
}
.box-size-container input {
  width: 50px;
}
.mx-1{
  margin: 0 4px 0 4px;
}
</style>