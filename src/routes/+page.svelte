<script>
  import StarBackground from './StarBackground.svelte';
  import StartMenu from './StartMenu.svelte';
  import '../app.css';
  import { onMount, onDestroy } from 'svelte';

  let windows = [
    {
      id: 1,
      windowPos: { x: 0, y: 0 },
      currentUrl: 'https://www.msn.com/',
      inputUrl: 'https://www.msn.com/',
      urlHistory: ['https://www.msn.com/'],
      historyIndex: 0,
      isMinimized: false,
      isMaximized: false,
      originalSize: { width: 600, height: 'auto' },
      originalPos: { x: 100, y: 100 },
      sidebarOpen: false,
      favorites: [
        { name: 'NextJS', url: 'https://nextjs.org' },
        { name: 'HackClub', url: 'https://hackclub.com' }
      ]
    }
  ];

  let currentTime = '';
  let nextWindowId = 2;

  let timeInterval;

  function updateTime() {
    currentTime = new Date().toLocaleTimeString();
  }

  onMount(() => {
    console.log('jQuery version:', window.$.fn.jquery);
    console.log('jQuery UI available:', typeof window.$.fn.draggable === 'function');

    windows[0].windowPos.x = window.innerWidth / 2 - 300;
    windows[0].windowPos.y = window.innerHeight / 2 - 200;
    updateTime();
    timeInterval = setInterval(updateTime, 1000);

    setTimeout(() => {
      window.$('.window').each((i, el) => {
        if (!window.$(el).hasClass('maximized')) {
          window.$(el).draggable({
            handle: '.title-bar',
            containment: 'document'
          });
        }
      });
    }, 100);
  });

  onDestroy(() => {
    if (timeInterval) {
      clearInterval(timeInterval);
    }
  });

  function minimizeWindow(windowId) {
    windows = windows.map(w =>
      w.id === windowId
        ? { ...w, isMinimized: true }
        : w
    );
  }

  function maximizeWindow(windowId) {
    windows = windows.map(w => {
      if (w.id === windowId) {
        if (!w.isMaximized) {
          setTimeout(() => {
            window.$(`.window[data-window-id="${windowId}"]`).draggable('disable');
          }, 50);
          return {
            ...w,
            originalPos: { ...w.windowPos },
            originalSize: { width: 600, height: 400 },
            isMaximized: true,
            windowPos: { x: 0, y: 0 }
          };
        } else {
          setTimeout(() => {
            window.$(`.window[data-window-id="${windowId}"]`).draggable('enable');
          }, 50);
          return {
            ...w,
            windowPos: { ...w.originalPos },
            isMaximized: false
          };
        }
      }
      return w;
    });
  }

  function closeWindow(windowId) {
    windows = windows.filter(w => w.id !== windowId);
  }

  function navigateToUrl(windowId, url) {
    windows = windows.map(w => {
      if (w.id === windowId) {
        if (!url.startsWith('http://') && !url.startsWith('https://')) {
          url = 'https://' + url;
        }
        const newUrlHistory = w.historyIndex < w.urlHistory.length - 1
          ? w.urlHistory.slice(0, w.historyIndex + 1)
          : [...w.urlHistory];
        newUrlHistory.push(url);
        return {
          ...w,
          currentUrl: url,
          urlHistory: newUrlHistory,
          historyIndex: newUrlHistory.length - 1,
          inputUrl: url
        };
      }
      return w;
    });
  }

  function goToUrl(windowId) {
    const window = windows.find(w => w.id === windowId);
    if (window && window.inputUrl.trim()) {
      navigateToUrl(windowId, window.inputUrl.trim());
    }
  }

  function goBack(windowId) {
    windows = windows.map(w => {
      if (w.id === windowId && w.historyIndex > 0) {
        const newHistoryIndex = w.historyIndex - 1;
        const newCurrentUrl = w.urlHistory[newHistoryIndex];
        return {
          ...w,
          historyIndex: newHistoryIndex,
          currentUrl: newCurrentUrl,
          inputUrl: newCurrentUrl
        };
      }
      return w;
    });
  }

  function goForward(windowId) {
    windows = windows.map(w => {
      if (w.id === windowId && w.historyIndex < w.urlHistory.length - 1) {
        const newHistoryIndex = w.historyIndex + 1;
        const newCurrentUrl = w.urlHistory[newHistoryIndex];
        return {
          ...w,
          historyIndex: newHistoryIndex,
          currentUrl: newCurrentUrl,
          inputUrl: newCurrentUrl
        };
      }
      return w;
    });
  }

  function toggleSidebar(windowId) {
    windows = windows.map(w =>
      w.id === windowId
        ? { ...w, sidebarOpen: !w.sidebarOpen }
        : w
    );
  }

  function removeFavorite(windowId, url) {
    windows = windows.map(w =>
      w.id === windowId
        ? { ...w, favorites: w.favorites.filter(fav => fav.url !== url) }
        : w
    );
  }

  function openNewWindow(url = 'https://www.msn.com/') {
    const newWindow = {
      id: nextWindowId++,
      windowPos: { x: Math.random() * 200 + 100, y: Math.random() * 200 + 100 },
      currentUrl: url,
      inputUrl: url,
      urlHistory: [url],
      historyIndex: 0,
      isMinimized: false,
      isMaximized: false,
      originalSize: { width: 600, height: 'auto' },
      originalPos: { x: 100, y: 100 },
      sidebarOpen: false,
      favorites: [
        { name: 'NextJS', url: 'https://nextjs.org' },
        { name: 'HackClub', url: 'https://hackclub.com' }
      ]
    };
    windows = [...windows, newWindow];

    setTimeout(() => {
      window.$(`.window[data-window-id="${newWindow.id}"]`).draggable({
        handle: '.title-bar',
        containment: 'document'
      });
    }, 50);
  }
