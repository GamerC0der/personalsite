<script>
  import StarBackground from './StarBackground.svelte';
  import StartMenu from './StartMenu.svelte';
  import '../app.css';

  // Window drag state
  let windowPos = { x: 100, y: 100 };
  let isDragging = false;
  let dragOffset = { x: 0, y: 0 };

  // Window drag functions
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
</script>

<div style="background-color: black; position: fixed; top: 0; left: 0; right: 0; bottom: 0;">
  <StarBackground />
</div>

<div class="window" style="position: fixed; left: {windowPos.x}px; top: {windowPos.y}px; width: 250px; z-index: 2000;">
  <div class="title-bar" role="button" tabindex="0" on:mousedown={startDrag} on:mouseup={stopDrag}>
    <div class="title-bar-text">
      My First Program
    </div>

    <div class="title-bar-controls">
      <button aria-label="Minimize"></button>
      <button aria-label="Maximize"></button>
      <button aria-label="Close"></button>
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

<StartMenu />

<svelte:window on:mousemove={drag} on:mouseup={stopDrag} />

<style>
  .window {
    background: #c0c0c0;
    border: 2px solid black;
    box-shadow: 4px 4px 8px rgba(0,0,0,0.3);
  }
  .title-bar {
    background: blue;
    color: white;
    padding: 4px;
    cursor: move;
  }
  .window-body {
    padding: 8px;
    background: #c0c0c0;
  }
</style>
