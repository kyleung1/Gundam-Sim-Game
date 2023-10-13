<script lang="ts">
  import * as THREE from "three";
  import { browser } from "$app/environment";
  import {
    CSS3DRenderer,
    CSS3DObject,
    CSS3DSprite,
  } from "three/examples/jsm/renderers/CSS3DRenderer.js";
  //   import {
  //     GLTFLoader,
  //     type GLTF,
  //   } from "three/examples/jsm/loaders/GLTFLoader.js";
  import { FBXLoader } from "three/examples/jsm/loaders/FBXLoader";

  if (browser) {
    let prevTime = performance.now();
    const scene = new THREE.Scene();
    const camera = new THREE.PerspectiveCamera(
      75,
      window.innerWidth / window.innerHeight / 2,
      0.1,
      1000,
    );
    const webglRenderer = new THREE.WebGLRenderer();
    const CSS3Drenderer = new CSS3DRenderer();
    webglRenderer.setPixelRatio(window.devicePixelRatio);
    webglRenderer.setSize(window.innerWidth, window.innerHeight);
    CSS3Drenderer.setSize(window.innerWidth, window.innerHeight);
    CSS3Drenderer.domElement.style.position = "absolute";
    CSS3Drenderer.domElement.style.top = "0";
    document.body.appendChild(webglRenderer.domElement);
    document.body.appendChild(CSS3Drenderer.domElement);

    const logo = document.createElement("img");
    logo.src = "/logo.png";
    logo.alt = "logo";

    const gamename = document.createElement("h1");
    gamename.innerText = "Simulation Game";
    gamename.classList.add("text-cyan-400");

    const playBtn = document.createElement("button");
    let playClicked = false;
    playBtn.classList.add("text-cyan-400");
    playBtn.innerText = "Play";
    playBtn.addEventListener("click", () => {
      playClicked = !playClicked;
      setTimeout(() => {
        window.location.href = "/Game";
      }, 700);
    });

    const sourceBtn = document.createElement("button");
    let sourceClicked = false;
    sourceBtn.classList.add("text-cyan-400");
    sourceBtn.innerText = "Sources";
    sourceBtn.addEventListener("click", () => {
      sourceClicked = !sourceClicked;
      setTimeout(() => {
        window.location.href = "/Sources";
      }, 700);
    });

    const logoObj = new CSS3DObject(logo);
    const gamenameObj = new CSS3DObject(gamename);
    const playBtnObj = new CSS3DObject(playBtn);
    const sourceBtnObj = new CSS3DObject(sourceBtn);
    logoObj.position.set(0, 200, -500);
    gamenameObj.position.set(0, 50, -300);
    playBtnObj.position.set(0, 40, -500);
    sourceBtnObj.position.set(0, 0, -500);

    scene.add(logoObj);
    scene.add(gamenameObj);
    scene.add(playBtnObj);
    scene.add(sourceBtnObj);

    const fbxLoader = new FBXLoader();
    const ambientLight = new THREE.AmbientLight(0xffffff);
    scene.add(ambientLight);

    let gundamModel: THREE.Group<THREE.Object3DEventMap>;
    let mixer: THREE.AnimationMixer;
    let action: THREE.AnimationAction;

    fbxLoader.load("rx-78/test/source/model.fbx", (fbxScene) => {
      gundamModel = fbxScene;
      console.log(gundamModel);
      mixer = new THREE.AnimationMixer(gundamModel);
      const randomNum = Math.floor(Math.random() * 2) + 3;
      console.log(randomNum);
      action = mixer.clipAction(gundamModel.animations[randomNum]);
      //   console.log(gundamModel.animations);
      //   console.log(gundamModel.scene.animations);
      action.play();
      gundamModel.position.set(0, -100, -170);
      //   gundamModel.scene.rotateX((3 * Math.PI) / 2);
      scene.add(gundamModel);
    });

    const spaceTexture = new THREE.CubeTextureLoader()
      .setPath("/textures/spaceMap/")
      .load([
        "right.png",
        "left.png",
        "top.png",
        "bottom.png",
        "front.png",
        "back.png",
      ]);
    scene.background = spaceTexture;

    window.addEventListener("resize", () => {
      const newWidth = window.innerWidth;
      const newHeight = window.innerHeight;
      camera.aspect = newWidth / newHeight;
      camera.updateProjectionMatrix();
      webglRenderer.setSize(newWidth, newHeight);
      CSS3Drenderer.setSize(newWidth, newHeight);
    });

    function optionSelected(button: CSS3DObject) {
      button.position.x += 8;
    }

    function animate() {
      if (playClicked) optionSelected(playBtnObj);
      if (sourceClicked) optionSelected(sourceBtnObj);
      if (mixer) {
        const time = performance.now();
        const delta = (time - prevTime) / 1000;
        mixer.update(delta);
        prevTime = time;
      }

      webglRenderer.render(scene, camera);
      CSS3Drenderer.render(scene, camera);
      requestAnimationFrame(animate);
    }
    animate();
  }
</script>
