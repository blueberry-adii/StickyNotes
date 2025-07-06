<script>
  import TwinkleText from "./TwinkleText.svelte";
  import { createEventDispatcher } from "svelte";
  const dispatch = createEventDispatcher();
  let name;
  let isLoggedIn = false;
  let msg = ["Log In into your account", "Please enter your name"];
  let errorMsg = { msg: msg[0], isError: false };
  $: errorMsg.isError = errorMsg.msg == msg[0] ? false : true;
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
    class="nameInput"
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
    {#each instructions as todo, i}
      <li>{i + 1}. {todo}</li>
    {/each}
  {:else}
    <p class={errorMsg.isError ? "error" : ""}>{errorMsg.msg}</p>
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
