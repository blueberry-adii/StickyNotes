<script lang="ts">
  import { onMount, onDestroy } from "svelte";
  export let name;
  let users = [];
  let notes = [];
  let message = "";

  let draggedIndex = null;
  let offsetX = 0;
  let offsetY = 0;

  const colors = [
    "#ff6b6b",
    "#f9a825",
    "#42a5f5",
    "#7e57c2",
    "#26a69a",
    "#ef5350",
    "#5c6bc0",
    "#009688",
    "#ab47bc",
    "#fb8c00",
  ];

  function checkUser() {
    let idx = users.findIndex((user) => user.name == name);

    if (idx !== -1) {
      notes = [
        ...users[idx].notes.map((note) => {
          return { ...note, isEditing: false, editedText: "" };
        }),
      ];
    } else {
      users = [...users, { name: name, notes: [] }];
      notes = [];
      saveUsersToStorage();
    }

    return idx;
  }

  function saveUsersToStorage() {
    localStorage.setItem("users", JSON.stringify(users));
  }

  function loadUsersFromStorage() {
    const stored = localStorage.getItem("users");
    if (stored) {
      users = JSON.parse(stored);
    }
  }

  function startEditing(index) {
    save(index);
    notes = notes.map((n, i) => {
      if (i === index) {
        return { ...n, isEditing: true, editedText: n.text };
      } else {
        return { ...n, isEditing: false, editedText: "" };
      }
    });
  }

  function save(index) {
    const currentEditingIndex = notes.findIndex((n) => n.isEditing);

    if (currentEditingIndex !== -1 && currentEditingIndex !== index) {
      saveEdit(currentEditingIndex);
    }
  }

  // function clampNoteToScreen(note) {
  //   const noteWidth = 100;
  //   const noteHeight = 100;

  //   const maxX = window.innerWidth - noteWidth;
  //   const maxY = window.innerHeight - noteHeight;

  //   note.x = Math.max(0, Math.min(note.x, maxX));
  //   note.y = Math.max(0, Math.min(note.y, maxY));

  //   return note;
  // }

  // function handleResize() {
  //   notes = notes.map((note) => clampNoteToScreen(note));
  //   saveNotesToStorage();
  // }

  function handleEditKey(e, index) {
    if (e.key === "Enter") {
      saveEdit(index);
    } else if (e.key === "Escape") {
      cancelEdit(index);
    }
  }

  function cancelEdit(index) {
    notes[index].isEditing = false;
    notes[index].editedText = "";
    notes = [...notes];
  }

  function saveEdit(index) {
    const note = notes[index];
    if (!note.editedText.trim()) return;

    notes[index].text = note.editedText;
    notes[index].isEditing = false;
    notes[index].editedText = "";
    notes = [...notes];
    saveNotesToStorage();
  }

  function saveNotesToStorage() {
    let idx = users.findIndex((user) => user.name == name);

    if (idx !== -1) {
      users[idx] = { ...users[idx], notes: notes };
      users = [...users];
      saveUsersToStorage();
    }
  }

  function deleteNote(index) {
    console.log("Deleting note");
    notes = notes.filter((_, i) => i !== index);
    saveNotesToStorage();
  }

  function handleClick(event) {
    if (
      event.target.tagName === "INPUT" ||
      event.target.tagName === "BUTTON" ||
      event.target.tagName === "DIV"
    )
      return;
    if (!message.trim()) return;
    const randomColor = colors[Math.floor(Math.random() * colors.length)];

    const percentX = (event.clientX / window.innerWidth) * 100;
    const percentY = (event.clientY / window.innerHeight) * 100;

    notes = [
      ...notes,
      {
        x: percentX,
        y: percentY,
        text: message,
        color: randomColor,
        isEditing: false,
        editedText: "",
      },
    ];
    saveNotesToStorage();
    message = "";
    console.log("Clicked", notes);
  }

  function startDrag(e, index) {
    draggedIndex = index;
    const note = notes[index];
    let pxX = (note.x / 100) * window.innerWidth;
    let pxY = (note.y / 100) * window.innerHeight;
    offsetX = e.clientX - pxX;
    offsetY = e.clientY - pxY;

    e.preventDefault();
  }

  function drag(e) {
    if (draggedIndex !== null) {
      notes[draggedIndex].x = ((e.clientX - offsetX) / window.innerWidth) * 100;
      notes[draggedIndex].y =
        ((e.clientY - offsetY) / window.innerHeight) * 100;
      notes = [...notes];
    }
  }

  function stopDrag() {
    if (draggedIndex !== null) {
      saveNotesToStorage();
      draggedIndex = null;
    }
  }

  function handleGlobalClick(event) {
    const active = document.activeElement;

    if (
      active &&
      active.tagName === "INPUT" &&
      !event.target.closest("input")
    ) {
      (active as HTMLInputElement).blur();
    }
  }

  onMount(() => {
    loadUsersFromStorage();
    checkUser();
    window.addEventListener("click", handleClick);
    window.addEventListener("click", handleGlobalClick);
    window.addEventListener("pointermove", drag);
    window.addEventListener("pointerup", stopDrag);
  });

  onDestroy(() => {
    window.removeEventListener("click", handleClick);
    window.removeEventListener("click", handleGlobalClick);
    window.removeEventListener("pointermove", drag);
    window.removeEventListener("pointerup", stopDrag);
  });
</script>

<div>
  <input
    class="Type"
    type="text"
    bind:value={message}
    placeholder="Type note & click anywhere"
  />
</div>

