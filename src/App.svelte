<script>
  import Page1 from "./lib/Page1.svelte";
  import Page2 from "./lib/Page2.svelte";
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
      <Page1 on:addName={(e) => (name = e.detail.text)} />
    {:else if page2}
      <Page2 {name} />
    {/if}
  </div>
</main>

<style>
</style>
