<script lang="ts">
  import { createEventDispatcher, getContext } from 'svelte';
  import type { Writable } from 'svelte/store';
  import { push } from 'svelte-spa-router';

  import Modal from './base/Modal.svelte';
  import type nimbo from '../nimbo';
  
  export let isOpen: boolean;
  
  let boardTitle: string = "";
  let couldNotImport: boolean = false;

  const dispatch = createEventDispatcher();
  const nimboStore: Writable<nimbo> = getContext("nimbo");

  const handleInput = (e: KeyboardEvent): void => {
    if (e.key == "Enter") {
      handleCreateBoard(e);
    }
  }

  const handleClose = (e: CustomEvent): void => {
    couldNotImport = false;
    dispatch("close");
  }

  const handleCreateBoard = async (e: Event): Promise<void> => {
    e.preventDefault();

    if (boardTitle) {
      let boardId: string = await $nimboStore.createBoard(boardTitle);
      push("/b/" + boardId);
    }
  }

  const handlePasteEvent = async (e: ClipboardEvent): Promise<void> => {
    if (e.target.tagName === "INPUT") return;

    let data: string = e.clipboardData.getData('text');

    try {
      let importedBoard: object = JSON.parse(data);
      let boardId: string = await $nimboStore.importTrelloBoard(importedBoard);

      await $nimboStore.init();

      push("/b/" + boardId);
    } catch (e) {
      // failed to import board
      couldNotImport = true;
    }
  }

  $: {
    if (isOpen) {
      boardTitle = "";
    }
  };
</script>

<Modal {isOpen} on:close={handleClose}>
  <div class="p-4" on:paste={handlePasteEvent}>
    <form on:submit={handleCreateBoard}>
      <input title="Please enter a valid board name" autocomplete="off" pattern=".*\S+.*" class="text-light dark:text-dark text-2xl p-2 shadow-none w-full bg-light-300 dark:bg-dark-300 border border-transparent focus:border-indigo-500 appearance-none leading-normal rounded outline-none" placeholder="Nombre del Tablero" bind:value={boardTitle} on:keydown={handleInput} on:keyup|stopPropagation required type="text">
      <button type="submit" class="mt-4 bg-indigo-600 w-full hover:bg-indigo-700 text-white text-xl font-bold py-2 px-4 rounded">
        Crear Tablero
      </button>
      {#if couldNotImport}
        <div class="mt-4 text-red-400 p-1 w-full font-bold text-center rounded">
          Could not import board!
        </div>
      {/if}
    </form>
  </div>
</Modal>