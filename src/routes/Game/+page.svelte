<script lang="ts">
  import * as THREE from "three";
  import { browser } from "$app/environment";
  import { PointerLockControls } from "three/examples/jsm/controls/PointerLockControls.js";
  import {
    GLTFLoader,
    type GLTF,
  } from "three/examples/jsm/loaders/GLTFLoader.js";

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
    let moveUp = false;
    let moveDown = false;

    let prevTime = performance.now();
    const velocity = new THREE.Vector3();
    const direction = new THREE.Vector3();

    const raycaster = new THREE.Raycaster();
    const pointer = new THREE.Vector2();
    let selectedModel: THREE.Mesh | null = null;
    let moving: boolean = false;

    // target position for mouse
    let targetx: number | null = 0;
    let targety: number | null = 0;
    let targetz: number | null = 15;

    const controls = new PointerLockControls(camera, document.body);
    const blocker = document.getElementById("blocker");

    const glftLoader = new GLTFLoader();
    const ambientLight = new THREE.AmbientLight(0xffffff);
    scene.add(ambientLight);

    const enemies: THREE.Mesh[] = [];
    interface GroupObject {
      [key: string]: THREE.Group<THREE.Object3DEventMap> | undefined;
    }
    let parentScenes: GroupObject = {};
    function addEnemy() {
      let zaku: GLTF;
      glftLoader.load("zaku/scene.gltf", (gltfScene) => {
        zaku = gltfScene;
        zaku.scene.scale.set(1, 1, 1);
        zaku.scene.position.x = Math.random() * 10 - 5;
        zaku.scene.position.y = Math.random() * 10 - 5;
        zaku.scene.position.z = Math.random() * 10 - 5;
        parentScenes[gltfScene.scene.uuid] = gltfScene.scene;
        scene.add(gltfScene.scene);
        gltfScene.scene.traverse((child) => {
          if (child instanceof THREE.Mesh) enemies.push(child);
        });
      });
      // const geometry = new THREE.BoxGeometry();
      // const material = new THREE.MeshBasicMaterial({ color: 0x00ff00 });
      // const cube = new THREE.Mesh(geometry, material);
      // cube.position.x = Math.random() * 10 - 5;
      // cube.position.y = Math.random() * 10 - 5;
      // cube.position.z = Math.random() * 10 - 5;
      // scene.add(cube);
      // enemies.push(cube);
    }

    // Function to handle mouse click
    function onMouseClick(event: MouseEvent) {
      const worldPosition = new THREE.Vector3();
      moving = true;
      if (selectedModel) {
        selectedModel.updateMatrixWorld();
        worldPosition.setFromMatrixPosition(selectedModel.matrixWorld);
        // Dash to the selected model
        const targetPosition = selectedModel.position.clone();
        targetx = worldPosition.x;
        targety = worldPosition.y + 5;
        targetz = worldPosition.z;
        console.log("model selected", targetx, targety, targetz);
      } else {
        console.log("no model selected", targetx, targety, targetz);
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
          moveUp = true;
          break;
        case "ShiftLeft":
          moveDown = true;
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
        case "Space":
          moveUp = false;
          break;
        case "ShiftLeft":
          moveDown = false;
          break;
      }
    }

    function addDocClickEvt() {
      document?.addEventListener("click", () => {
        controls.lock();
      });
    }

    document.addEventListener("keydown", onKeyDown);
    document.addEventListener("keyup", onKeyUp);

    controls.addEventListener("lock", () => {
      blocker?.classList.add("hidden");
    });

    addDocClickEvt();

    window.addEventListener("click", onMouseClick);

    controls.addEventListener("unlock", () => {
      blocker?.classList.remove("hidden");
      addDocClickEvt();
    });

    window.addEventListener("resize", () => {
      const newWidth = window.innerWidth;
      const newHeight = window.innerHeight;
      camera.aspect = newWidth / newHeight;
      camera.updateProjectionMatrix();
      renderer.setSize(newWidth, newHeight);
    });

    window.addEventListener("mousemove", (event: MouseEvent) => {
      // adjust pointer
      // wherever the pointer is
      // pointer.x = (event.clientX / window.innerWidth) * 2 - 1;
      // pointer.y = -(event.clientY / window.innerHeight) * 2 + 1;
      // at the center of the window
      pointer.x = 0;
      pointer.y = 0;
    });

    scene.add(controls.getObject());

    const moveSpeed = 0.7; // Adjust this to control the movement speed
    const tolerance = 7; // Adjust this to control how close you want to get to the target
    const geometry = new THREE.SphereGeometry();
    const fireTexture = new THREE.TextureLoader().load("fire.jpg");
    const material = new THREE.MeshBasicMaterial({ map: fireTexture });
    const sphere = new THREE.Mesh(geometry, material);
    sphere.scale.set(5.5, 5.5, 5.5);
    let beamSaberModel: GLTF;
    glftLoader.load("lightsaber/scene.gltf", (gltfScene) => {
      beamSaberModel = gltfScene;
      beamSaberModel.scene.scale.set(0.05, 0.05, 0.05);
      beamSaberModel.scene.rotation.x -= 0;
      beamSaberModel.scene.rotation.y += 0.9;
      beamSaberModel.scene.rotation.z -= 0.0;
      camera.add(gltfScene.scene);
    });

    async function animate() {
      sphere.rotation.x += 0.01;
      sphere.rotation.y += 0.01;
      // Update the raycaster
      raycaster.setFromCamera(pointer, camera);

      const intersects: THREE.Intersection[] =
        raycaster.intersectObjects(enemies);
      if (intersects.length > 0) {
        selectedModel = intersects[0].object as THREE.Mesh;
      } else {
        // No intersection, clear the selected model
        selectedModel = null;
      }

      if (beamSaberModel) {
        beamSaberModel.scene.position.x = 0.3;
        beamSaberModel.scene.position.y = -0.7;
        beamSaberModel.scene.position.z = -1;
      }

      if (moving) {
        const cameraPosition = camera.position;

        // Calculate the direction from camera position to target position
        if (targetx && targety && targetz) {
          const newDirection = new THREE.Vector3(
            targetx - cameraPosition.x,
            targety - cameraPosition.y,
            targetz - cameraPosition.z,
          );
          newDirection.normalize();

          // Move the camera and controls towards the target
          cameraPosition.add(newDirection.clone().multiplyScalar(moveSpeed));

          // Check if we've reached the target
          if (
            cameraPosition.distanceTo(
              new THREE.Vector3(targetx, targety, targetz),
            ) < tolerance
          ) {
            moving = false; // Stop moving once we're close enough
            if (selectedModel) {
              console.log(selectedModel);
              const parentSceneId =
                selectedModel.parent?.parent?.parent?.parent?.uuid;
              if (parentSceneId !== undefined) {
                const parentScene = parentScenes[parentSceneId];
                if (parentScene) scene.remove(parentScene);
              }
              console.log(intersects);
              selectedModel = null;
              sphere.position.set(targetx, targety, targetz);
              scene.add(sphere);
              setTimeout(() => {
                scene.remove(sphere);
              }, 2000);
              targetx = null;
              targety = null;
              targetz = null;
            }
          }
        }
      }

      const time = performance.now();
      const delta = (time - prevTime) / 1000;
      if (controls.isLocked === true) {
        velocity.x -= velocity.x * 10.0 * delta;
        velocity.z -= velocity.z * 10.0 * delta;
        velocity.y -= velocity.y * 10.0 * delta;

        direction.z = Number(moveForward) - Number(moveBackward);
        direction.x = Number(moveRight) - Number(moveLeft);
        direction.y = Number(moveUp) - Number(moveDown);
        direction.normalize();

        if (moveForward || moveBackward) {
          velocity.z -= direction.z * 400.0 * delta;
        } else if (moveLeft || moveRight) {
          velocity.x -= direction.x * 400.0 * delta;
        } else if (moveUp || moveDown) {
          velocity.y -= direction.y * 400.0 * delta;
        }

        controls.moveRight(-velocity.x * delta);
        controls.moveForward(-velocity.z * delta);
        controls.getObject().position.y -= velocity.y * delta;
      }

      prevTime = time;

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

<div class="flex justify-center items-center h-screen absolute inset-0">
  <div class="bg-white w-48 h-48 fixed" id="blocker">
    <h1>Click any where to start</h1>
  </div>
  <img src="crosshair.png" alt="crosshair" class="fixed" />
</div>
