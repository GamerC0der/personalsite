<script>
  import StarBackground from './StarBackground.svelte';
  import StartMenu from './StartMenu.svelte';
  import '../app.css';
  import { onMount, onDestroy } from 'svelte';

  let projects = [];
  let filteredProjects = [];
  let searchQuery = '';
  let isLoading = false;

  async function fetchProjects() {
    isLoading = true;
    try {
      const response = await fetch('https://api.github.com/users/gamerc0der/repos?sort=updated&per_page=100');
      if (response.ok) {
        projects = await response.json();
        updateFilteredProjects();
      } else {
        console.error('Failed to fetch projects:', response.status);
      }
    } catch (error) {
      console.error('Error fetching projects:', error);
    } finally {
      isLoading = false;
    }
  }

  function updateFilteredProjects() {
    const nonAiProjects = projects.filter(project =>
      !project.name.toLowerCase().includes('ai')
    );

    if (!searchQuery.trim()) {
      filteredProjects = [...nonAiProjects].slice(0, 5);
    } else {
      const query = searchQuery.toLowerCase();
      filteredProjects = nonAiProjects.filter(project =>
        project.name.toLowerCase().includes(query) ||
        (project.description && project.description.toLowerCase().includes(query)) ||
        project.language?.toLowerCase().includes(query)
      );
    }
  }

  $: if (searchQuery !== undefined) {
    updateFilteredProjects();
  }

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
      historySearchQuery: '',
      favorites: [
        { name: 'Github', url: 'https://github.com/gamerc0der' },
        { name: 'HackClub', url: 'https://hackclub.com' }
      ]
    }
  ];

  let currentTime = '';
  let nextWindowId = 2;

  let runMenuOpen = false;
  let runWindowPos = { x: 100, y: 100 };
  let runCommand = '';

  let messageWindows = [];
  let nextMessageWindowId = 1;

  let contextMenu = {
    visible: false,
    x: 0,
    y: 0,
    target: null
  };

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
    fetchProjects();

    setTimeout(() => {
      window.$('.window').each((i, el) => {
        if (!window.$(el).hasClass('maximized')) {
          window.$(el).draggable({
            handle: '.title-bar',
            containment: 'document'
          });
        }
      });
      window.$('.run-window').draggable({
        handle: '.title-bar',
        containment: 'document'
      });
    }, 100);

    // Add right-click context menu
    document.addEventListener('contextmenu', showContextMenu);
    document.addEventListener('click', hideContextMenu);
  });

  onDestroy(() => {
    if (timeInterval) {
      clearInterval(timeInterval);
    }
    if (typeof document !== 'undefined') {
      document.removeEventListener('contextmenu', showContextMenu);
      document.removeEventListener('click', hideContextMenu);
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

  function showContextMenu(event) {
    event.preventDefault();
    contextMenu.visible = true;
    contextMenu.x = event.clientX;
    contextMenu.y = event.clientY;
    contextMenu.target = event.target;
  }

  function hideContextMenu() {
    contextMenu.visible = false;
  }

  function handleContextMenuAction(action) {
    switch (action) {
      case 'refresh':
        window.location.reload();
        break;
      case 'close':
        if (contextMenu.target && contextMenu.target.closest('.window')) {
          const windowEl = contextMenu.target.closest('.window');
          const windowId = parseInt(windowEl.getAttribute('data-window-id'));
          closeWindow(windowId);
        }
        break;
      case 'minimize':
        if (contextMenu.target && contextMenu.target.closest('.window')) {
          const windowEl = contextMenu.target.closest('.window');
          const windowId = parseInt(windowEl.getAttribute('data-window-id'));
          minimizeWindow(windowId);
        }
        break;
      case 'maximize':
        if (contextMenu.target && contextMenu.target.closest('.window')) {
          const windowEl = contextMenu.target.closest('.window');
          const windowId = parseInt(windowEl.getAttribute('data-window-id'));
          maximizeWindow(windowId);
        }
        break;
    }
    hideContextMenu();
  }

  function toggleRunMenu() {
    runMenuOpen = !runMenuOpen;
    if (runMenuOpen) {
      runCommand = '';
    setTimeout(() => {
      window.$('.run-window').draggable({
        handle: '.title-bar',
        containment: 'document'
      });
    }, 50);
  }
  }

  function openMessageWindow(message, title = 'Message') {
    const offset = messageWindows.length * 30;
    const newMessageWindow = {
      id: nextMessageWindowId++,
      title,
      message,
      windowPos: { x: Math.random() * 200 + 150 + offset, y: Math.random() * 150 + 100 + offset },
      isMinimized: false
    };
    messageWindows = [...messageWindows, newMessageWindow];

    setTimeout(() => {
      window.$(`.message-window[data-window-id="${newMessageWindow.id}"]`).draggable({
        handle: '.title-bar',
        containment: 'document'
      });
    }, 50);
  }

  function closeMessageWindow(windowId) {
    messageWindows = messageWindows.filter(w => w.id !== windowId);
  }

  function executeRunCommand() {
    const command = runCommand.trim().toLowerCase();
    if (!command) return;

    switch (command) {
      case 'help':
        openNewWindow('https://www.msn.com/about');
        break;
      case 'time':
        openMessageWindow(`Current time: ${new Date().toLocaleString()}`, 'Alert');
        break;
      case 'user':
        openMessageWindow('Current user: Guest', 'Alert');
        break;
      case 'logout':
        openMessageWindow('Are you sure you want to log out?', 'Alert');
        break;
      default:
        openMessageWindow(`Unknown command: ${command}. Type 'help' for available commands.`, 'Alert');
        break;
    }

    runCommand = '';
    if (command !== 'help' && command !== 'time' && command !== 'user') {
      runMenuOpen = false;
    }
  }

  function openNewWindow(url = 'https://www.msn.com/') {
    const isPinball = url === 'https://98.js.org/programs/pinball/space-cadet.html';
    const isMinesweeper = url === 'https://98plus.js.org/programs/minesweeper/index.html';
    const isCommandPrompt = url === 'https://98.js.org/programs/command/index.html';
    const newWindow = {
      id: nextWindowId++,
      windowPos: { x: 0, y: 0 },
      currentUrl: url,
      inputUrl: url,
      urlHistory: [url],
      historyIndex: 0,
      isMinimized: false,
      isMaximized: isPinball,
      originalSize: (isPinball || isMinesweeper) ? { width: '100vw', height: 'calc(100vh - 42px)' } : { width: 600, height: 'auto' },
      originalPos: { x: 0, y: 0 },
      sidebarOpen: false,
      historySearchQuery: '',
      isPinball: isPinball,
      isFullscreenGame: isPinball || isMinesweeper,
      isMinesweeper: isMinesweeper,
      isCommandPrompt: isCommandPrompt,
      hideBrowserUI: isPinball || isMinesweeper || isCommandPrompt,
      favorites: [
        { name: 'Github', url: 'https://github.com/gamerc0der' },
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

{#if contextMenu.visible}
  <div
    class="context-menu"
    style="left: {contextMenu.x}px; top: {contextMenu.y}px;"
    role="menu"
    tabindex="-1"
    on:click|stopPropagation={() => {}}
    on:keydown={(e) => {
      if (e.key === 'Escape') hideContextMenu();
    }}
  >
    <div
      class="context-menu-item"
      role="menuitem"
      tabindex="0"
      on:click={() => handleContextMenuAction('refresh')}
      on:keydown={(e) => { if (e.key === 'Enter' || e.key === ' ') handleContextMenuAction('refresh'); }}
    >
      <span class="context-menu-icon">🔄</span>
      Refresh
    </div>
    {#if contextMenu.target && contextMenu.target.closest('.window')}
      <div class="context-menu-separator"></div>
      <div
        class="context-menu-item"
        role="menuitem"
        tabindex="0"
        on:click={() => handleContextMenuAction('minimize')}
        on:keydown={(e) => { if (e.key === 'Enter' || e.key === ' ') handleContextMenuAction('minimize'); }}
      >
        <span class="context-menu-icon">_</span>
        Minimize
      </div>
      <div
        class="context-menu-item"
        role="menuitem"
        tabindex="0"
        on:click={() => handleContextMenuAction('maximize')}
        on:keydown={(e) => { if (e.key === 'Enter' || e.key === ' ') handleContextMenuAction('maximize'); }}
      >
        <span class="context-menu-icon">⬜</span>
        Maximize
      </div>
      <div
        class="context-menu-item"
        role="menuitem"
        tabindex="0"
        on:click={() => handleContextMenuAction('close')}
        on:keydown={(e) => { if (e.key === 'Enter' || e.key === ' ') handleContextMenuAction('close'); }}
      >
        <span class="context-menu-icon">✕</span>
        Close
      </div>
    {/if}
  </div>
{/if}

{#each windows as window (window.id)}
  {#if !window.isMinimized}
    <div class="window"
       class:maximized={window.isMaximized}
       class:fullscreen-game={window.isFullscreenGame}
       data-window-id={window.id}
       style="position: fixed; left: {window.windowPos.x}px; top: {window.windowPos.y}px; width: {window.isMaximized ? '100vw' : window.originalSize.width + 'px'}; height: {window.isMaximized ? '100vh' : (window.isFullscreenGame ? window.originalSize.height + 'px' : '400px')}; z-index: 2000;">
    <div class="title-bar" role="button" tabindex="0">
      <div class="title-bar-text">
        {#if window.isPinball}
          Space Cadet Pinball 3D
        {:else if window.isMinesweeper}
          Minesweeper
        {:else if window.isCommandPrompt}
          Command Prompt
        {:else}
          Microsoft Internet Explorer
        {/if}
      </div>

      <div class="title-bar-controls">
        {#if !window.hideBrowserUI}
          <button aria-label="Minimize" on:click={() => minimizeWindow(window.id)}></button>
        {/if}
        <button aria-label="Maximize" on:click={() => maximizeWindow(window.id)}></button>
        <button aria-label="Close" on:click={() => closeWindow(window.id)}></button>
      </div>
    </div>
    {#if !window.hideBrowserUI}
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
        <button class="nav-btn" on:click={() => navigateToUrl(window.id, 'https://www.msn.com/history')}>
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
    {/if}

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
      <div class="webpage-content" class:fullscreen-game-content={window.isFullscreenGame || window.isCommandPrompt}>
      {#if window.currentUrl === 'https://www.msn.com/about'}
        <div class="about-page">
          <h1 class="about-title">About This Website</h1>
          <div class="about-content">
            <button class="about-btn" on:click={() => navigateToUrl(window.id, 'https://www.msn.com/')}>Profile</button>
            <button class="about-btn" on:click={() => navigateToUrl(window.id, 'https://www.msn.com/projects')}>Projects</button>
          </div>
        </div>
      {:else if window.currentUrl === 'https://www.msn.com/'}
        <h1 class="name-header">gamerc0der</h1>
        <div class="bio-section">bio goes here</div>
        <div class="social-icons">
          <a href="https://github.com" target="_blank" rel="noopener noreferrer">
            <img src="https://github.githubassets.com/images/modules/site/icons/footer/github-mark.svg" alt="GitHub" class="social-icon github-icon">
          </a>
          <a href="https://slack.com" target="_blank" rel="noopener noreferrer">
            <img src="https://a.slack-edge.com/80588/marketing/img/icons/icon_slack_hash_colored.png" alt="Slack" class="social-icon">
          </a>
        </div>

        <div class="languages-section">
          <h2 class="languages-title">Programming Languages</h2>
          <div class="languages-grid">
            <div class="language-item">
              <i class="devicon-react-original language-icon"></i>
              <span class="language-name">React</span>
            </div>
            <div class="language-item">
              <i class="devicon-html5-plain language-icon"></i>
              <span class="language-name">HTML</span>
            </div>
            <div class="language-item">
              <i class="devicon-javascript-plain language-icon"></i>
              <span class="language-name">JavaScript</span>
            </div>
            <div class="language-item">
              <i class="devicon-css3-plain language-icon"></i>
              <span class="language-name">CSS</span>
            </div>
          </div>
          <div class="view-more-container">
            <button class="view-more-link" on:click={() => navigateToUrl(window.id, 'https://www.msn.com/languages')}>See All Languages</button>
          </div>
        </div>

        <button class="projects-btn" on:click={() => navigateToUrl(window.id, 'https://www.msn.com/projects')}>Projects</button>
      {:else if window.currentUrl === 'https://www.msn.com/projects'}
        <div class="projects-page">
          <h1 class="projects-title">My Projects</h1>

          <div class="languages-section">
            <h2 class="languages-title">Programming Languages</h2>
            <div class="languages-grid">
              <div class="language-item">
                <i class="devicon-react-original language-icon"></i>
                <span class="language-name">React</span>
              </div>
              <div class="language-item">
                <i class="devicon-html5-plain language-icon"></i>
                <span class="language-name">HTML</span>
              </div>
              <div class="language-item">
                <i class="devicon-javascript-plain language-icon"></i>
                <span class="language-name">JavaScript</span>
              </div>
              <div class="language-item">
                <i class="devicon-css3-plain language-icon"></i>
                <span class="language-name">CSS</span>
              </div>
            </div>
          </div>

          <div class="search-container">
            <input
              type="text"
              class="search-input"
              placeholder="Search projects..."
              bind:value={searchQuery}
            />
          </div>
          {#if isLoading}
            <div class="loading">Loading projects...</div>
          {:else}
            <div class="projects-grid">
              {#each filteredProjects as project}
                <a class="project-card" href={project.html_url} target="_blank" rel="noopener noreferrer">
                  <h3 class="project-name">{project.name}</h3>
                  <p class="project-description">{project.description || 'No description available'}</p>
                  <div class="project-meta">
                    {#if project.language}
                      <span class="project-language">{project.language}</span>
                    {/if}
                  </div>
                  <div class="project-updated">Updated {new Date(project.updated_at).toLocaleDateString()}</div>
                </a>
              {/each}
            </div>
            {#if filteredProjects.length === 0 && !isLoading}
              <div class="no-results">No projects found matching your search.</div>
            {/if}
          {/if}
        </div>
      {:else if window.currentUrl === 'https://www.msn.com/languages'}
        <div class="languages-page">
          <h1 class="languages-main-title">Programming Languages</h1>

          <div class="languages-showcase">
            <div class="languages-grid">
              <div class="language-item">
                <i class="devicon-react-original language-icon"></i>
                <span class="language-name">React</span>
              </div>
              <div class="language-item">
                <i class="devicon-html5-plain language-icon"></i>
                <span class="language-name">HTML</span>
              </div>
              <div class="language-item">
                <i class="devicon-javascript-plain language-icon"></i>
                <span class="language-name">JavaScript</span>
              </div>
              <div class="language-item">
                <i class="devicon-css3-plain language-icon"></i>
                <span class="language-name">CSS</span>
              </div>
              <div class="language-item">
                <i class="devicon-typescript-plain language-icon"></i>
                <span class="language-name">TypeScript</span>
              </div>
              <div class="language-item">
                <i class="devicon-python-plain language-icon"></i>
                <span class="language-name">Python</span>
              </div>
              <div class="language-item">
                <i class="devicon-lua-plain language-icon"></i>
                <span class="language-name">Lua</span>
              </div>
            </div>
          </div>

          <div class="back-container">
            <button class="back-btn" on:click={() => navigateToUrl(window.id, 'https://www.msn.com/')}>
              <span class="back-icon">←</span>
              Back
            </button>
          </div>
        </div>
      {:else if window.currentUrl === 'https://www.msn.com/update'}
        <div class="update-message">
          No Updates Available
        </div>
      {:else if window.currentUrl === 'https://www.msn.com/history'}
        <div class="history-page">
          <h1 class="history-title">Browsing History</h1>
          <div class="search-container">
            <input
              type="text"
              class="search-input"
              placeholder="Search history..."
              bind:value={window.historySearchQuery}
            />
          </div>
          <div class="history-content">
            {#if window.urlHistory.length > 0}
              {@const filteredHistory = window.urlHistory.filter(url => url.toLowerCase().includes(window.historySearchQuery.toLowerCase()))}
              {#if filteredHistory.length > 0}
                <div class="history-list">
                  {#each filteredHistory as url}
                    {@const originalIndex = window.urlHistory.indexOf(url)}
                    <div class="history-item {originalIndex === window.historyIndex ? 'current' : ''}" role="button" tabindex="0" on:click={() => navigateToUrl(window.id, url)} on:keydown={(e) => { if (e.key === 'Enter' || e.key === ' ') navigateToUrl(window.id, url); }}>
                      <span class="history-url">{url}</span>
                      {#if originalIndex === window.historyIndex}
                        <span class="current-indicator">(Current)</span>
                      {/if}
                    </div>
                  {/each}
                </div>
              {:else}
                <div class="no-history">No history items match your search.</div>
              {/if}
            {:else}
              <div class="no-history">No browsing history available.</div>
            {/if}
          </div>
        </div>
      {:else if window.currentUrl === '/help'}
        <div class="help-page">
          <h1 class="help-title">Help Documentation</h1>
          <div class="help-content">
            <h2>Getting Started</h2>
            <p>Lorem ipsum dolor sit amet, consectetur adipiscing elit. Sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat.</p>

            <h2>Available Commands</h2>
            <ul>
              <li><strong>help</strong> - Display this help documentation</li>
              <li><strong>time</strong> - Show the current date and time</li>
              <li><strong>user</strong> - Display current user information</li>
              <li><strong>logout</strong> - Log out and restart the system</li>
            </ul>

            <h2>Navigation</h2>
            <p>Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.</p>

            <h2>Troubleshooting</h2>
            <p>Sed ut perspiciatis unde omnis iste natus error sit voluptatem accusantium doloremque laudantium, totam rem aperiam, eaque ipsa quae ab illo inventore veritatis et quasi architecto beatae vitae dicta sunt explicabo.</p>
          </div>
        </div>
      {:else}
        <iframe
          src={window.currentUrl}
          class="web-iframe"
          title="Web content"
          tabindex="0"
          sandbox={window.isCommandPrompt ? "allow-scripts allow-same-origin allow-forms allow-popups allow-modals allow-pointer-lock allow-top-navigation" : "allow-scripts allow-same-origin allow-forms allow-popups"}
        ></iframe>
      {/if}
      </div>
    </div>

    {#if !window.hideBrowserUI}
    <div class="status-bar">
      <span>Done</span>
      <span>My Computer</span>
    </div>
    {/if}
  </div>
  {/if}
{/each}

{#if runMenuOpen}
<div class="run-window" style="position: fixed; left: {runWindowPos.x}px; top: {runWindowPos.y}px;">
  <div class="title-bar">
    <div class="title-bar-text">Run</div>
    <div class="title-bar-controls">
      <button aria-label="Close" on:click={toggleRunMenu}></button>
    </div>
  </div>
  <div class="run-content">
    <div class="run-label">Type the name of a program, folder, document, or Internet resource, and Windows will open it for you.</div>
    <div class="run-input-container">
      <label class="run-input-label" for="run-input-field">Open:</label>
      <input type="text" class="run-input" id="run-input-field" placeholder="Type here..." bind:value={runCommand} on:keydown={(e) => { if (e.key === 'Enter') executeRunCommand(); }}>
    </div>
    <div class="run-buttons">
      <button class="run-ok-btn" on:click={executeRunCommand}>OK</button>
      <button class="run-cancel-btn" on:click={toggleRunMenu}>Cancel</button>
    </div>
  </div>
</div>
{/if}

{#each messageWindows as messageWindow (messageWindow.id)}
  {#if !messageWindow.isMinimized}
    <div class="message-window" data-window-id={messageWindow.id} style="position: fixed; left: {messageWindow.windowPos.x}px; top: {messageWindow.windowPos.y}px;">
      <div class="title-bar">
        <div class="title-bar-text">{messageWindow.title}</div>
        <div class="title-bar-controls">
          <button aria-label="Close" on:click={() => closeMessageWindow(messageWindow.id)}></button>
        </div>
      </div>
      <div class="message-content">
        <div class="message-text">{messageWindow.message}</div>
      </div>
    </div>
  {/if}
{/each}

<StartMenu {openNewWindow} openRunMenu={toggleRunMenu} />

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

  .webpage-content.fullscreen-game-content {
    padding: 0;
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

  .about-page {
    padding: 20px;
    font-family: 'MS Sans Serif', sans-serif;
    color: #000;
  }

  .about-title {
    font-size: 18px;
    font-weight: bold;
    margin-bottom: 20px;
    color: #000;
  }

  .about-btn {
    padding: 8px 16px;
    background: #c0c0c0;
    border: 2px outset #c0c0c0;
    font-family: 'MS Sans Serif', sans-serif;
    font-size: 12px;
    cursor: pointer;
    margin: 5px;
  }

  .about-btn:hover {
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

  .help-page {
    padding: 20px;
    font-family: 'MS Sans Serif', sans-serif;
    color: #000;
  }

  .help-title {
    font-size: 18px;
    font-weight: bold;
    margin-bottom: 20px;
    color: #000;
  }

  .help-content h2 {
    font-size: 14px;
    font-weight: bold;
    margin-top: 20px;
    margin-bottom: 10px;
    color: #000;
  }

  .help-content p {
    font-size: 12px;
    line-height: 1.5;
    margin-bottom: 15px;
    color: #000;
  }

  .help-content ul {
    font-size: 12px;
    line-height: 1.5;
    margin-bottom: 15px;
    color: #000;
  }

  .help-content li {
    margin-bottom: 5px;
  }

  .history-page {
    padding: 20px;
    font-family: 'MS Sans Serif', sans-serif;
    color: #000;
  }

  .history-title {
    font-size: 18px;
    font-weight: bold;
    margin-bottom: 20px;
    color: #000;
  }

  .history-content {
    max-height: 300px;
    overflow-y: auto;
  }

  .history-list {
    border: 1px inset #c0c0c0;
    background: white;
  }

  .history-item {
    padding: 8px 12px;
    border-bottom: 1px solid #c0c0c0;
    cursor: pointer;
    display: flex;
    justify-content: space-between;
    align-items: center;
    font-family: 'MS Sans Serif', sans-serif;
    font-size: 12px;
  }

  .history-item:hover {
    background: #e0e0e0;
  }

  .history-item.current {
    background: #000080;
    color: white;
  }

  .history-url {
    flex: 1;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
  }

  .current-indicator {
    font-size: 11px;
    font-weight: bold;
    color: #ffff00;
  }

  .no-history {
    text-align: center;
    font-family: 'MS Sans Serif', sans-serif;
    font-size: 14px;
    color: #666;
    padding: 40px;
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

  .projects-page {
    padding: 10px;
  }

  .projects-title {
    font-family: 'MS Sans Serif', sans-serif;
    font-size: 16px;
    font-weight: bold;
    margin-bottom: 10px;
    color: #000;
  }

  .search-container {
    margin-bottom: 10px;
  }

  .search-input {
    width: 100%;
    padding: 3px 4px;
    border: 1px inset #c0c0c0;
    background: white;
    color: #000;
    font-family: 'MS Sans Serif', sans-serif;
    font-size: 12px;
  }

  .loading {
    text-align: center;
    font-family: 'MS Sans Serif', sans-serif;
    font-size: 12px;
    color: #000;
    padding: 20px;
  }

  .projects-grid {
    display: grid;
    grid-template-columns: 1fr;
    gap: 8px;
    max-height: 300px;
    overflow-y: auto;
    border: 1px inset #c0c0c0;
    background: white;
    padding: 8px;
  }

  .project-card {
    background: #c0c0c0;
    border: 1px outset #c0c0c0;
    padding: 8px;
    cursor: pointer;
    font-family: 'MS Sans Serif', sans-serif;
  }

  .project-card:hover {
    background: #d0d0d0;
  }

  .project-name {
    font-size: 14px;
    font-weight: bold;
    margin: 0 0 4px 0;
    color: #000;
  }

  .project-description {
    font-size: 12px;
    color: #000;
    margin: 0 0 6px 0;
    line-height: 1.3;
    display: -webkit-box;
    -webkit-line-clamp: 2;
    line-clamp: 2;
    -webkit-box-orient: vertical;
    overflow: hidden;
  }

  .project-meta {
    font-size: 11px;
    color: #000;
  }

  .project-language {
    background: #008000;
    color: white;
    padding: 1px 4px;
    font-weight: bold;
  }


  .project-updated {
    font-size: 11px;
    color: #666;
  }

  .no-results {
    text-align: center;
    font-family: 'MS Sans Serif', sans-serif;
    font-size: 12px;
    color: #000;
    padding: 20px;
  }

  .languages-page {
    padding: 20px;
  }

  .languages-main-title {
    font-family: 'MS Sans Serif', sans-serif;
    font-size: 20px;
    font-weight: bold;
    margin-bottom: 20px;
    color: #000;
    text-align: center;
  }

  .languages-showcase {
    max-width: 800px;
    margin: 0 auto;
  }


  .languages-section {
    margin-bottom: 20px;
  }

  .languages-title {
    font-family: 'MS Sans Serif', sans-serif;
    font-size: 14px;
    font-weight: bold;
    margin-bottom: 10px;
    color: #000;
  }

  .languages-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(120px, 1fr));
    gap: 8px;
    margin-bottom: 15px;
  }

  .language-item {
    background: #c0c0c0;
    border: 1px outset #c0c0c0;
    padding: 8px;
    display: flex;
    align-items: center;
    gap: 8px;
    font-family: 'MS Sans Serif', sans-serif;
    font-size: 12px;
    color: #000;
  }

  .language-item:hover {
    background: #d0d0d0;
  }

  .language-icon {
    font-size: 24px;
    color: #000;
  }

  .language-name {
    flex: 1;
  }

  .view-more-container {
    margin-top: 15px;
    text-align: center;
  }

  .view-more-link {
    padding: 6px 12px;
    background: #c0c0c0;
    border: 2px outset #c0c0c0;
    font-family: 'MS Sans Serif', sans-serif;
    font-size: 12px;
    cursor: pointer;
    color: #000;
    text-decoration: none;
    display: inline-block;
  }

  .view-more-link:hover {
    border: 2px inset #c0c0c0;
    text-decoration: none;
  }

  .back-container {
    margin-top: 20px;
    text-align: center;
  }

  .back-btn {
    padding: 6px 12px;
    background: #c0c0c0;
    border: 2px outset #c0c0c0;
    font-family: 'MS Sans Serif', sans-serif;
    font-size: 12px;
    cursor: pointer;
    color: #000;
    display: inline-flex;
    align-items: center;
    gap: 4px;
  }

  .back-btn:hover {
    border: 2px inset #c0c0c0;
  }

  .back-icon {
    font-size: 14px;
    line-height: 1;
  }

  .run-window {
    background: #c0c0c0;
    border: 2px solid black;
    box-shadow: 4px 4px 8px rgba(0,0,0,0.3);
    transition: width 0.3s ease-in-out, height 0.3s ease-in-out, border 0.3s ease-in-out, box-shadow 0.3s ease-in-out;
    display: flex;
    flex-direction: column;
    width: 400px;
    z-index: 2000;
  }


  .run-content {
    padding: 16px;
  }

  .run-label {
    font-family: 'MS Sans Serif', sans-serif;
    font-size: 14px;
    color: #000;
    margin-bottom: 12px;
  }

  .run-input-container {
    display: flex;
    align-items: center;
    margin-bottom: 12px;
  }

  .run-input-label {
    font-family: 'MS Sans Serif', sans-serif;
    font-size: 12px;
    color: #000;
    margin-right: 8px;
    width: 40px;
  }

  .run-input {
    flex: 1;
    padding: 2px 4px;
    border: 2px inset #c0c0c0;
    background: white;
    font-family: 'MS Sans Serif', sans-serif;
    font-size: 12px;
  }

  .run-buttons {
    display: flex;
    justify-content: flex-end;
    gap: 8px;
  }

  .run-ok-btn,
  .run-cancel-btn {
    padding: 4px 12px;
    background: #c0c0c0;
    border: 2px outset #c0c0c0;
    font-family: 'MS Sans Serif', sans-serif;
    font-size: 12px;
    cursor: pointer;
  }

  .run-ok-btn:hover,
  .run-cancel-btn:hover {
    border: 2px inset #c0c0c0;
  }

  .message-window {
    background: #c0c0c0;
    border: 2px solid black;
    box-shadow: 4px 4px 8px rgba(0,0,0,0.3);
    transition: width 0.3s ease-in-out, height 0.3s ease-in-out, border 0.3s ease-in-out, box-shadow 0.3s ease-in-out;
    display: flex;
    flex-direction: column;
    width: 350px;
    z-index: 2000;
  }

  .message-content {
    padding: 16px;
    border: 2px inset #c0c0c0;
    display: flex;
    align-items: center;
    justify-content: center;
    min-height: 60px;
  }

  .message-text {
    font-family: 'MS Sans Serif', sans-serif;
    font-size: 12px;
    color: #000;
    text-align: center;
    line-height: 1.4;
  }

  .name-header {
    font-family: 'MS Sans Serif', sans-serif;
    font-size: 24px;
    font-weight: bold;
    color: #000;
    margin-bottom: 20px;
    text-align: center;
  }

  .bio-section {
    font-family: 'MS Sans Serif', sans-serif;
    font-size: 14px;
    color: #000;
    margin-bottom: 20px;
    text-align: center;
    max-width: 400px;
    margin-left: auto;
    margin-right: auto;
    line-height: 1.4;
    letter-spacing: -0.5px;
  }

  .context-menu {
    position: fixed;
    background: #c0c0c0;
    border: 2px outset #c0c0c0;
    box-shadow: 2px 2px 4px rgba(0, 0, 0, 0.5);
    z-index: 10000;
    min-width: 120px;
    font-family: 'MS Sans Serif', sans-serif;
    font-size: 12px;
    padding: 2px 0;
  }

  .context-menu-item {
    display: flex;
    align-items: center;
    gap: 6px;
    padding: 4px 12px;
    cursor: pointer;
    color: #000;
    user-select: none;
  }

  .context-menu-item:hover {
    background: #000080;
    color: #fff;
  }

  .context-menu-icon {
    width: 16px;
    text-align: center;
    font-size: 10px;
  }

  .context-menu-separator {
    height: 1px;
    background: #808080;
    margin: 4px 2px;
    border-bottom: 1px solid #fff;
  }

</style>
