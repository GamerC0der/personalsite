<script>
  import StarBackground from './StarBackground.svelte';
  import StartMenu from './StartMenu.svelte';
  import '../app.css';
  import { onMount } from 'svelte';

  let windowPos = { x: 0, y: 0 };

  onMount(() => {
    windowPos.x = window.innerWidth / 2 - 300;
    windowPos.y = window.innerHeight / 2 - 200;
  });
  let isDragging = false;
  let dragOffset = { x: 0, y: 0 };

  let isMinimized = false;
  let isMaximized = false;
  let originalSize = { width: 600, height: 'auto' };
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
     style="position: fixed; left: {windowPos.x}px; top: {windowPos.y}px; width: {isMaximized ? '100vw' : '600px'}; height: {isMaximized ? '100vh' : '400px'}; z-index: 2000;">
  <div class="title-bar" role="button" tabindex="0" on:mousedown={isMaximized ? null : startDrag} on:mouseup={stopDrag}>
    <div class="title-bar-text">
      Microsoft Internet Explorer
    </div>

    <div class="title-bar-controls">
      <button aria-label="Minimize" on:click={minimizeWindow}></button>
      <button aria-label="Maximize" on:click={maximizeWindow}></button>
      <button aria-label="Close" on:click={closeWindow}></button>
    </div>
  </div>
  <div class="browser-toolbar">
    <div class="nav-buttons">
      <button class="nav-btn" disabled>&lt;</button>
      <button class="nav-btn" disabled>&gt;</button>
      <button class="nav-btn">X</button>
      <button class="nav-btn" on:click={() => window.location.reload()}>R</button>
      <button class="nav-btn" on:click={() => window.location.href = '/'}>H</button>
    </div>
  </div>

  <div class="address-bar">
    <div class="address-label">Address:</div>
    <input type="text" class="address-input" value="http://localhost:5173/" readonly>
    <button class="go-btn">Go</button>
  </div>

  <div class="webpage-content">
    <div class="social-icons">
      <a href="https://github.com" target="_blank" rel="noopener noreferrer">
        <img src="https://github.githubassets.com/images/modules/site/icons/footer/github-mark.svg" alt="GitHub" class="social-icon github-icon">
      </a>
      <a href="https://slack.com" target="_blank" rel="noopener noreferrer">
        <img src="https://a.slack-edge.com/80588/marketing/img/icons/icon_slack_hash_colored.png" alt="Slack" class="social-icon">
      </a>
    </div>
    <button class="projects-btn" on:click={() => window.location.href = '/projects'}>Projects</button>
  </div>

  <div class="status-bar">
    <span>Done</span>
    <span>My Computer</span>
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
    display: flex;
    flex-direction: column;
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

  .browser-toolbar {
    background: #c0c0c0;
    border-bottom: 2px inset #c0c0c0;
    padding: 4px 8px;
    display: flex;
    align-items: center;
  }

  .nav-buttons {
    display: flex;
    gap: 2px;
  }

  .nav-btn {
    width: 22px;
    height: 22px;
    background: #c0c0c0;
    border: 2px outset #c0c0c0;
    font-family: 'MS Sans Serif', sans-serif;
    font-size: 12px;
    cursor: pointer;
    display: flex;
    align-items: center;
    justify-content: center;
  }

  .nav-btn:hover:not(:disabled) {
    border: 2px inset #c0c0c0;
  }

  .nav-btn:disabled {
    color: #808080;
  }

  .address-bar {
    background: #c0c0c0;
    border-bottom: 2px inset #c0c0c0;
    padding: 4px 8px;
    display: flex;
    align-items: center;
    gap: 8px;
  }

  .address-label {
    font-family: 'MS Sans Serif', sans-serif;
    font-size: 12px;
    font-weight: bold;
  }

  .address-input {
    flex: 1;
    padding: 2px 4px;
    border: 2px inset #c0c0c0;
    background: white;
    font-family: 'MS Sans Serif', sans-serif;
    font-size: 12px;
  }

  .go-btn {
    padding: 2px 8px;
    background: #c0c0c0;
    border: 2px outset #c0c0c0;
    font-family: 'MS Sans Serif', sans-serif;
    font-size: 12px;
    cursor: pointer;
  }

  .go-btn:hover {
    border: 2px inset #c0c0c0;
  }

  .webpage-content {
    flex: 1;
    background: white;
    padding: 20px;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    gap: 20px;
    overflow-y: auto;
  }

  .status-bar {
    background: #c0c0c0;
    border-top: 2px inset #c0c0c0;
    padding: 2px 8px;
    display: flex;
    justify-content: space-between;
    font-family: 'MS Sans Serif', sans-serif;
    font-size: 12px;
  }

  .social-icons {
    display: flex;
    gap: 20px;
  }

  .social-icon {
    width: 32px;
    height: 32px;
    cursor: pointer;
    transition: transform 0.2s;
  }

  .social-icon:hover {
    transform: scale(1.1);
  }

  .github-icon {
    filter: brightness(0);
  }

  .projects-btn {
    padding: 8px 16px;
    background: #c0c0c0;
    border: 2px outset #c0c0c0;
    font-family: 'MS Sans Serif', sans-serif;
    font-size: 14px;
    cursor: pointer;
  }

  .projects-btn:hover {
    border: 2px inset #c0c0c0;
  }
</style>
