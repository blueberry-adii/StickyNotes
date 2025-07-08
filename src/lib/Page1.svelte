<script>
  import TwinkleText from "./TwinkleText.svelte";
  let { addName } = $props();
  let name = $state();
  let isLoggedIn = $state(false);
  let msg = ["Log In into your account", "Please enter your name"];
  let errorMsg = $state(msg[0]);
  let isError = $derived(errorMsg == msg[1]);
  const instructions = [
    "Click on Page 2",
    "Enter your notes",
    "Click Anywhere",
    "Click on edit icon to edit notes",
    "Press Enter to save them",
    "OR Esc to Escape editing",
    "Click on delete icon to delete notes",
    "Enter your exact name every time to access your saved notes",
  ];
  const logIn = () => {
    if (name) {
      addName(name);
      return true;
    } else {
      errorMsg = msg[1];
      return false;
    }
  };
  const changeErr = () => {
    if (!isLoggedIn) {
      if (!name) {
        errorMsg = msg[1];
      } else {
        errorMsg = msg[0];
      }
    }
  };
  const logOut = () => {
    name = "";
    addName(name);
    return false;
  };
  const handleClick = () => {
    errorMsg = msg[0];
    isLoggedIn = isLoggedIn ? logOut() : logIn();
  };
</script>

<div>
  <input
    class="nameInput"
    type="text"
    bind:value={name}
    readonly={isLoggedIn}
    oninput={changeErr}
  />
  <button onclick={handleClick}>
    {#if isLoggedIn}
      Log Out
    {:else}
      Log In
    {/if}
  </button>
  {#if isLoggedIn}
    <p>Welcome back {name}!</p>
    {#each instructions as todo, i}
      <li>{i + 1}. {todo}</li>
    {/each}
  {:else}
    <p class={isError ? "error" : ""}>{errorMsg}</p>
  {/if}
</div>
<TwinkleText />

<style>
  li {
    list-style: none;
    text-align: justify;
    padding-left: 15%;
    max-width: 300px;
  }
  .error {
    color: rgb(255, 123, 123);
  }
  .nameInput {
    margin-right: 6px;
    margin-bottom: 6px;
  }
</style>
