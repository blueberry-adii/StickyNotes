<script>
  import { fade } from "svelte/transition";
  let visible = true;
  setInterval(() => {
    visible = !visible;
  }, 500);
</script>

<!-- <button
  on:click={() => {
    visible = !visible;
  }}>Magic!!</button
> -->

{#if visible}
  <p class="absolute" transition:fade>I blink as twinkle twinkle lil stars!!</p>
{/if}

<style>
  .absolute {
    position: absolute;
    left: 50%;
    transform: translateX(-50%);
  }
</style>
