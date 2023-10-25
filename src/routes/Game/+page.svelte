<script lang="ts">
  import * as THREE from "three";
  import { browser } from "$app/environment";
  import { PointerLockControls } from "three/examples/jsm/controls/PointerLockControls.js";
  import {
    GLTFLoader,
    type GLTF,
  } from "three/examples/jsm/loaders/GLTFLoader.js";
  import {
    CSS3DRenderer,
    CSS3DObject,
    CSS3DSprite,
  } from "three/examples/jsm/renderers/CSS3DRenderer.js";

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

  let ammo = 16;

  let killCounter = 0;
  let rifleOrSaber = false; // true will mean rifle and false will mean saber

  let controlsLocked = false;

  if (browser) {
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
    let prevTargetCoords: targetCoords[] = []; // only needed to keep track of targets for the beams
    let moving: boolean = false;
    let meleeAnimation: boolean = false;
    let fire: boolean = false;

    // target position for mouse
    let targetx: number | null = null;
    let targety: number | null = null;
    let targetz: number | null = null;

    const controls = new PointerLockControls(camera, document.body);
    const blocker = document.getElementById("blocker");
    const victoryScreen = document.getElementById("victory-screen");
    const bossScreen = document.getElementById("boss-battle");
    const bossDialogue = document.getElementById("boss-dialogue");

    // css3d popup when hovering an enemy
    const highlightDiv = document.createElement("div");
    const highlight = new CSS3DObject(highlightDiv);
    highlight.scale.set(0.03, 0.03, 0.03);
    const points1 = [
      new THREE.Vector3(-2, -1, 0),
      new THREE.Vector3(-4, -1, 0),
      new THREE.Vector3(-4, 10, 0),
      new THREE.Vector3(-2, 10, 0),
    ];
    const points2 = [
      new THREE.Vector3(2, -1, 0),
      new THREE.Vector3(4, -1, 0),
      new THREE.Vector3(4, 10, 0),
      new THREE.Vector3(2, 10, 0),
    ];

    // Create geometry by extruding the shape
    const bracketGeometry1 = new THREE.BufferGeometry().setFromPoints(points1);
    const bracketGeometry2 = new THREE.BufferGeometry().setFromPoints(points2);
    const bracketMaterial = new THREE.MeshBasicMaterial({
      color: 0x22d3ee,
    });
    const bracket1 = new THREE.Line(bracketGeometry1, bracketMaterial);
    const bracket2 = new THREE.Line(bracketGeometry2, bracketMaterial);

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
      beamSaberModel.scene.scale.set(0.05, 0.15, 0.05);
      // beamSaberModel.scene.position.set(2, 0, 0);
      camera.add(gltfScene.scene);
    });
    let beamRifle: GLTF;
    glftLoader.load("rifle/scene.gltf", (gltfScene) => {
      beamRifle = gltfScene;
      // scene.add(gltfScene.scene);
    });
    const beamGeometry = new THREE.CylinderGeometry(0.1, 0.1, 10, 32);
    const beamMaterial = new THREE.MeshBasicMaterial({ color: 0xde73ff });
    const enemyBeamMaterial = new THREE.MeshBasicMaterial({ color: 0xffea00 });

    const enemies: GLTF[] = [];
    interface GroupObject {
      [key: string]: THREE.Group<THREE.Object3DEventMap> | undefined;
    }
    let parentScenes: GroupObject = {};
    function addEnemy() {
      let zaku: GLTF;
      glftLoader.load("zaku/scene.gltf", (gltfScene) => {
        zaku = gltfScene;
        zaku.scene.scale.set(1, 1, 1);
        zaku.scene.position.x = Math.random() * 300 - 150;
        zaku.scene.position.y = Math.random() * 300 - 150;
        zaku.scene.position.z = Math.random() * 300 - 150;
        parentScenes[gltfScene.scene.uuid] = gltfScene.scene;
        scene.add(gltfScene.scene);
        enemies.push(gltfScene);
        // gltfScene.scene.traverse((child) => {
        //   if (child instanceof THREE.Mesh) enemies.push(child);
        // });
      });
    }

    function addAsteroid() {
      glftLoader.load("asteroid/scene.gltf", (gltfScene) => {
        gltfScene.scene.scale.set(10, 10, 10);
        gltfScene.scene.position.x = Math.random() * 150 - 75;
        gltfScene.scene.position.y = Math.random() * 150 - 75;
        gltfScene.scene.position.z = Math.random() * 150 - 75;
        scene.add(gltfScene.scene);
      });
    }

    const enemyBeams: beams[] = [];
    function enemyShoot() {
      const worldPosition = new THREE.Vector3();
      enemies.forEach((zaku) => {
        const random = Math.floor(Math.random() * 4);
        if (random === 1) {
          const zakuScene = zaku.scene;
          zakuScene.updateMatrixWorld();
          worldPosition.setFromMatrixPosition(zakuScene.matrixWorld);
          const randomX = Math.floor(Math.random() * 10) - 5;
          const randomY = Math.floor(Math.random() * 10) - 5;
          const randomZ = Math.floor(Math.random() * 10) - 5;

          const direction = new THREE.Vector3();
          direction.subVectors(
            new THREE.Vector3(randomX, randomY, randomZ),
            worldPosition,
          );
          direction.normalize();

          const enemyBeam = new THREE.Mesh(beamGeometry, enemyBeamMaterial);
          enemyBeam.position.set(
            worldPosition.x,
            worldPosition.y + 5,
            worldPosition.z,
          );
          enemyBeam.lookAt(direction);
          enemyBeam.rotateX((3 * Math.PI) / 2);
          scene.add(enemyBeam);
          setTimeout(() => {
            scene.remove(enemyBeam);
          }, 5000);
          const enemyBeamObj = {
            model: enemyBeam,
            direction: direction,
          };
          enemyBeams.push(enemyBeamObj);
        }
      });
    }

    let bossDirection: THREE.Vector3;
    let bossMoving: boolean = false;
    function bossAction() {
      if (boss && boss.scene) {
        bossMoving = true;
        const worldPosition = new THREE.Vector3();
        let newBossDirection = new THREE.Vector3();
        const bossScene = boss.scene;
        bossScene.updateMatrixWorld();
        worldPosition.setFromMatrixPosition(bossScene.matrixWorld);
        const randomX = Math.floor(Math.random() * 20) - 10;
        const randomY = Math.floor(Math.random() * 20) - 10;
        const randomZ = Math.floor(Math.random() * 20) - 10;

        if (
          worldPosition.distanceTo(
            new THREE.Vector3(
              camera.position.x,
              camera.position.y,
              camera.position.z,
            ),
          ) > 200
        ) {
          newBossDirection.subVectors(worldPosition, camera.position);
        } else {
          newBossDirection.set(randomX, randomY, randomZ);
        }

        newBossDirection.normalize();
        bossDirection = newBossDirection;
      }
    }

    let boss: GLTF;
    let bossID: number;
    function checkVictory() {
      if (enemies.length === 0) {
        bossScreen?.classList.remove("hidden");
        setTimeout(() => {
          bossScreen?.classList.remove("openLonger");
          bossScreen?.classList.add("closeLonger");
          bossDialogue?.classList.add("hidden");
          setTimeout(() => {
            bossScreen?.classList.add("hidden");
            glftLoader.load("char-zaku/scene.glb", (gltfScene) => {
              boss = gltfScene;
              bossID = gltfScene.scene.id;
              gltfScene.scene.scale.set(4, 4, 4);
              gltfScene.scene.position.x = Math.random() * 300 - 150;
              gltfScene.scene.position.y = Math.random() * 300 - 150;
              gltfScene.scene.position.z = Math.random() * 300 - 150;
              parentScenes[gltfScene.scene.uuid] = gltfScene.scene;
              scene.add(gltfScene.scene);
            });
          }, 500);
        }, 7000);
      }
    }

    function checkVictory2() {
      if (!scene.getObjectById(bossID)) {
        victoryScreen?.classList.remove("hidden");
        setTimeout(() => {
          victoryScreen?.classList.remove("open");
          victoryScreen?.classList.add("close");
          victoryScreen?.classList.add("hidden");
          setTimeout(() => {
            victoryScreen?.classList.add("hidden");
          }, 500);
        }, 8000);
      }
    }

    const beams: beams[] = [];
    function fireBeam() {
      fire = true;
      ammo--;
      const cameraPosition = camera.position;
      const rifleBarrel = new THREE.Vector3();

      if (targetx && targety && targetz) {
        const clickedPosition = new THREE.Vector3();
        raycaster.ray.at(2000, clickedPosition); // Change 20 to your desired distance
        rifleBarrel.set(
          beamRifle.scene.position.x,
          beamRifle.scene.position.y + 1.5,
          beamRifle.scene.position.z,
        );
        const beam = new THREE.Mesh(beamGeometry, beamMaterial);
        beam.position.copy(rifleBarrel);

        const newDirection = new THREE.Vector3(targetx, targety, targetz).sub(
          rifleBarrel,
        );
        newDirection.normalize();

        beam.lookAt(clickedPosition);
        beam.rotateX((3 * Math.PI) / 2);

        scene.add(beam);
        setTimeout(() => {
          scene.remove(beam);
        }, 6000);
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
                  // This if statement is to check for zakus
                  const parentSceneId =
                    currentCoord.model.parent?.parent?.parent?.parent?.uuid;
                  if (parentSceneId !== undefined) {
                    const parentScene = parentScenes[parentSceneId];
                    if (parentScene) {
                      scene.remove(parentScene);
                      prevTargetCoords.splice(i, 1);
                      killCounter++;
                      for (let i = 0; i < enemies.length; i++) {
                        // Removes zaku from enemies array
                        const enemyScene = enemies[i].scene.uuid;
                        if (enemyScene === parentScene.uuid) {
                          enemies.splice(i, 1);
                        }
                      }
                      checkVictory();
                    }
                  }
                  if (currentCoord.model.name !== "ENEMY_ZAKO_3_0") {
                    scene.remove(boss.scene);
                    checkVictory2();
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
              }
            }
          }
        }
      }

      for (let i = 0; i < enemyBeams.length; i++) {
        const currentBeam = enemyBeams[i];
        if (currentBeam.model && currentBeam.direction) {
          currentBeam.model?.position.sub(
            currentBeam.direction.clone().multiplyScalar(beamSpeed),
          );
        }
      }
      // const gunbarrelposition = new THREE.Vector3(
      //   beamRifle.scene.position.x,
      //   beamRifle.scene.position.y,
      //   beamRifle.scene.position.z,
      // );
    }

    function highlightEnemy() {
      const worldPosition = new THREE.Vector3();
      if (selectedModel) {
        const checkBoss = selectedModel.name.substring(
          selectedModel.name.length - 6,
        );
        if (selectedModel.name === "ENEMY_ZAKO_3_0" || checkBoss === "1861_0") {
          selectedModel.updateMatrixWorld();
          worldPosition.setFromMatrixPosition(selectedModel.matrixWorld);
          const dx = worldPosition.x - camera.position.x;
          const dy = worldPosition.y - camera.position.y;
          const dz = worldPosition.z - camera.position.z;
          const distance = Math.sqrt(dx * dx + dy * dy + dz * dz);
          highlight.position.set(5, 3, -15);
          if (selectedModel.name === "ENEMY_ZAKO_3_0") {
            highlightDiv.innerHTML = `<div class='w-48'><h1 class='text-cyan-400 underline'>ENEMY_ZAKO_3_0</h1><p class='text-cyan-400'>Enemy is ${distance} meters away.</p></div>`;
            bracket1.position.set(
              worldPosition.x,
              worldPosition.y,
              worldPosition.z,
            );
            bracket2.position.set(
              worldPosition.x,
              worldPosition.y,
              worldPosition.z,
            );
          } else {
            highlightDiv.innerHTML = `<div class='w-48'><h1 class='text-cyan-400 underline'>RED_COMET_ZAKU</h1><p class='text-cyan-400'>Enemy is ${distance} meters away.</p></div>`;
            bracket1.position.set(
              worldPosition.x,
              worldPosition.y - 6,
              worldPosition.z,
            );
            bracket2.position.set(
              worldPosition.x,
              worldPosition.y - 6,
              worldPosition.z,
            );
          }

          bracket1.lookAt(camera.position);
          bracket2.lookAt(camera.position);
          scene.add(bracket1);
          scene.add(bracket2);
          camera.add(highlight);
          // scene.add(line);
        }
      } else {
        scene.remove(bracket1);
        scene.remove(bracket2);
        camera.remove(highlight);
      }
    }

    // Function to handle mouse click
    function onMouseClick(event: MouseEvent) {
      const worldPosition = new THREE.Vector3();
      const clickedPosition = new THREE.Vector3();
      raycaster.ray.at(2000, clickedPosition); // Change 20 to your desired distance
      targetx = clickedPosition.x;
      targety = clickedPosition.y;
      targetz = clickedPosition.z;
      let checkBoss = "";

      if (selectedModel) {
        checkBoss = selectedModel.name.substring(selectedModel.name.length - 6);
        selectedModel.updateMatrixWorld();
        worldPosition.setFromMatrixPosition(selectedModel.matrixWorld);
        if (selectedModel.name === "ENEMY_ZAKO_3_0") {
          targetx = worldPosition.x;
          targety = worldPosition.y + 5;
          targetz = worldPosition.z;

          const targetObj = {
            x: targetx,
            y: targety,
            z: targetz,
            model: selectedModel,
          };

          if (rifleOrSaber === true && ammo > 0)
            prevTargetCoords.push(targetObj);
        } else if (checkBoss === "1861_0") {
          targetx = worldPosition.x;
          targety = worldPosition.y;
          targetz = worldPosition.z;

          const targetObj = {
            x: targetx,
            y: targety,
            z: targetz,
            model: selectedModel,
          };

          if (rifleOrSaber === true && ammo > 0)
            prevTargetCoords.push(targetObj);
        }
      }

      if (
        rifleOrSaber === false &&
        selectedModel &&
        (selectedModel.name === "ENEMY_ZAKO_3_0" || checkBoss === "1861_0")
      ) {
        moving = true;
      }

      if (rifleOrSaber === true && ammo > 0) fireBeam();
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
      if (rifleOrSaber) {
        camera.remove(beamSaberModel.scene);
        scene.add(beamRifle.scene);
      } else {
        scene.remove(beamRifle.scene);
        camera.add(beamSaberModel.scene);
      }
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
      controlsLocked = true;
    });

    addDocClickEvt();

    window.addEventListener("click", onMouseClick);

    controls.addEventListener("unlock", () => {
      blocker?.classList.remove("hidden");
      addDocClickEvt();
      controlsLocked = false;
    });

    window.addEventListener("resize", () => {
      const newWidth = window.innerWidth;
      const newHeight = window.innerHeight;
      camera.aspect = newWidth / newHeight;
      camera.updateProjectionMatrix();
      webglRenderer.setSize(newWidth, newHeight);
      CSS3Drenderer.setSize(newWidth, newHeight);
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

      if (!intervalID) {
        intervalID = setInterval(enemyShoot, 3000);
      }

      if (!intervalID2) {
        intervalID2 = setInterval(bossAction, 1150);
      }

      if (boss && boss.scene) {
        boss.scene.lookAt(camera.position);
        boss.scene.rotateY((3 * Math.PI) / 2);
        if (bossDirection && bossMoving) {
          boss.scene.position.sub(bossDirection.clone().multiplyScalar(0.5));
          setTimeout(() => {
            bossMoving = false;
          }, 900);
        }
      }

      explosionList.forEach((explosion) => {
        explosion.rotation.x += 0.01;
        explosion.rotation.y += 0.01;
      });

      for (let i = 0; i < enemies.length; i++) {
        enemies[i].scene.lookAt(camera.position);
      }

      updateBeams();
      highlightEnemy();

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
      } else {
        // No intersection, clear the selected model
        selectedModel = null;
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
            meleeAnimation = true;
            setTimeout(() => {
              meleeAnimation = false;
              if (previousSelectedModel.model) {
                let parentSceneId =
                  previousSelectedModel.model.parent?.parent?.parent?.parent
                    ?.uuid;

                if (parentSceneId !== undefined) {
                  // This statement is to check for normal zakus
                  const parentScene = parentScenes[parentSceneId];
                  if (parentScene) {
                    scene.remove(parentScene);
                    previousSelectedModel.deleted = true;
                    killCounter++;
                    for (let i = 0; i < enemies.length; i++) {
                      const enemyScene = enemies[i].scene.uuid;
                      if (enemyScene === parentScene.uuid) {
                        enemies.splice(i, 1);
                      }
                    }
                    checkVictory();
                  }
                }

                if (previousSelectedModel.model.name !== "ENEMY_ZAKO_3_0") {
                  scene.remove(boss.scene);
                  checkVictory2();
                }

                const sphere = new THREE.Mesh(geometry, fireMaterial);
                sphere.scale.set(5.5, 5.5, 5.5);
                if (targetx && targety && targetz)
                  sphere.position.set(targetx, targety, targetz);
                explosionList.push(sphere);
                scene.add(sphere);
                setTimeout(() => {
                  scene.remove(sphere);
                }, 2000);
                targetx = null;
                targety = null;
                targetz = null;
              }
            }, 500);
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

      if (beamRifle && beamSaberModel) {
        if (rifleOrSaber) {
          // Define an offset vector for the weapon
          const offset = new THREE.Vector3(0.8, -1.9, -1); // Adjust these values as needed

          // Apply the camera's rotation to the offset
          offset.applyEuler(camera.rotation);
          const cameraPosition = camera.position.clone();
          const weaponPosition = cameraPosition.clone().add(offset);

          beamRifle.scene.rotation.copy(camera.rotation);
          beamRifle.scene.position.copy(weaponPosition);
        } else {
          if (meleeAnimation === false) {
            beamSaberModel.scene.rotation.x = 0;
            beamSaberModel.scene.rotation.y = 0;
          } else {
            beamSaberModel.scene.rotation.x -= 0.03;
            beamSaberModel.scene.rotation.y -= 0.01;
          }
          beamSaberModel.scene.position.set(0.3, -0.5, -1);
        }
      }

      prevTime = time;

      webglRenderer.render(scene, camera);
      CSS3Drenderer.render(scene, camera);
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

    for (let i = 0; i < 15; i++) {
      addEnemy();
    }

    for (let i = 0; i < 6; i++) {
      addAsteroid();
    }

    let intervalID: ReturnType<typeof setInterval> | null = null;
    let intervalID2: ReturnType<typeof setInterval> | null = null;

    animate();
  }
</script>

<div class="flex justify-center items-center h-screen absolute inset-0">
  <div class="fixed" id="blocker">
    <h1 class="text-cyan-400 text-2xl underline font-mono font-semiboldbold">
      CLICK ANYWHERE TO START
    </h1>
  </div>
  {#if controlsLocked === true}
    <img src="crosshair.png" alt="crosshair" class="fixed" />
  {/if}
</div>

<div
  class="border-4 border-blue-400 w-56 fixed text-neutral-400 my-16 mx-16 opening-line open"
>
  <div class="border-4 border-cyan-400 h-full">
    <div class="border-4 border-blue-400 h-full bg-opacity-50 bg-blue-400">
      <p class="text-xl flex flex-col p-2 font-mono font-semiboldbold">
        WEAPON EQUIPPED: {#if rifleOrSaber}
          <div class="flex flex-col">
            <span>BEAM RIFLE</span>
            <span class="">AMMO: {ammo}/16</span>
          </div>
        {/if}
        {#if !rifleOrSaber}
          <span>BEAM SABER</span>
        {/if}

        <span>ENEMIES REMAINING: {15 - killCounter}</span>
      </p>
    </div>
  </div>
</div>

<div
  class="border-4 border-blue-400 w-56 fixed text-neutral-400 my-16 mx-16 items-end right-0 opening-line open hidden"
  id="victory-screen"
>
  <div class="border-4 border-cyan-400 h-full">
    <div class="border-4 border-blue-400 h-full bg-opacity-50 bg-blue-400">
      <h1 class="text-xl p-2 font-mono font-semiboldbold">
        ALL ENEMIES HAVE BEEN DEFEATED.
      </h1>
    </div>
  </div>
</div>

<div
  class="border-4 border-blue-400 w-[300px] fixed text-neutral-400 my-16 mx-16 items-end right-0 opening-line openLonger hidden"
  id="boss-battle"
>
  <div class="border-4 border-cyan-400 h-full">
    <div
      class="border-4 border-blue-400 h-full bg-opacity-50 bg-blue-400 flex flex-col justify-around"
      id="boss-dialogue"
    >
      <img src="char.webp" alt="char" class="p-2" />
      <h1 class="text-xl p-2 font-mono font-semiboldbold">
        Let's test the performance of the Federation's mobile suit!
      </h1>
    </div>
  </div>
</div>
