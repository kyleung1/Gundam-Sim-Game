<script lang="ts">
  import { browser } from "$app/environment";
  import * as THREE from "three";
  import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
  import { FBXLoader } from "three/examples/jsm/loaders/FBXLoader";
  import {
    GLTFLoader,
    type GLTF,
  } from "three/examples/jsm/loaders/GLTFLoader.js";

  function backHome() {
    window.location.href = "/";
  }

  let callFunction = "";

  if (browser) {
    const scene = new THREE.Scene();
    scene.background = new THREE.Color("gainsboro");
    const model1Div = document.getElementById("scene");
    const canvas = document.createElement("canvas");

    let width = 0;
    let height = 0;
    if (model1Div && canvas) {
      width = model1Div.clientWidth;
      height = model1Div.clientHeight;

      const camera = new THREE.PerspectiveCamera(75, width / height, 0.1, 1000);
      const webglRenderer = new THREE.WebGLRenderer({
        antialias: true,
        canvas: canvas,
      });
      webglRenderer.setPixelRatio(window.devicePixelRatio);

      webglRenderer.setSize(width, height);
      model1Div.appendChild(webglRenderer.domElement);

      const controls = new OrbitControls(camera, webglRenderer.domElement);
      controls.target = new THREE.Vector3(0, 0, -15.5);
      controls.enableDamping = true;

      const glftLoader = new GLTFLoader();
      const fbxLoader = new FBXLoader();
      const ambientLight = new THREE.AmbientLight(0xffffff);
      scene.add(ambientLight);

      let gundamModel: THREE.Group<THREE.Object3DEventMap>;
      function addGundam() {
        fbxLoader.load("rx-78/test/source/model.fbx", (fbxScene) => {
          gundamModel = fbxScene;
          gundamModel.scale.set(0.1, 0.1, 0.1);
          gundamModel.position.set(0, -10, -15.5);
          if (beamSaberModel && beamSaberModel.scene) {
            scene.remove(beamSaberModel.scene);
          }
          if (beamRifle && beamRifle.scene) {
            scene.remove(beamRifle.scene);
          }
          if (zaku && zaku.scene) {
            scene.remove(zaku.scene);
          }
          if (asteroid && asteroid.scene) {
            scene.remove(asteroid.scene);
          }
          if (redZaku && redZaku.scene) {
            scene.remove(redZaku.scene);
          }
          scene.add(gundamModel);
        });
      }

      let beamSaberModel: GLTF;
      function addSaber() {
        glftLoader.load("lightsaber/scene.gltf", (gltfScene) => {
          beamSaberModel = gltfScene;
          // beamSaberModel.scene.scale.set(0.05, 0.15, 0.05);
          beamSaberModel.scene.position.set(0, 0, -15.5);
          if (gundamModel) {
            scene.remove(gundamModel);
          }
          if (beamRifle && beamRifle.scene) {
            scene.remove(beamRifle.scene);
          }
          if (zaku && zaku.scene) {
            scene.remove(zaku.scene);
          }
          if (asteroid && asteroid.scene) {
            scene.remove(asteroid.scene);
          }
          if (redZaku && redZaku.scene) {
            scene.remove(redZaku.scene);
          }
          scene.add(gltfScene.scene);
        });
      }

      let beamRifle: GLTF;
      function addRifle() {
        glftLoader.load("rifle/scene.gltf", (gltfScene) => {
          beamRifle = gltfScene;
          beamRifle.scene.position.set(0, -8, -10);
          beamRifle.scene.scale.set(6, 6, 6);
          console.log(scene.children);
          if (gundamModel) {
            scene.remove(gundamModel);
          }
          if (beamSaberModel && beamSaberModel.scene) {
            scene.remove(beamSaberModel.scene);
          }
          if (zaku && zaku.scene) {
            scene.remove(zaku.scene);
          }
          if (asteroid && asteroid.scene) {
            scene.remove(asteroid.scene);
          }
          if (redZaku && redZaku.scene) {
            scene.remove(redZaku.scene);
          }
          scene.add(gltfScene.scene);
        });
      }

      let zaku: GLTF;
      function addZaku() {
        glftLoader.load("zaku/scene.gltf", (gltfScene) => {
          zaku = gltfScene;
          zaku.scene.scale.set(2, 2, 2);
          zaku.scene.position.set(0, -8, -15.5);
          if (gundamModel) {
            scene.remove(gundamModel);
          }
          if (beamSaberModel && beamSaberModel.scene) {
            scene.remove(beamSaberModel.scene);
          }
          if (beamRifle && beamRifle.scene) {
            scene.remove(beamRifle.scene);
          }
          if (asteroid && asteroid.scene) {
            scene.remove(asteroid.scene);
          }
          if (redZaku && redZaku.scene) {
            scene.remove(redZaku.scene);
          }
          scene.add(gltfScene.scene);
        });
      }

      let asteroid: GLTF;
      function addAsteroid() {
        glftLoader.load("asteroid/scene.gltf", (gltfScene) => {
          asteroid = gltfScene;
          gltfScene.scene.scale.set(5, 5, 5);
          gltfScene.scene.position.set(0, -5, -15.5);
          if (gundamModel) {
            scene.remove(gundamModel);
          }
          if (beamSaberModel && beamSaberModel.scene) {
            scene.remove(beamSaberModel.scene);
          }
          if (beamRifle && beamRifle.scene) {
            scene.remove(beamRifle.scene);
          }
          if (zaku && zaku.scene) {
            scene.remove(zaku.scene);
          }
          if (redZaku && redZaku.scene) {
            scene.remove(redZaku.scene);
          }

          scene.add(gltfScene.scene);
        });
      }

      let redZaku: GLTF;
      function addRedZaku() {
        glftLoader.load("char-zaku/scene.glb", (gltfScene) => {
          redZaku = gltfScene;
          gltfScene.scene.scale.set(5, 5, 5);
          gltfScene.scene.position.set(0, 0, -15.5);
          gltfScene.scene.rotateY((3 * Math.PI) / 2);
          if (gundamModel) {
            scene.remove(gundamModel);
          }
          if (beamSaberModel && beamSaberModel.scene) {
            scene.remove(beamSaberModel.scene);
          }
          if (beamRifle && beamRifle.scene) {
            scene.remove(beamRifle.scene);
          }
          if (zaku && zaku.scene) {
            scene.remove(zaku.scene);
          }
          if (asteroid && asteroid.scene) {
            scene.remove(asteroid.scene);
          }

          scene.add(gltfScene.scene);
        });
      }

      function checkWhichModel() {
        if (callFunction === "addGundam") {
          console.log(callFunction);
          callFunction = "";
          addGundam();
        } else if (callFunction === "addSaber") {
          callFunction = "";
          addSaber();
        } else if (callFunction === "addRifle") {
          callFunction = "";
          addRifle();
        } else if (callFunction === "addZaku") {
          callFunction = "";
          addZaku();
        } else if (callFunction === "addAsteroid") {
          callFunction = "";
          addAsteroid();
        } else if (callFunction === "addRedZaku") {
          callFunction = "";
          addRedZaku();
        }
      }

      function animate() {
        checkWhichModel();
        controls.update();
        webglRenderer.render(scene, camera);
        requestAnimationFrame(animate);
      }
      animate();
    }
  }