{#each notes as note, i}
  <!-- svelte-ignore a11y_click_events_have_key_events -->
  <div
    class="floating"
    role="button"
    tabindex="0"
    style="left: {note.x}%; top: {note.y}%; background-color: {note.color}"
    on:pointerdown={(e) => {
      if (!note.isEditing) startDrag(e, i);
    }}
  >
    {#if note.isEditing}
      <!-- svelte-ignore a11y_autofocus -->
      <input
        type="text"
        bind:value={note.editedText}
        on:keydown={(e) => handleEditKey(e, i)}
        on:blur={() => saveEdit(i)}
        autofocus
      />
    {:else}
      <!-- svelte-ignore a11y_consider_explicit_label -->
      <button
        class="del"
        on:click={(e) => {
          e.stopPropagation();
          startEditing(i);
        }}
      >
        <svg
          width="20px"
          height="20px"
          viewBox="0 0 24.00 24.00"
          id="_24x24_On_Light_Edit"
          data-name="24x24/On Light/Edit"
          xmlns="http://www.w3.org/2000/svg"
          fill="#ffffff"
          stroke="#ffffff"
          stroke-width="0.00024000000000000003"
          ><g id="SVGRepo_bgCarrier" stroke-width="0"></g><g
            id="SVGRepo_tracerCarrier"
            stroke-linecap="round"
            stroke-linejoin="round"
            stroke="#CCCCCC"
            stroke-width="0.192"
          ></g><g id="SVGRepo_iconCarrier">
            <rect id="view-box" width="24" height="24" fill="none"></rect>
            <path
              id="Shape"
              d="M.75,17.5A.751.751,0,0,1,0,16.75V12.569a.755.755,0,0,1,.22-.53L11.461.8a2.72,2.72,0,0,1,3.848,0L16.7,2.191a2.72,2.72,0,0,1,0,3.848L5.462,17.28a.747.747,0,0,1-.531.22ZM1.5,12.879V16h3.12l7.91-7.91L9.41,4.97ZM13.591,7.03l2.051-2.051a1.223,1.223,0,0,0,0-1.727L14.249,1.858a1.222,1.222,0,0,0-1.727,0L10.47,3.91Z"
              transform="translate(3.25 3.25)"
              fill="#ffffff"
            ></path>
          </g></svg
        >
      </button>
      {note.text}
      <!-- svelte-ignore a11y_consider_explicit_label -->
      <button
        class="del"
        on:click={() => {
          deleteNote(i);
        }}
        ><svg
          xmlns="http://www.w3.org/2000/svg"
          x="0px"
          y="0px"
          width="20"
          height="20"
          viewBox="0,0,256,256"
        >
          <g
            fill="#ffffff"
            fill-rule="nonzero"
            stroke="none"
            stroke-width="1"
            stroke-linecap="butt"
            stroke-linejoin="miter"
            stroke-miterlimit="10"
            stroke-dasharray=""
            stroke-dashoffset="0"
            font-family="none"
            font-weight="none"
            font-size="none"
            text-anchor="none"
            style="mix-blend-mode: normal"
            ><g transform="scale(2,2)"
              ><path
                d="M49,1c-1.66,0 -3,1.34 -3,3c0,1.66 1.34,3 3,3h30c1.66,0 3,-1.34 3,-3c0,-1.66 -1.34,-3 -3,-3zM24,15c-7.17,0 -13,5.83 -13,13c0,7.17 5.83,13 13,13h77v63c0,9.37 -7.63,17 -17,17h-40c-9.37,0 -17,-7.63 -17,-17v-52c0,-1.66 -1.34,-3 -3,-3c-1.66,0 -3,1.34 -3,3v52c0,12.68 10.32,23 23,23h40c12.68,0 23,-10.32 23,-23v-63.35937c5.72,-1.36 10,-6.50062 10,-12.64062c0,-7.17 -5.83,-13 -13,-13zM24,21h80c3.86,0 7,3.14 7,7c0,3.86 -3.14,7 -7,7h-80c-3.86,0 -7,-3.14 -7,-7c0,-3.86 3.14,-7 7,-7zM50,55c-1.66,0 -3,1.34 -3,3v46c0,1.66 1.34,3 3,3c1.66,0 3,-1.34 3,-3v-46c0,-1.66 -1.34,-3 -3,-3zM78,55c-1.66,0 -3,1.34 -3,3v46c0,1.66 1.34,3 3,3c1.66,0 3,-1.34 3,-3v-46c0,-1.66 -1.34,-3 -3,-3z"
              ></path></g
            ></g
          >
        </svg></button
      >
    {/if}
  </div>
{/each}

<style>
  .floating {
    position: absolute;
    display: flex;
    justify-content: center;
    align-items: center;
    gap: 6px;
    border-radius: 6px;
    font-weight: 600;
    text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.2);
    max-width: 300px;
    word-wrap: break-word;
    background-color: "#fff";
    transform: translate(-50%, -50%);
    padding: 5px 10px;
    cursor: grab;
    user-select: none;
  }
  .floating:active {
    cursor: grabbing;
  }
  .floating input {
    max-width: 100px;
    max-height: 100px;
    font-size: 1rem;
    padding: 4px;
    border-radius: 4px;
    border: none;
    outline: none;
    background: rgba(255, 255, 255, 0.9);
    color: #222;
  }
  .del {
    display: inline-flex;

    justify-content: center;
    align-items: center;
    padding: 2px;
    border: 0;
    background-color: rgba(240, 255, 255, 0.211);
    transition: background-color 0.2s ease-in-out;
  }
  .del:hover {
    border: 0;
    background-color: rgba(240, 248, 255, 0.393);
  }
</style>
