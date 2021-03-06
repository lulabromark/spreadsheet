<script>
  import { onMount } from "svelte";
  export let row;
  export let col;
  export let value;
  export let onChangedValue;

  const alpha = " abcdefghijklmnopqrstuvwxyz".split("");
  const delay = 200;
  let timer = 0;
  let selected = false;
  let prevent = false;
  let editing = false;

  // Emits the `unselectAll` event, used to tell all the other cells to unselect
  const emitUnselectAllEvent = () => {
    const unselectAllEvent = new Event("unselectAll");
    if (typeof window !== "undefined") {
      window.document.dispatchEvent(unselectAllEvent);
    }
  };

  //  Used by `componentDid(Un)Mount`, handles the `unselectAll` event response
  const handleUnselectAll = () => {
    if (selected) {
      selected = false;
    }
  };

  // Lifecycle events
  onMount(() => {
    if (typeof window !== "undefined") {
      window.document.addEventListener("unselectAll", handleUnselectAll);
    }
  });

  // When a Cell value changes, re-determine the display value by calling the formula calculation
  const onChange = event => {
    value = event.target.value;
  };

  /**
   * Called by the `onBlur` or `onKeyPressOnInput` event handlers,
   * it escalates the value changed event, and restore the editing
   * state to `false`.
   */
  const hasNewValue = value => {
    onChangedValue(col, row, value);
    editing = false;
  };

  // Handle moving away from a cell, stores the new value
  const onBlur = event => {
    hasNewValue(event.target.value);
  };

  // Handle pressing a key when the Cell is an input element
  const onKeyPressOnInput = event => {
    if (event.key === "Enter") {
      hasNewValue(event.target.value);
    }
  };

  // Handle clicking a cell
  const clicked = () => {
    // Prevent click and double click to conflict
    timer = setTimeout(() => {
      if (!prevent) {
        // Unselect all the other cells and set the current ell state to `selected`
        emitUnselectAllEvent();
        selected = true;
      }
      prevent = false;
    }, delay);
  };

  // Handle doubleclicking a cell
  const doubleClicked = () => {
    // Prevent click and double click to conflict
    clearTimeout(timer);
    prevent = true;

    // Unselect all the other cells and set the current cell state to `selected` & `editing`
    emitUnselectAllEvent();
    editing = true;
    selected = true;
  };
</script>

<style>
  span.cell {
    width: 80px;
    padding: 4px;
    margin: 0;
    height: 25px;
    box-sizing: border-box;
    position: relative;
    display: inline-block;
    color: black;
    border: 1px solid #cacaca;
    text-align: left;
    vertical-align: top;
    font-size: 14px;
    line-height: 15px;
    overflow: hidden;
    font-family: Calibri, Segoe UI, Thonburi, Arial, Verdana, sans-serif;
  }

  span.cell.first-row,
  span.cell.first-col {
    text-align: center;
    background-color: #f0f0f0;
    font-weight: bold;
  }

  span.cell.selected {
    outline-color: blue;
    outline-style: solid;
  }
</style>

{#if col === 0}
  <!-- column 0 -->
  <span class="cell first-col">{row}</span>
{:else}
  {#if row === 0}
    <!-- row 0 -->
    <span class="cell first-row">{alpha[col]}</span>
  {:else if editing}
    <!-- svelte-ignore a11y-autofocus -->
    <input
      class="cell"
      type="text"
      {value}
      on:change={onChange}
      on:blur={onBlur}
      on:keypress={onKeyPressOnInput}
      autoFocus />
  {:else}
    <!-- regular cell -->
    <span
      class="cell {selected ? 'selected' : ''}"
      on:click={clicked}
      on:dblclick={doubleClicked}>
      {value}
    </span>
  {/if}
{/if}
