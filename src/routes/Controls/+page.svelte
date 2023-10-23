<script lang="ts">
  function backHome() {
    window.location.href = "/";
  }
</script>

<div class="bg-neutral-700 h-screen flex flex-col items-center text-cyan-400">
  <img src="logo.png" alt="logo" class="w-1/5 mt-10" />
  <h1 class="text-3xl">Simulation Game</h1>
  <div class="my-10 bg-white">
    <h1 class="text-7xl text-center">Controls</h1>
    <div class="w-screen flex justify-around my-10">
      <div>
        <h2 class="text-xl">Movement</h2>
        <h3 class="text-xl">WASD + Shift + Spacebar</h3>
        <div class="grid grid-cols-3 grid-rows-2 gap-2">
          <div
            class="col-span-3 col-start-2 border-2 border-black bg-white text-black h-20 w-20 flex justify-center items-center text-5xl"
          >
            W
          </div>

          <div
            class="row-start-2 col-start-1 border-2 border-black bg-white text-black h-20 w-20 flex justify-center items-center text-5xl"
          >
            A
          </div>
          <div
            class="row-start-2 col-start-2 border-2 border-black bg-white text-black h-20 w-20 flex justify-center items-center text-5xl"
          >
            S
          </div>
          <div
            class="row-start-2 col-start-3 border-2 border-black bg-white text-black h-20 w-20 flex justify-center items-center text-5xl"
          >
            D
          </div>
        </div>

        <div
          class="border-2 border-black bg-white text-black h-20 w-50 flex justify-center items-center text-3xl mt-10"
        >
          Shift
        </div>
        <div
          class="border-2 border-black bg-white text-black h-20 w-50 flex justify-center items-center text-3xl"
        >
          Spacebar
        </div>
        <div></div>
      </div>
      <div>
        <h2 class="text-xl">Attack</h2>
        <h3 class="text-xl">Left Mouse Click</h3>
        <img src="/left-click.png" alt="mouse" class="bg-white" />
      </div>
      <div>
        <h2 class="text-xl">Toggle Weapon</h2>
        <h3 class="text-xl">Scroll Wheel</h3>
        <img src="/scroll-wheel.png" alt="mouse" class="bg-white" />
      </div>
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
