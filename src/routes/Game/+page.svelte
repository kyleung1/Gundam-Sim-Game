<script lang="ts">
  import * as THREE from "three";
  import { browser } from "$app/environment";

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

    // add 10 enemies
    for (let i = 0; i < 10; i++) {
      addEnemy();
    }

    function onMouseClick(event: MouseEvent) {
      console.log(event);
    }

    camera.position.x = 0;
    camera.position.y = 0;
    camera.position.z = 15;
    const spaceTexture = new THREE.TextureLoader().load("space.png");
    scene.background = spaceTexture;

    // with threejs can only add event listeners to the dom not individual elements, need raycaster
    // https://threejs.org/docs/#api/en/core/Raycaster

    const raycaster = new THREE.Raycaster();
    const pointer = new THREE.Vector2();
    let selectedModel: THREE.Mesh | null = null;

    function onPointerMove(event: MouseEvent) {
      pointer.x = (event.clientX / window.innerWidth) * 2 - 1;
      pointer.y = -(event.clientY / window.innerHeight) * 2 + 1;
    }

    window.addEventListener("resize", () => {
      const newWidth = window.innerWidth;
      const newHeight = window.innerHeight;
      camera.aspect = newWidth / newHeight;
      camera.updateProjectionMatrix();
      renderer.setSize(newWidth, newHeight);
    });

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
        selectedModel = intersects[0].object;
      }

      renderer.render(scene, camera);
      requestAnimationFrame(animate);
    }

    animate();
  }
</script>