</script>

<div
  class="bg-neutral-700 h-full flex flex-col items-center text-cyan-400 font-mono font-semibold"
>
  <img src="logo.png" alt="logo" class="w-1/5 mt-10" />
  <h1 class="text-3xl">Simulation Game</h1>

  <div class="flex flex-col items-center bg-white w-screen my-10">
    <h1 class="text-7xl text-center mt-10">Credits</h1>
    <p class="p-10 w-2/5 text-xl">
      The game was created by me, <a href="https://github.com/kyleung1"
        >kyleung1</a
      >. However, none of the 3D models in this game were made by me. Here are
      credits to their original creators on Sketchfab. Click on a model to view
      it in the viewer below.
    </p>
    <div>
      <button
        class="text-3xl text-center mt-10 hover:underline"
        on:click={() => (callFunction = "addGundam")}>Gundam</button
      >
      <p>
        Creator: <a
          href="https://sketchfab.com/3d-models/the-origingundamthe-origin-ver-b1fbcc97214a431bbc024f7bc929c41a"
          class="hover:underline">みそ太郎</a
        >
      </p>
      <button
        class="text-3xl text-center mt-10 hover:underline"
        on:click={() => (callFunction = "addZaku")}>Zaku</button
      >
      <p>
        Creator: <a
          href="https://sketchfab.com/3d-models/gundam-zako-3-0fbcea94273d43ecbb0e9d278bb1ea8e"
          class="hover:underline">Hiago Tadeu</a
        >
      </p>
      <button
        class="text-3xl text-center mt-10 hover:underline"
        on:click={() => (callFunction = "addRedZaku")}>Red Zaku</button
      >
      <p>
        Creator: <a
          href="https://sketchfab.com/3d-models/low-poly-ms-gundam-ms-06s-chars-zaku-ii-ddb4574a9eb14b55b6cb52f5ae77f279"
          class="hover:underline">tipatat</a
        >
      </p>
      <button
        class="text-3xl text-center mt-10 hover:underline"
        on:click={() => (callFunction = "addRifle")}>Rifle</button
      >
      <p>
        Creator: <a
          href="https://sketchfab.com/3d-models/ar-h470-4b38c160243743d681d6fdaf3bd77ddb"
          class="hover:underline">Frostoise</a
        >
      </p>
      <button
        class="text-3xl text-center mt-10 hover:underline"
        on:click={() => (callFunction = "addSaber")}>Saber</button
      >
      <p>
        Creator: <a
          href="https://sketchfab.com/3d-models/obi-wans-lightsaber-6e40dabb7cf3455bae70cebbf107a8a9"
          class="hover:underline">rkmorello</a
        >
      </p>
      <button
        class="text-3xl text-center mt-10 hover:underline"
        on:click={() => (callFunction = "addAsteroid")}>Asteroid</button
      >
      <p>
        Creator: <a
          href="https://sketchfab.com/3d-models/asteroide-low-poly-c9e06fdad0374345b5c516b89a57d1ec"
          class="hover:underline">AkinD</a
        >
      </p>
    </div>

    <div id="scene" class="h-[400px] w-[400px] my-10">
      <!-- filler -->
    </div>
  </div>

  <div class="hover:border-2 border-blue-400 my-10 rounded-md">
    <div class="hover:border-2 border-cyan-400">
      <button
        class="bg-black p-2 hover:border-2 border-blue-400"
        on:click={backHome}>Back To Menu</button
      >
    </div>
  </div>
</div>
