<script lang="ts">
  import * as THREE from "three";
  import { browser } from "$app/environment";
  import { PointerLockControls } from "three/examples/jsm/controls/PointerLockControls.js";
  import {
    GLTFLoader,
    type GLTF,
  } from "three/examples/jsm/loaders/GLTFLoader.js";

  interface previousSMDeleted {
    model: THREE.Mesh | null;
    deleted: boolean;
  }

  interface beams {
    model: THREE.Mesh | null;
    direction?: THREE.Vector3;
  }

  interface targetCoords {
    model: THREE.Mesh | null;
    x: number;
    y: number;
    z: number;
  }

  let shots = 16;

  let killCounter = 0;
  let rifleOrSaber = false; // true will mean rifle and false will mean saber

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
    let previousSelectedModel: previousSMDeleted = {
      model: null,
      deleted: true,
    };
    let prevTargetCoords: targetCoords[] = [];
    let moving: boolean = false;
    let fire: boolean = false;

    // target position for mouse
    let targetx: number | null = 0;
    let targety: number | null = 0;
    let targetz: number | null = 15;

    const controls = new PointerLockControls(camera, document.body);
    const blocker = document.getElementById("blocker");
    const victoryScreen = document.getElementById("victory-screen");

    const glftLoader = new GLTFLoader();
    const ambientLight = new THREE.AmbientLight(0xffffff);
    scene.add(ambientLight);

    const moveSpeed = 0.7; // Adjust this to control the movement speed
    const geometry = new THREE.SphereGeometry();
    const fireTexture = new THREE.TextureLoader().load("fire.jpg");
    const fireMaterial = new THREE.MeshBasicMaterial({ map: fireTexture });
    let explosionList: THREE.Mesh[] = [];

    let beamSaberModel: GLTF;
    glftLoader.load("lightsaber/scene.gltf", (gltfScene) => {
      beamSaberModel = gltfScene;
      beamSaberModel.scene.scale.set(0.05, 0.05, 0.05);
      beamSaberModel.scene.rotation.x -= 0;
      beamSaberModel.scene.rotation.y += 0.9;
      beamSaberModel.scene.rotation.z -= 0.0;
      camera.add(gltfScene.scene);
    });
    let beamRifle: GLTF;
    glftLoader.load("rifle/scene.gltf", (gltfScene) => {
      beamRifle = gltfScene;
      beamRifle.scene.rotation.x += 0.5;
      beamRifle.scene.rotation.y += 0.5;
      beamRifle.scene.rotation.z += 0.3;
      camera.add(gltfScene.scene);
    });
    const beamGeometry = new THREE.CylinderGeometry(0.1, 0.1, 10, 32);
    const beamMaterial = new THREE.MeshBasicMaterial({ color: 0xde73ff });

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
        zaku.scene.position.x = Math.random() * 100 - 50;
        zaku.scene.position.y = Math.random() * 100 - 50;
        zaku.scene.position.z = Math.random() * 100 - 50;
        parentScenes[gltfScene.scene.uuid] = gltfScene.scene;
        scene.add(gltfScene.scene);
        gltfScene.scene.traverse((child) => {
          if (child instanceof THREE.Mesh) enemies.push(child);
        });
      });
    }

    function checkVictory() {
      if (killCounter % 10 === 0) {
        victoryScreen?.classList.remove("hidden");
        setTimeout(() => {
          victoryScreen?.classList.add("hidden");
        }, 5000);
      }
    }

    const beams: beams[] = [];
    function fireBeam() {
      fire = true;
      shots--;
      const beam = new THREE.Mesh(beamGeometry, beamMaterial);
      scene.add(beam);
      beam.position.x = camera.position.x;
      beam.position.y = camera.position.y;
      beam.position.z = camera.position.z;
      const cameraPosition = camera.position.clone();

      if (targetx && targety && targetz) {
        const newDirection = new THREE.Vector3(
          targetx - cameraPosition.x,
          targety - cameraPosition.y,
          targetz - cameraPosition.z,
        );
        newDirection.normalize();
        const beamObj = {
          model: beam,
          direction: newDirection,
        };
        beams.push(beamObj);
      }
    }

    function updateBeams() {
      const beamSpeed = 0.7;
      for (let i = 0; i < beams.length; i++) {
        const currentBeam = beams[i];
        if (currentBeam.model && currentBeam.direction) {
          currentBeam.model?.position.add(
            currentBeam.direction.clone().multiplyScalar(beamSpeed),
          );
          if (prevTargetCoords) {
            for (let i = 0; i < prevTargetCoords.length; i++) {
              const currentCoord = prevTargetCoords[i];
              if (
                currentBeam.model.position.distanceTo(
                  new THREE.Vector3(
                    currentCoord.x,
                    currentCoord.y,
                    currentCoord.z,
                  ),
                ) < 1
              ) {
                if (currentCoord.model) {
                  const parentSceneId =
                    currentCoord.model.parent?.parent?.parent?.parent?.uuid;
                  if (parentSceneId !== undefined) {
                    const parentScene = parentScenes[parentSceneId];
                    if (parentScene) scene.remove(parentScene);
                    console.log("removed");
                  }
                }
                const sphere = new THREE.Mesh(geometry, fireMaterial);
                sphere.scale.set(5.5, 5.5, 5.5);
                sphere.position.set(
                  currentCoord.x,
                  currentCoord.y,
                  currentCoord.z,
                );
                explosionList.push(sphere);
                scene.add(sphere);
                setTimeout(() => {
                  scene.remove(sphere);
                }, 2000);
                targetx = null;
                targety = null;
                targetz = null;
                killCounter++;
                // checkVictory();
              }
            }
          }
        }
      }

      // const gunbarrelposition = new THREE.Vector3(
      //   beamRifle.scene.position.x,
      //   beamRifle.scene.position.y,
      //   beamRifle.scene.position.z,
      // );
    }

    // Function to handle mouse click
    function onMouseClick(event: MouseEvent) {
      const worldPosition = new THREE.Vector3();

      if (selectedModel) {
        selectedModel.updateMatrixWorld();
        worldPosition.setFromMatrixPosition(selectedModel.matrixWorld);
        // Dash to the selected model
        targetx = worldPosition.x;
        targety = worldPosition.y + 5;
        targetz = worldPosition.z;

        const targetObj = {
          x: targetx,
          y: targety,
          z: targetz,
          model: selectedModel,
        };

        prevTargetCoords.push(targetObj);
      } else {
        const clickedPosition = new THREE.Vector3();
        raycaster.ray.at(2000, clickedPosition); // Change 20 to your desired distance
        targetx = clickedPosition.x;
        targety = clickedPosition.y;
        targetz = clickedPosition.z;
      }

      if (rifleOrSaber === false && selectedModel) moving = true;
      if (rifleOrSaber === true && shots > 0) fireBeam();
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

    function toggleWeapon() {
      rifleOrSaber = !rifleOrSaber;
    }

    function addDocClickEvt() {
      document?.addEventListener("click", () => {
        controls.lock();
      });
    }

    document.addEventListener("keydown", onKeyDown);
    document.addEventListener("keyup", onKeyUp);
    document.addEventListener("wheel", toggleWeapon);

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

    async function animate() {
      // animation function here animation function here animation function here animation function here animation function here animation function here
      explosionList.forEach((explosion) => {
        explosion.rotation.x += 0.01;
        explosion.rotation.y += 0.01;
      });
      // sphere.rotation.x += 0.01;
      // sphere.rotation.y += 0.01;
      for (let i = 0; i < beams.length; i++) {
        beams[i].model?.lookAt(camera.position);
        beams[i].model?.rotateX((3 * Math.PI) / 2);
      }

      for (let i = 0; i < enemies.length; i++) {
        enemies[i].lookAt(camera.position);
        enemies[i].rotateX((3 * Math.PI) / 2);
      }

      updateBeams();

      // Update the raycaster
      raycaster.setFromCamera(pointer, camera);

      const intersects: THREE.Intersection[] = raycaster.intersectObjects(
        scene.children,
        true,
      );
      if (intersects.length > 0) {
        selectedModel = intersects[0].object as THREE.Mesh;
        if (fire) {
          previousSelectedModel.model = selectedModel;
          fire = false;
        }
        if (moving && previousSelectedModel.deleted) {
          previousSelectedModel.model = selectedModel;
          previousSelectedModel.deleted = false;
        }
        // previousSelectedMoel is causing the models to not be removed, need to fix bug
      } else {
        // No intersection, clear the selected model
        selectedModel = null;
      }

      if (beamRifle && beamSaberModel && rifleOrSaber === true) {
        beamRifle.scene.position.x = 1.8;
        beamRifle.scene.position.y = -2.3;
        beamRifle.scene.position.z = -1;

        beamSaberModel.scene.position.x = 0.3;
        beamSaberModel.scene.position.y = -0.7;
        beamSaberModel.scene.position.z = 1;
      } else if (beamRifle && beamSaberModel && rifleOrSaber === false) {
        beamSaberModel.scene.position.x = 0.3;
        beamSaberModel.scene.position.y = -0.7;
        beamSaberModel.scene.position.z = -1;

        beamRifle.scene.position.x = 1.8;
        beamRifle.scene.position.y = -2.3;
        beamRifle.scene.position.z = 1;
      }

      if (beamRifle && rifleOrSaber === true) {
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
            ) < 7
          ) {
            moving = false; // Stop moving once we're close enough
            if (previousSelectedModel.model) {
              const parentSceneId =
                previousSelectedModel.model.parent?.parent?.parent?.parent
                  ?.uuid;
              if (parentSceneId !== undefined) {
                const parentScene = parentScenes[parentSceneId];
                if (parentScene) {
                  scene.remove(parentScene);
                  previousSelectedModel.deleted = true;
                  console.log("removed");
                }
              }
              const sphere = new THREE.Mesh(geometry, fireMaterial);
              sphere.scale.set(5.5, 5.5, 5.5);
              sphere.position.set(targetx, targety, targetz);
              explosionList.push(sphere);
              scene.add(sphere);
              setTimeout(() => {
                scene.remove(sphere);
              }, 2000);
              targetx = null;
              targety = null;
              targetz = null;
              killCounter++;
              // checkVictory();
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
    camera.position.z = 100;
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
  <div class="bg-white w-48 h-48 fixed hidden" id="victory-screen">
    <h1>All Enemies have been defeated.</h1>
  </div>
  <img src="crosshair.png" alt="crosshair" class="fixed" />
</div>
<div
  class="border-2 border-neutral-400 w-48 h-48 fixed text-neutral-400"
  id="gui"
>
  <p class="text-xl">
    Weapon Equipped: {#if rifleOrSaber}
      <div class="flex flex-col">
        <span>Beam Rifle</span>
        <span class="text-xl">Shots: {shots}/16</span>
      </div>
    {/if}
    {#if !rifleOrSaber}
      Beam Saber{/if}
  </p>
</div>
