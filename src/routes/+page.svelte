<script>
  import StarBackground from './StarBackground.svelte';
  import StartMenu from './StartMenu.svelte';
  import '../app.css';
  import { onMount, onDestroy } from 'svelte';

  let windowPos = { x: 0, y: 0 };
  let currentUrl = 'https://www.msn.com/';
  let inputUrl = 'https://www.msn.com/';
  let urlHistory = ['https://www.msn.com/'];
  let historyIndex = 0;
  let currentTime = '';

  $: inputUrl = currentUrl;

  let timeInterval;

  function updateTime() {
    currentTime = new Date().toLocaleTimeString();
  }

  onMount(() => {
    windowPos.x = window.innerWidth / 2 - 300;
    windowPos.y = window.innerHeight / 2 - 200;
    updateTime();
    timeInterval = setInterval(updateTime, 1000);
  });

  onDestroy(() => {
    if (timeInterval) {
      clearInterval(timeInterval);
    }
  });
  let isDragging = false;
  let dragOffset = { x: 0, y: 0 };

  let isMinimized = false;
  let isMaximized = false;
  let originalSize = { width: 600, height: 'auto' };
  let originalPos = { x: 100, y: 100 };
  let sidebarOpen = false;
  let favorites = [
    { name: 'NextJS', url: 'https://nextjs.org' },
    { name: 'HackClub', url: 'https://hackclub.com' }
  ];

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

  function navigateToUrl(url) {
    if (!url.startsWith('http://') && !url.startsWith('https://')) {
      url = 'https://' + url;
    }
    currentUrl = url;
    if (historyIndex < urlHistory.length - 1) {
      urlHistory = urlHistory.slice(0, historyIndex + 1);
    }
    urlHistory.push(url);
    historyIndex = urlHistory.length - 1;
  }

  function goToUrl() {
    if (inputUrl.trim()) {
      navigateToUrl(inputUrl.trim());
    }
  }

  function goBack() {
    if (historyIndex > 0) {
      historyIndex--;
      currentUrl = urlHistory[historyIndex];
    }
  }

  function goForward() {
    if (historyIndex < urlHistory.length - 1) {
      historyIndex++;
      currentUrl = urlHistory[historyIndex];
    }
  }

  function toggleSidebar() {
    sidebarOpen = !sidebarOpen;
  }

  function removeFavorite(url) {
    favorites = favorites.filter(fav => fav.url !== url);
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
      <button class="nav-btn" on:click={() => window.location.reload()}>
        <svg class="nav-icon" viewBox="0 0 16 16" fill="none" xmlns="http://www.w3.org/2000/svg">
          <rect x="4" y="4" width="8" height="8" fill="white" stroke="currentColor" stroke-width="1"/>
          <path d="M6 5 L8 3 L10 5" stroke="currentColor" stroke-width="1.5" fill="none" stroke-linecap="round" stroke-linejoin="round"/>
          <path d="M10 11 L8 13 L6 11" stroke="currentColor" stroke-width="1.5" fill="none" stroke-linecap="round" stroke-linejoin="round"/>
          <path d="M8 3 L8 6" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
          <path d="M8 10 L8 13" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
        </svg>
        <span class="nav-text">Refresh</span>
      </button>
      <button class="nav-btn" on:click={() => navigateToUrl('https://www.msn.com/')}>
        <svg class="nav-icon" viewBox="0 0 16 16" fill="none" xmlns="http://www.w3.org/2000/svg">
          <path d="M8 2 L3 6 L3 13 L6 13 L6 9 L10 9 L10 13 L13 13 L13 6 Z" fill="white" stroke="currentColor" stroke-width="1"/>
          <rect x="6" y="11" width="4" height="2" fill="currentColor"/>
        </svg>
        <span class="nav-text">Home</span>
      </button>
      <button class="nav-btn" on:click={toggleSidebar}>
        <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 32 32">
          <polygon points="16,4 20,12 29,12 22,18 25,26 16,21 7,26 10,18 3,12 12,12"
            fill="white"/>
        </svg>


        <span class="nav-text">Favorites</span>
      </button>
      <button class="nav-btn">
        <svg class="nav-icon" viewBox="0 0 16 16" fill="none" xmlns="http://www.w3.org/2000/svg">
          <circle cx="8" cy="8" r="6" fill="white" stroke="currentColor" stroke-width="1"/>
          <path d="M8 4 L8 8 L10 10" stroke="black" stroke-width="1.5" stroke-linecap="round" fill="none"/>
          <path d="M8 8 L8 12" stroke="black" stroke-width="1.5" stroke-linecap="round"/>
          <path d="M8 4 A4 4 0 0 1 8 8" stroke="currentColor" stroke-width="1" fill="none"/>
        </svg>
        <span class="nav-text">History</span>
      </button>
    </div>
  </div>

  <div class="address-bar">
    <div class="address-label">Address:</div>
    <input type="text" class="address-input" bind:value={inputUrl} on:keydown={(e) => { if (e.key === 'Enter') goToUrl(); }}>
    <button class="go-btn" on:click={goToUrl}>Go</button>
  </div>

  <div class="browser-content" class:sidebar-open={sidebarOpen}>
    {#if sidebarOpen}
    <div class="sidebar">
      <div class="sidebar-header">
        <span class="sidebar-title">Favorites</span>
      </div>
      <div class="favorites-list">
        {#each favorites as favorite}
        <div class="favorite-item" role="button" tabindex="0" on:click={() => navigateToUrl(favorite.url)} on:keydown={(e) => { if (e.key === 'Enter' || e.key === ' ') navigateToUrl(favorite.url); }}>
          <span class="favorite-name">{favorite.name}</span>
          <button class="remove-favorite-btn" on:click|stopPropagation={() => removeFavorite(favorite.url)} on:keydown|stopPropagation={(e) => { if (e.key === 'Enter' || e.key === ' ') removeFavorite(favorite.url); }} title="Remove favorite" aria-label="Remove {favorite.name} from favorites">×</button>
        </div>
        {/each}
      </div>
    </div>
    {/if}
    <div class="webpage-content">
    {#if currentUrl === 'https://www.msn.com/'}
      <div class="social-icons">
        <a href="https://github.com" target="_blank" rel="noopener noreferrer">
          <img src="https://github.githubassets.com/images/modules/site/icons/footer/github-mark.svg" alt="GitHub" class="social-icon github-icon">
        </a>
        <a href="https://slack.com" target="_blank" rel="noopener noreferrer">
          <img src="https://a.slack-edge.com/80588/marketing/img/icons/icon_slack_hash_colored.png" alt="Slack" class="social-icon">
        </a>
      </div>
      <button class="projects-btn" on:click={() => window.location.href = '/projects'}>Projects</button>
    {:else}
      <iframe
        src={currentUrl}
        class="web-iframe"
        title="Web content"
        sandbox="allow-scripts allow-same-origin allow-forms allow-popups"
      ></iframe>
    {/if}
    </div>
  </div>

  <div class="status-bar">
    <span>Done</span>
    <span>My Computer</span>
  </div>
</div>
{/if}

<StartMenu />

<div class="time-display">
  {currentTime}
</div>

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
    min-width: 60px;
    height: 50px;
    background: #c0c0c0;
    border: 2px outset #c0c0c0;
    font-family: 'MS Sans Serif', sans-serif;
    font-size: 11px;
    cursor: pointer;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: flex-start;
    padding: 3px 4px;
    position: relative;
    image-rendering: pixelated;
    image-rendering: -moz-crisp-edges;
    image-rendering: crisp-edges;
  }

  .nav-btn:hover:not(:disabled) {
    border: 2px inset #c0c0c0;
  }

  .nav-btn:active:not(:disabled) {
    border: 2px inset #c0c0c0;
  }

  .nav-btn:disabled {
    opacity: 0.5;
  }

  .nav-icon {
    width: 18px;
    height: 18px;
    margin-bottom: 1px;
    display: block;
    image-rendering: pixelated;
    image-rendering: -moz-crisp-edges;
    image-rendering: crisp-edges;
  }

  .nav-btn:disabled .nav-icon {
    stroke: #808080;
    fill: #808080;
  }

  .nav-text {
    font-size: 11px;
    font-weight: normal;
    color: #000;
    line-height: 1;
    display: inline-block;
  }

  .nav-btn:disabled .nav-text {
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

  .web-iframe {
    width: 100%;
    height: 100%;
    border: none;
  }

  .time-display {
    position: fixed;
    bottom: 10px;
    right: 10px;
    background: #c0c0c0;
    border: 2px outset #c0c0c0;
    padding: 4px 8px;
    font-family: 'CustomFont', monospace;
    font-size: 14px;
    color: #000;
    z-index: 10000;
    white-space: nowrap;
  }

  .browser-content {
    flex: 1;
    display: flex;
    transition: all 0.3s ease;
  }

  .sidebar {
    width: 200px;
    background: #c0c0c0;
    border-right: 2px inset #c0c0c0;
    display: flex;
    flex-direction: column;
  }

  .sidebar-header {
    background: #c0c0c0;
    border-bottom: 2px inset #c0c0c0;
    padding: 4px 8px;
    display: flex;
    justify-content: space-between;
    align-items: center;
    font-family: 'MS Sans Serif', sans-serif;
    font-size: 12px;
    font-weight: bold;
  }

  .sidebar-title {
    color: #000;
  }


  .favorites-list {
    flex: 1;
    overflow-y: auto;
  }

  .favorite-item {
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 4px 8px;
    cursor: pointer;
    font-family: 'MS Sans Serif', sans-serif;
    font-size: 12px;
    border-bottom: 1px solid #808080;
  }

  .favorite-item:hover {
    background: #000080;
    color: white;
  }

  .favorite-name {
    flex: 1;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
  }

  .remove-favorite-btn {
    width: 16px;
    height: 16px;
    background: transparent;
    border: none;
    color: #000;
    cursor: pointer;
    font-size: 14px;
    font-weight: bold;
    display: flex;
    align-items: center;
    justify-content: center;
    margin-left: 4px;
  }

  .favorite-item:hover .remove-favorite-btn {
    color: white;
  }

  .remove-favorite-btn:hover {
    background: #ff0000;
    color: white;
  }
</style>
