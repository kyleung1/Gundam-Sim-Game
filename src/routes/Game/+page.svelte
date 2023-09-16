<script lang="ts">
  import * as THREE from "three";
  import { browser } from "$app/environment";
  import { FirstPersonControls } from "three/examples/jsm/controls/FirstPersonControls.js";
  import { PointerLockControls } from "three/examples/jsm/controls/PointerLockControls.js";
  import { FlyControls } from "three/examples/jsm/controls/FlyControls.js";
  import { onMount, onDestroy } from "svelte";

  if (browser) {
    const scene = new THREE.Scene();
    const camera = new THREE.PerspectiveCamera(
      75,
      window.innerWidth / window.innerHeight / 2,
      0.1,
      1000,
    );
    const renderer = new THREE.WebGLRenderer();
    renderer.setPixelRatio(window.devicePixelRatio);
    renderer.setSize(window.innerWidth, window.innerHeight);
    document.body.appendChild(renderer.domElement);
    document.body.classList.add("crosshair-cursor");

    let moveForward = false;
    let moveBackward = false;
    let moveLeft = false;
    let moveRight = false;
    let canJump = false;

    let prevTime = performance.now();
    const velocity = new THREE.Vector3();
    const direction = new THREE.Vector3();
    const vertex = new THREE.Vector3();
    const color = new THREE.Color();

    const raycaster = new THREE.Raycaster();
    const pointer = new THREE.Vector2();
    let selectedModel: THREE.Mesh | null = null;

    // target position for mouse
    let targetx = 0;
    let targety = 0;
    let targetz = 15;

    const controls = new PointerLockControls(camera, document.body);
    const blocker = document.getElementById("blocker");

    const enemies: THREE.Mesh[] = [];
    function addEnemy() {
      const geometry = new THREE.BoxGeometry();
      const material = new THREE.MeshBasicMaterial({ color: 0x00ff00 });
      const cube = new THREE.Mesh(geometry, material);
      cube.position.x = Math.random() * 16 - 8;
      cube.position.y = Math.random() * 16 - 8;
      cube.position.z = Math.random() * 10 - 5;
      scene.add(cube);
      enemies.push(cube);
    }

    // Function to handle mouse click
    function onMouseClick(event: MouseEvent) {
      if (selectedModel) {
        // Dash to the selected model
        const targetPosition = selectedModel.position.clone();
        targetx = targetPosition.x;
        targety = targetPosition.y;
        targetz = targetPosition.z;
        console.log("model selected");
      } else {
        console.log("no model selected");
      }
    }

    function onKeyDown(event: KeyboardEvent) {
      switch (event.code) {
        case "ArrowUp":
        case "KeyW":
          moveForward = true;
          break;
        case "ArrowLeft":
        case "KeyA":
          moveLeft = true;
          break;

        case "ArrowDown":
        case "KeyS":
          moveBackward = true;
          break;

        case "ArrowRight":
        case "KeyD":
          moveRight = true;
          break;
        case "Space":
          if (canJump === true) velocity.y += 350;
          canJump = false;
          break;
      }
    }

    function onKeyUp(event: KeyboardEvent) {
      switch (event.code) {
        case "ArrowUp":
        case "KeyW":
          moveForward = false;
          break;

        case "ArrowLeft":
        case "KeyA":
          moveLeft = false;
          break;

        case "ArrowDown":
        case "KeyS":
          moveBackward = false;
          break;

        case "ArrowRight":
        case "KeyD":
          moveRight = false;
          break;
      }
    }

    function addEventBlocker() {
      blocker?.addEventListener("click", () => {
        controls.lock();
      });
    }

    document.addEventListener("keydown", onKeyDown);
    document.addEventListener("keyup", onKeyUp);

    controls.addEventListener("lock", () => {
      console.log("it is locked");
      blocker?.classList.add("hidden");
    });

    addEventBlocker();

    window.addEventListener("click", onMouseClick);

    controls.addEventListener("unlock", () => {
      blocker?.classList.remove("hidden");
      addEventBlocker();
    });

    window.addEventListener("resize", () => {
      const newWidth = window.innerWidth;
      const newHeight = window.innerHeight;
      camera.aspect = newWidth / newHeight;
      camera.updateProjectionMatrix();
      renderer.setSize(newWidth, newHeight);
    });

    const cameraOrientation = new THREE.Euler(0, 0, 0, "YXZ");
    const sensitivity = 0.001;
    window.addEventListener("mousemove", (event: MouseEvent) => {
      // adjust pointer
      pointer.x = (event.clientX / window.innerWidth) * 2 - 1;
      pointer.y = -(event.clientY / window.innerHeight) * 2 + 1;

      // // adjust camera orientation
      // cameraOrientation.y -= event.movementX * sensitivity;
      // cameraOrientation.x -= event.movementY * sensitivity;

      // // Limit the camera's vertical rotation
      // cameraOrientation.x = Math.max(
      //   -Math.PI / 2,
      //   Math.min(Math.PI / 2, cameraOrientation.x),
      // );

      // const controls = new FirstPersonControls(camera, renderer.domElement);
      // controls.lookSpeed = sensitivity;
      // controls.movementSpeed = 5;
      // controls.lookVertical = false;
    });

    scene.add(controls.getObject());

    // const controls = new FirstPersonControls(camera, renderer.domElement);
    // controls.lookSpeed = sensitivity;
    // controls.movementSpeed = 0.2;

    function animate() {
      // Update the raycaster
      raycaster.setFromCamera(pointer, camera);

      const intersects: THREE.Intersection[] =
        raycaster.intersectObjects(enemies);
      if (intersects.length > 0) {
        // highlights the selected enemy
        if (selectedModel) {
          const material = selectedModel.material as THREE.MeshBasicMaterial;
          material.color.set(0x00ff00);
        }
        selectedModel = intersects[0].object as THREE.Mesh;
        const material = selectedModel.material as THREE.MeshBasicMaterial;
        material.color.set(0xff0000);
      } else {
        // No intersection, clear the selected model
        if (selectedModel) {
          const material = selectedModel.material as THREE.MeshBasicMaterial;
          material.color.set(0x00ff00);
        }
        selectedModel = null;
      }

      // update camera rotation based on orientation
      // camera.setRotationFromEuler(cameraOrientation);

      // // travels to target when clicked
      // const moveSpeed = 0.2;

      // if (
      //   camera.position.x !== targetx ||
      //   camera.position.y !== targety ||
      //   camera.position.z !== targetz
      // ) {
      //   const direction = new THREE.Vector3(
      //     targetx - camera.position.x,
      //     targety - camera.position.y,
      //     targetz - camera.position.z,
      //   );
      //   direction.normalize();
      //   camera.position.add(direction.multiplyScalar(moveSpeed));
      // }

      // controls.update(1);

      renderer.render(scene, camera);
      requestAnimationFrame(animate);
    }

    // set the scene
    camera.position.x = 0;
    camera.position.y = 0;
    camera.position.z = 15;
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

    for (let i = 0; i < 10; i++) {
      addEnemy();
    }

    animate();
  }

  // interface current {
  //   leftButton: boolean;
  //   rightButton: boolean;
  //   mouseX: number;
  //   mouseY: number;
  // }

  // class InputController {
  //   constructor() {
  //     this.initialize();
  //   }
  //   public current = {
  //     leftButton: false,
  //     rightButton: false,
  //     mouseX: 0,
  //     mousey: 0,
  //   };

  //   public previous: null | current = null;
  //   public keys: { [keyCode: number]: boolean } = {};
  //   public previousKeys = {};

  //   initialize() {
  //     document.addEventListener(
  //       "mousedown",
  //       (e) => this.onMouseDown(e),
  //       false,
  //     );
  //     document.addEventListener("mouseup", (e) => this.onMouseUp(e), false);
  //     document.addEventListener(
  //       "mousemove",
  //       (e) => this.onMouseMove(e),
  //       false,
  //     );
  //     document.addEventListener("keydown", (e) => this.onKeyDown(e), false);
  //     document.addEventListener("keyup", (e) => this.onKeyUp(e), false);
  //   }

  //   onMouseDown(e: MouseEvent) {
  //     switch (e.button) {
  //       case 0: {
  //         this.current.leftButton = true;
  //         break;
  //       }
  //       case 2: {
  //         this.current.rightButton = true;
  //         break;
  //       }
  //     }
  //   }
  //   onMouseUp(e: MouseEvent) {
  //     switch (e.button) {
  //       case 0: {
  //         this.current.leftButton = false;
  //         break;
  //       }
  //       case 2: {
  //         this.current.rightButton = false;
  //         break;
  //       }
  //     }
  //   }
  //   onMouseMove(e: MouseEvent) {
  //     this.current.mouseX = e.pageX - window.innerWidth / 2;
  //     this.current.mousey = e.pageY - window.innerHeight / 2;

  //     if (this.previous === null) {
  //       this.previous = { ...this.current };
  //     }

  //     this.current.mouseXDelta = this.current.mouseX - this.previous.mouseX;
  //     this.current.mouseYDelta = this.current.mouseY - this.previous.mouseY;
  //   }
  //   onKeyDown(e: KeyboardEvent) {
  //     this.keys[e.keyCode] = true;
  //   }
  //   onKeyUp(e: KeyboardEvent) {
  //     this.keys[e.keyCode] = false;
  //   }

  //   update() {}
  // }

  // class FirstPersonCamera {
  //   camera: THREE.PerspectiveCamera;
  //   input: InputController;
  //   rotation: THREE.Quaternion;
  //   translation: THREE.Vector3;
  //   phi: number;
  //   theta: number;
  //   constructor(camera: THREE.PerspectiveCamera) {
  //     this.camera = camera;
  //     this.input = new InputController();
  //     this.rotation = new THREE.Quaternion();
  //     this.translation = new THREE.Vector3();
  //     this.phi = 0;
  //     this.theta = 0;
  //   }

  //   update(timeElapsedS: number) {
  //     this.updateRotation(timeElapsedS);
  //   }

  //   updateRotation(timeElapsedS: number) {
  //     const xh = this.input.current.mouseX;
  //   }
  // }

  // const fpsCamera = new FirstPersonCamera(camera);
</script>

<!-- {#if showBlocker} -->
<div class="bg-white w-48 h-48 fixed top-1/2 left-1/2" id="blocker">
  <h1>Click any where to start</h1>
</div>
<!-- {/if} -->
