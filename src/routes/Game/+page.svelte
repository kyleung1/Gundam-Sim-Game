<script lang="ts">
  import * as THREE from "three";
  import { browser } from "$app/environment";
  import { FirstPersonControls } from "three/examples/jsm/controls/FirstPersonControls.js";
  import { PointerLockControls } from "three/examples/jsm/controls/PointerLockControls.js";
  import { FlyControls } from "three/examples/jsm/controls/FlyControls.js";

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

    class InputController {
      constructor() {
        this.initialize();
      }
      private current = {
        leftButton: false,
        rightButton: false,
        mouseX: 0,
        mousey: 0,
      };

      private previous = null;
      private keys = {};
      private previousKeys = {};

      initialize() {
        document.addEventListener(
          "mousedown",
          (e) => this.onMouseDown(e),
          false,
        );
        document.addEventListener("mouseup", (e) => this.onMouseUp(e), false);
        document.addEventListener(
          "mousemove",
          (e) => this.onMouseMove(e),
          false,
        );
        document.addEventListener("keydown", (e) => this.onKeyDown(e), false);
        document.addEventListener("keyup", (e) => this.onKeyUp(e), false);
      }

      onMouseDown(e: MouseEvent) {}
      onMouseUp(e: MouseEvent) {}
      onMouseMove(e: MouseEvent) {}
      onKeyDown(e: KeyboardEvent) {}
      onKeyUp(e: KeyboardEvent) {}
    }

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

    // target position for mouse
    let targetx = 0;
    let targety = 0;
    let targetz = 15;

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

    // with threejs can only add event listeners to the dom not individual elements, need raycaster
    // https://threejs.org/docs/#api/en/core/Raycaster

    const raycaster = new THREE.Raycaster();
    const pointer = new THREE.Vector2();
    let selectedModel: THREE.Mesh | null = null;

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

    // const controls = new FirstPersonControls(camera, renderer.domElement);
    // controls.lookSpeed = sensitivity;
    // controls.movementSpeed = 0.2;

    const controls = new FlyControls(camera, document.body);
    // document.body.addEventListener("click", () => {
    //   controls.lock();
    // });
    // scene.add(controls.getObject());

    window.addEventListener("click", onMouseClick);

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
      camera.setRotationFromEuler(cameraOrientation);

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

      controls.update(1);

      renderer.render(scene, camera);
      requestAnimationFrame(animate);
    }

    // add 10 enemies
    for (let i = 0; i < 10; i++) {
      addEnemy();
    }

    animate();
  }
</script>
