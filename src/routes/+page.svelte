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
      100,
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
    gamename.classList.add(
      "text-cyan-400",
      "css3dText",
      "full-width",
      "text-center",
    );

    const playBtn = document.createElement("button");
    let playClicked = false;
    playBtn.classList.add("menu-buttons", "css3dText");
    playBtn.innerText = "Play";
    playBtn.addEventListener("click", () => {
      playClicked = !playClicked;
      setTimeout(() => {
        window.location.href = "/Game";
      }, 700);
    });

    const controlsBtn = document.createElement("button");
    let controlsClicked = false;
    controlsBtn.classList.add("menu-buttons", "css3dText");
    controlsBtn.innerText = "Controls";
    controlsBtn.addEventListener("click", () => {
      controlsClicked = !controlsClicked;
      setTimeout(() => {
        window.location.href = "/Controls";
      }, 700);
    });

    const creditBtn = document.createElement("button");
    let creditsClicked = false;
    creditBtn.classList.add("menu-buttons", "css3dText");
    creditBtn.innerText = "Credits";
    creditBtn.addEventListener("click", () => {
      creditsClicked = !creditsClicked;
      setTimeout(() => {
        window.location.href = "/Credits";
      }, 700);
    });

    const aboutBtn = document.createElement("button");
    let aboutClicked = false;
    aboutBtn.classList.add("menu-buttons", "css3dText");
    aboutBtn.innerText = "About";
    aboutBtn.addEventListener("click", () => {
      aboutClicked = !aboutClicked;
      setTimeout(() => {
        window.location.href = "/About";
      }, 700);
    });

    const logoObj = new CSS3DObject(logo);
    const gamenameObj = new CSS3DObject(gamename);
    const playBtnObj = new CSS3DObject(playBtn);
    const controlsBtnObj = new CSS3DObject(controlsBtn);
    const creditBtnObj = new CSS3DObject(creditBtn);
    const aboutBtnObj = new CSS3DObject(aboutBtn);
    logoObj.position.set(0, 200, -300);
    gamenameObj.scale.set(0.15, 0.15, 0.15);
    gamenameObj.position.set(0, 75, -300);
    playBtnObj.position.set(0, 25, -300);
    playBtnObj.scale.set(0.1, 0.1, 0.1);
    controlsBtnObj.position.set(0, -25, -300);
    controlsBtnObj.scale.set(0.1, 0.1, 0.1);
    creditBtnObj.position.set(0, -75, -300);
    creditBtnObj.scale.set(0.1, 0.1, 0.1);
    aboutBtnObj.position.set(0, -125, -300);
    aboutBtnObj.scale.set(0.1, 0.1, 0.1);

    scene.add(logoObj);
    scene.add(gamenameObj);
    scene.add(playBtnObj);
    scene.add(controlsBtnObj);
    scene.add(creditBtnObj);
    scene.add(aboutBtnObj);

    const fbxLoader = new FBXLoader();
    const ambientLight = new THREE.AmbientLight(0xffffff);
    scene.add(ambientLight);

    let gundamModel: THREE.Group<THREE.Object3DEventMap>;
    let mixer: THREE.AnimationMixer;
    let action: THREE.AnimationAction;

    fbxLoader.load("rx-78/test/source/model.fbx", (fbxScene) => {
      gundamModel = fbxScene;
      mixer = new THREE.AnimationMixer(gundamModel);
      const randomNum = Math.floor(Math.random() * 2) + 3;
      action = mixer.clipAction(gundamModel.animations[randomNum]);
      action.play();
      gundamModel.position.set(0, -100, -155);
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

    let oldx = 0;
    let oldy = 0;
    window.onmousemove = function (event) {
      let changeX = event.x - oldx;
      let changeY = event.y - oldy;
      camera.position.x += changeX / 100;
      camera.position.y -= changeY / 100;
      oldx = event.x;
      oldy = event.y;
    };

    function optionSelected(button: CSS3DObject) {
      const originalPosition = button.position.clone();
      if (button.position.x < 1000) button.position.x += 10;
      setTimeout(() => {
        button.position.set(
          originalPosition.x,
          originalPosition.y,
          originalPosition.z,
        );
        button.position.x += 0;
      }, 800);
    }

    function animate() {
      if (playClicked) optionSelected(playBtnObj);
      if (controlsClicked) optionSelected(controlsBtnObj);
      if (creditsClicked) optionSelected(creditBtnObj);
      if (aboutClicked) optionSelected(aboutBtnObj);
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
