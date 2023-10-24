<script lang="ts">
  function backHome() {
    window.location.href = "/";
  }
</script>

<div
  class="bg-neutral-700 h-full flex flex-col items-center text-cyan-400 font-mono font-semibold"
>
  <img src="logo.png" alt="logo" class="w-1/5 mt-10" />
  <h1 class="text-3xl">Simulation Game</h1>

  <div class="w-screen bg-blue-800 my-10 border-4 border-red-500">
    <div class="border-4 border-yellow-500 flex flex-col items-center">
      <h1 class="text-7xl text-center mt-10">About</h1>
      <p class="text-white p-10 w-3/5 text-xl">
        As a Gundam fan, this first person simulation game was made as a passion
        project in order to learn the Three.JS library. In this game you will be
        able to pilot the rx-78-2 Gundam in space and battle enemy zakus. This
        project was made using SvelteKit, Three.JS, TailwindCSS, TypeScript, and
        Vite.
      </p>
      <p class="text-white p-10 w-3/5 text-xl">
        This project was my first time trying out Three.JS and it proved to be
        somewhat challenging. Altogether the game took around a month and a half
        to make. There are still some flaws in the game I would like to fix in
        the future such as the beams not shooting from exactly the barrel of the
        rifle and some bugs when melee attacking. In the future I would really
        like to try learning blender in order to produce my own 3d Models and
        replace the current weapons as they are just placeholders.
      </p>
      <p class="text-white p-10 w-3/5 text-xl">
        Feel free to try out the game or leave suggestions about improvements I
        should make in the future. Below is the link to this project's github
        repository.
      </p>
      <button
        class=""
        on:click={() => {
          window.location.href = "https://github.com/kyleung1/Gundam-Sim-Game";
        }}
      >
        <img src="newGithub.png" alt="github" class="" />
      </button>
      <p>Last updated 10/23/2023</p>
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
