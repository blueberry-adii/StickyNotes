<script>
  import Ani from "./Ani.svelte";
  import { createEventDispatcher } from "svelte";
  const dispatch = createEventDispatcher();
  let name;
  let isLoggedIn = false;
  let msg = ["Log In into your account", "Please enter your name"];
  let errorMsg = { msg: msg[0], isError: false };
  $: errorMsg.isError = errorMsg.msg == msg[0] ? false : true;
  const todos = [
    "Click on Page 2",
    "Enter your notes",
    "Click Anywhere",
    "Double click to  edit notes",
    "Press Enter to save them",
    "OR Esc to Escape editing",
    "Click on delete icon to delete notes",
  ];
  const logIn = () => {
    if (name) {
      dispatch("addName", { text: name });
      return true;
    } else {
      errorMsg = { msg: msg[1], isError: true };
      return false;
    }
  };
  const changeErr = () => {
    if (!isLoggedIn) {
      if (!name) {
        errorMsg.msg = msg[1];
      } else {
        errorMsg.msg = msg[0];
      }
    }
  };
  const logOut = () => {
    name = "";
    dispatch("addName", { text: name });
    return false;
  };
  const handleClick = () => {
    errorMsg.msg = msg[0];
    isLoggedIn = isLoggedIn ? logOut() : logIn();
  };
</script>

<div>
  <input
    type="text"
    bind:value={name}
    readonly={isLoggedIn}
    on:input={changeErr}
  />
  <button on:click={handleClick}>
    {#if isLoggedIn}
      Log Out
    {:else}
      Log In
    {/if}
  </button>
  {#if isLoggedIn}
    <p>Welcome back {name}!</p>
    {#each todos as todo, i}
      <li>{i + 1}. {todo}</li>
    {/each}
  {:else}
    <p class={errorMsg.isError ? "error" : ""}>{errorMsg.msg}</p>
  {/if}
</div>
<Ani />

<style>
  li {
    list-style: none;
  }
  .error {
    color: rgb(255, 123, 123);
  }
</style>
