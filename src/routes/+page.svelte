<script>
  import StarBackground from './StarBackground.svelte';
  import StartMenu from './StartMenu.svelte';
  import '../app.css';

  let windowPos = { x: 100, y: 100 };
  let isDragging = false;
  let dragOffset = { x: 0, y: 0 };

  let isMinimized = false;
  let isMaximized = false;
  let originalSize = { width: 250, height: 'auto' };
  let originalPos = { x: 100, y: 100 };

  function startDrag(event) {
    isDragging = true;
    dragOffset.x = event.clientX - windowPos.x;
    dragOffset.y = event.clientY - windowPos.y;
  }

  function drag(event) {
    if (isDragging) {
      windowPos.x = event.clientX - dragOffset.x;
      windowPos.y = event.clientY - dragOffset.y;
    }
  }

  function stopDrag() {
    isDragging = false;
  }

  function minimizeWindow() {
    isMinimized = true;
  }

  function maximizeWindow() {
    if (!isMaximized) {
      originalPos = { ...windowPos };
      isMaximized = true;
      windowPos = { x: 0, y: 0 };
    } else {
      windowPos = { ...originalPos };
      isMaximized = false;
    }
  }

  function closeWindow() {
    isMinimized = true;
  }
</script>

<div style="background-color: black; position: fixed; top: 0; left: 0; right: 0; bottom: 0;">
  <StarBackground />
</div>

{#if !isMinimized}
<div class="window"
     class:maximized={isMaximized}
     style="position: fixed; left: {windowPos.x}px; top: {windowPos.y}px; width: {isMaximized ? '100vw' : '250px'}; height: {isMaximized ? '100vh' : 'auto'}; z-index: 2000;">
  <div class="title-bar" role="button" tabindex="0" on:mousedown={isMaximized ? null : startDrag} on:mouseup={stopDrag}>
    <div class="title-bar-text">
      My First Program
    </div>

    <div class="title-bar-controls">
      <button aria-label="Minimize" on:click={minimizeWindow}></button>
      <button aria-label="Maximize" on:click={maximizeWindow}></button>
      <button aria-label="Close" on:click={closeWindow}></button>
    </div>
  </div>
  <div class="window-body">
    <p>Hello, world!</p>
    <section class="field-row" style="justify-content: flex-end">
      <button>OK</button>
      <button>Cancel</button>
    </section>
  </div>
</div>
{/if}

<StartMenu />

<svelte:window on:mousemove={drag} on:mouseup={stopDrag} />

<style>
  .window {
    background: #c0c0c0;
    border: 2px solid black;
    box-shadow: 4px 4px 8px rgba(0,0,0,0.3);
    transition: width 0.3s ease-in-out, height 0.3s ease-in-out, border 0.3s ease-in-out, box-shadow 0.3s ease-in-out;
  }

  .window.maximized {
    border: none;
    box-shadow: none;
  }

  .title-bar {
    background: blue;
    color: white;
    padding: 4px;
    cursor: move;
  }

  .window.maximized .title-bar {
    cursor: default;
  }

  .window-body {
    padding: 8px;
    background: #c0c0c0;
  }
</style>
