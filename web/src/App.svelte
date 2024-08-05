<script lang="ts">
  import { fade, fly } from "svelte/transition";
  import Counter from "./lib/Counter.svelte";
  import Trajectory from "./lib/Trajectory.svelte";
  import Orientation from "./lib/Orientation.svelte";
  const pages = [Counter, Trajectory, Orientation];
  let choice = 0;
  $: nextChoice = (choice + 1) % pages.length;
</script>

<main>
  <button
    on:click={() => {
      choice = nextChoice;
    }}
  >
    switch to {nextChoice}
  </button>
  {#key choice}
    <div in:fly={{ x: 100 }} out:fly={{ x: -100 }}>
      <svelte:component this={pages[choice]}></svelte:component>
    </div>
  {/key}
</main>

<style>
  main {
    font-family: Arial, sans-serif;
    position: fixed;
    inset: 0px;
    top: -1px;
    overflow-y: scroll;
    overflow-x: hidden;
  }
  button {
    margin: 20px;
  }
  div {
    position: absolute;
    background-color: white;
    padding: 20px;
    width: calc(100% - 40px);
  }
</style>
