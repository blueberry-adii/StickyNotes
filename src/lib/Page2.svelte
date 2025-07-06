<script lang="ts">
  import { onMount, onDestroy } from "svelte";
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
    localStorage.setItem("stickyNotes", JSON.stringify(notes));
  }

  function loadNotesFromStorage() {
    const stored = localStorage.getItem("stickyNotes");
    if (stored) {
      notes = JSON.parse(stored).map((n) => ({
        ...n,
        isEditing: false,
        editedText: "",
      }));
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

    notes = [
      ...notes,
      {
        x: event.clientX,
        y: event.clientY,
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
    offsetX = e.clientX - note.x;
    offsetY = e.clientY - note.y;

    e.preventDefault();
  }

  function drag(e) {
    if (draggedIndex !== null) {
      notes[draggedIndex].x = e.clientX - offsetX;
      notes[draggedIndex].y = e.clientY - offsetY;
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
    loadNotesFromStorage();
    window.addEventListener("click", handleClick);
    window.addEventListener("click", handleGlobalClick);
    window.addEventListener("mousemove", drag);
    window.addEventListener("mouseup", stopDrag);
  });

  onDestroy(() => {
    window.removeEventListener("click", handleClick);
    window.removeEventListener("click", handleGlobalClick);
    window.removeEventListener("mousemove", drag);
    window.removeEventListener("mouseup", stopDrag);
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
    style="left: {note.x}px; top: {note.y}px; background-color: {note.color}"
    on:dblclick={(e) => {
      e.stopPropagation();
      startEditing(i);
    }}
    on:mousedown={(e) => {
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
    max-width: fit-content;
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