</script>

<div style="background-color: black; position: fixed; top: 0; left: 0; right: 0; bottom: 0;">
  <StarBackground />
</div>

{#each windows as window (window.id)}
  {#if !window.isMinimized}
    <div class="window"
       class:maximized={window.isMaximized}
       data-window-id={window.id}
       style="position: fixed; left: {window.windowPos.x}px; top: {window.windowPos.y}px; width: {window.isMaximized ? '100vw' : '600px'}; height: {window.isMaximized ? '100vh' : '400px'}; z-index: 2000;">
    <div class="title-bar" role="button" tabindex="0">
      <div class="title-bar-text">
        Microsoft Internet Explorer
      </div>

      <div class="title-bar-controls">
        <button aria-label="Minimize" on:click={() => minimizeWindow(window.id)}></button>
        <button aria-label="Maximize" on:click={() => maximizeWindow(window.id)}></button>
        <button aria-label="Close" on:click={() => closeWindow(window.id)}></button>
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
        <button class="nav-btn" on:click={() => navigateToUrl(window.id, 'https://www.msn.com/')}>
          <svg class="nav-icon" viewBox="0 0 16 16" fill="none" xmlns="http://www.w3.org/2000/svg">
            <path d="M8 2 L3 6 L3 13 L6 13 L6 9 L10 9 L10 13 L13 13 L13 6 Z" fill="white" stroke="currentColor" stroke-width="1"/>
            <rect x="6" y="11" width="4" height="2" fill="currentColor"/>
          </svg>
          <span class="nav-text">Home</span>
        </button>
        <button class="nav-btn" on:click={() => toggleSidebar(window.id)}>
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
      <input type="text" class="address-input" bind:value={window.inputUrl} on:keydown={(e) => { if (e.key === 'Enter') goToUrl(window.id); }}>
      <button class="go-btn" on:click={() => goToUrl(window.id)}>Go</button>
    </div>

    <div class="browser-content" class:sidebar-open={window.sidebarOpen}>
      {#if window.sidebarOpen}
      <div class="sidebar">
        <div class="sidebar-header">
          <span class="sidebar-title">Favorites</span>
        </div>
        <div class="favorites-list">
          {#each window.favorites as favorite}
          <div class="favorite-item" role="button" tabindex="0" on:click={() => navigateToUrl(window.id, favorite.url)} on:keydown={(e) => { if (e.key === 'Enter' || e.key === ' ') navigateToUrl(window.id, favorite.url); }}>
            <span class="favorite-name">{favorite.name}</span>
            <button class="remove-favorite-btn" on:click|stopPropagation={() => removeFavorite(window.id, favorite.url)} on:keydown|stopPropagation={(e) => { if (e.key === 'Enter' || e.key === ' ') removeFavorite(window.id, favorite.url); }} title="Remove favorite" aria-label="Remove {favorite.name} from favorites">×</button>
          </div>
          {/each}
        </div>
      </div>
      {/if}
      <div class="webpage-content">
      {#if window.currentUrl === 'https://www.msn.com/'}
        <div class="social-icons">
          <a href="https://github.com" target="_blank" rel="noopener noreferrer">
            <img src="https://github.githubassets.com/images/modules/site/icons/footer/github-mark.svg" alt="GitHub" class="social-icon github-icon">
          </a>
          <a href="https://slack.com" target="_blank" rel="noopener noreferrer">
            <img src="https://a.slack-edge.com/80588/marketing/img/icons/icon_slack_hash_colored.png" alt="Slack" class="social-icon">
          </a>
        </div>
        <button class="projects-btn" on:click={() => window.location.href = '/projects'}>Projects</button>
      {:else if window.currentUrl === 'https://www.msn.com/update'}
        <div class="update-message">
          No Updates Available
        </div>
      {:else}
        <iframe
          src={window.currentUrl}
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
{/each}

<StartMenu {openNewWindow} />

<div class="time-display">
  {currentTime}
</div>

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
    position: fixed !important;
    top: 0 !important;
    left: 0 !important;
    width: 100vw !important;
    height: 100vh !important;
    border: none;
    box-shadow: none;
    z-index: 3000 !important;
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

  .update-message {
    display: flex;
    align-items: center;
    justify-content: center;
    height: 100%;
    font-family: 'MS Sans Serif', sans-serif;
    font-size: 16px;
    color: #000;
    font-weight: normal;
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
