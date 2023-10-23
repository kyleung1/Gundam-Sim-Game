<script lang="ts">
  function backHome() {
    window.location.href = "/";
  }
</script>

<div class="bg-neutral-700 h-screen flex flex-col items-center text-cyan-400">
  <img src="logo.png" alt="logo" class="w-1/5 mt-10" />
  <h1 class="text-3xl">Simulation Game</h1>

  <div class="hover:border-2 border-blue-400 my-10 rounded-md">
    <div class="hover:border-2 border-cyan-400">
      <button
        class="bg-black p-2 hover:border-2 border-blue-400"
        on:click={backHome}>Back To Menu</button
      >
    </div>
  </div>
</div>
