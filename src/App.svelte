<script>
  import Counter from "./lib/Counter.svelte";
  import ToDo from "./lib/ToDo.svelte";
  let name;
  let page1 = true;
  $: page2 = !page1;
</script>

<main>
  {#if name}
    <button
      class={page1 ? "active" : ""}
      on:click={() => {
        if (page2) {
          name = "";
        }
        page1 = true;
      }}
    >
      Page 1
    </button>
    <button
      class={page2 ? "active" : ""}
      on:click={() => {
        page1 = false;
      }}
    >
      Page 2
    </button>
  {/if}
  <div class="card">
    {#if page1}
      <Counter on:addName={(e) => (name = e.detail.text)} />
    {:else if page2}
      <ToDo />
    {/if}
  </div>
</main>

<style>
</style>
