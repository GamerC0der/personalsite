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

  let winampOpen = false;

  let runMenuOpen = false;
  let runWindowPos = { x: 100, y: 100 };
  let runCommand = '';

  function openNewWindow(url = 'https://www.msn.com/') {
    const isPinball = url === 'https://98.js.org/programs/pinball/space-cadet.html';
    const isMinesweeper = url === 'https://98plus.js.org/programs/minesweeper/index.html';
    const isCommandPrompt = url === 'https://98.js.org/programs/command/index.html';
    const isExplorer = url === 'explorer://';

    if (isMobile) {
      windows = [];
    }

    const newWindow = {
      id: nextWindowId++,
      windowPos: { x: 0, y: 0 },
      currentUrl: url,
      inputUrl: url,
      urlHistory: [url],
      historyIndex: 0,
      isMinimized: false,
      isMaximized: isPinball || (isMobile && !isCommandPrompt),
      originalSize: (isPinball || isMinesweeper) ? { width: '100vw', height: 'calc(100vh - 42px)' } : isExplorer ? { width: 600, height: 400 } : { width: 600, height: 'auto' },
      originalPos: { x: 0, y: 0 },
      sidebarOpen: false,
      historySearchQuery: '',
      isPinball: isPinball,
      isFullscreenGame: isPinball || isMinesweeper,
      isMinesweeper: isMinesweeper,
      isCommandPrompt: isCommandPrompt,
      isExplorer: isExplorer,
      hideBrowserUI: isPinball || isMinesweeper || isCommandPrompt || isExplorer,
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

  let messageWindows = [];
  let nextMessageWindowId = 1;

  let contextMenu = {
    visible: false,
    x: 0,
    y: 0,
    target: null
  };

  let isMobile = false;
  let timeInterval;

  let isIdle = false;
  let idleTimeout;
  let lastMouseMove = Date.now();
  const screensaverUrl = 'https://98.js.org/programs/pipes/#{"hideUI":true}';

  function updateTime() {
    currentTime = new Date().toLocaleTimeString();
  }

  function checkMobile() {
    isMobile = window.innerWidth <= 768 || window.innerHeight <= 600;
  }

  onMount(() => {
    console.log('jQuery version:', window.$.fn.jquery);
    console.log('jQuery UI available:', typeof window.$.fn.draggable === 'function');

    checkMobile();
    window.addEventListener('resize', checkMobile);

    if (isMobile) {
      windows[0].isMaximized = true;
      windows[0].originalPos = { x: window.innerWidth / 2 - 300, y: window.innerHeight / 2 - 200 };
      windows[0].windowPos = { x: 0, y: 0 };
    } else {
      windows[0].windowPos.x = window.innerWidth / 2 - 300;
      windows[0].windowPos.y = window.innerHeight / 2 - 200;
    }
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

    document.addEventListener('contextmenu', showContextMenu);
    document.addEventListener('click', hideContextMenu);

    const resetIdleTimer = () => {
      lastMouseMove = Date.now();
      if (isIdle) {
        isIdle = false;
        clearTimeout(idleTimeout);
        idleTimeout = setTimeout(() => {
          if (Date.now() - lastMouseMove >= 15000) {
            isIdle = true;
          }
        }, 15000);
      } else {
        clearTimeout(idleTimeout);
        idleTimeout = setTimeout(() => {
          if (Date.now() - lastMouseMove >= 15000) {
            isIdle = true;
          }
        }, 15000);
      }
    };

    document.addEventListener('mousemove', resetIdleTimer);
    document.addEventListener('mousedown', resetIdleTimer);
    document.addEventListener('keydown', resetIdleTimer);
    document.addEventListener('scroll', resetIdleTimer);

    idleTimeout = setTimeout(() => {
      if (Date.now() - lastMouseMove >= 15000) {
        isIdle = true;
      }
    }, 15000);
  });

  onDestroy(() => {
    if (timeInterval) {
      clearInterval(timeInterval);
    }
    if (idleTimeout) {
      clearTimeout(idleTimeout);
    }
    if (typeof document !== 'undefined') {
      document.removeEventListener('contextmenu', showContextMenu);
      document.removeEventListener('click', hideContextMenu);
    }
    if (typeof window !== 'undefined') {
      window.removeEventListener('resize', checkMobile);
    }
  });

  function minimizeWindow(windowId) {
    if (isMobile) return;
    windows = windows.map(w =>
      w.id === windowId
        ? { ...w, isMinimized: true }
        : w
    );
  }

  function maximizeWindow(windowId) {
    if (isMobile) {
      windows = windows.map(w => {
        if (w.id === windowId && !w.isMaximized) {
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
        }
        return w;
      });
      return;
    }

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
    if (isMobile) return;
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

  function openFactorioPage() {
    window.open('https://factorio.com/', '_blank');
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
    <div class="context-menu-item" role="menuitem" tabindex="0">
      <span class="context-menu-text"><u>A</u>ctive Desktop</span>
      <span class="context-menu-arrow">▶</span>
    </div>
    <div class="context-menu-item" role="menuitem" tabindex="0">
      <span class="context-menu-text"><u>A</u>rrange Icons</span>
      <span class="context-menu-arrow">▶</span>
    </div>
    <div class="context-menu-item" role="menuitem" tabindex="0">
      <span class="context-menu-text">L<u>i</u>ne Up Icons</span>
    </div>
    <div class="context-menu-separator"></div>
    <div
      class="context-menu-item"
      role="menuitem"
      tabindex="0"
      on:click={() => handleContextMenuAction('refresh')}
      on:keydown={(e) => { if (e.key === 'Enter' || e.key === ' ') handleContextMenuAction('refresh'); }}
    >
      <span class="context-menu-text">R<u>e</u>fresh</span>
    </div>
    <div class="context-menu-separator"></div>
    <div class="context-menu-item disabled" role="menuitem" aria-disabled="true">
      <span class="context-menu-text"><u>P</u>aste</span>
    </div>
    <div class="context-menu-item disabled" role="menuitem" aria-disabled="true">
      <span class="context-menu-text">Paste <u>S</u>hortcut</span>
    </div>
    <div class="context-menu-separator"></div>
    <div class="context-menu-item" role="menuitem" tabindex="0">
      <span class="context-menu-text"><u>N</u>ew</span>
      <span class="context-menu-arrow">▶</span>
    </div>
    <div class="context-menu-item" role="menuitem" tabindex="0">
      <span class="context-menu-text"><u>P</u>roperties</span>
    </div>
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
        {:else if window.isExplorer}
          My Computer
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
          <i class="fas fa-rotate-right nav-icon"></i>
          <span class="nav-text">Refresh</span>
        </button>
        <button class="nav-btn" on:click={() => navigateToUrl(window.id, 'https://www.msn.com/')}>
          <i class="fas fa-home nav-icon"></i>
          <span class="nav-text">Home</span>
        </button>
        <button class="nav-btn" on:click={() => toggleSidebar(window.id)}>
          <i class="fas fa-star nav-icon"></i>
          <span class="nav-text">Favorites</span>
        </button>
        <button class="nav-btn" on:click={() => navigateToUrl(window.id, 'https://www.msn.com/history')}>
          <i class="fas fa-history nav-icon"></i>
          <span class="nav-text">History</span>
        </button>
      </div>
    </div>

    <div class="address-bar">
      <div class="address-label">Address:</div>
      <input type="text" class="address-input" bind:value={window.inputUrl} on:keydown={(e) => { if (e.key === 'Enter') goToUrl(window.id); }}>
      <button class="go-btn" on:click={() => goToUrl(window.id)}>
        <svg class="go-icon" viewBox="0 0 16 16" fill="none" xmlns="http://www.w3.org/2000/svg">
          <path d="M4 8 L12 8 M10 6 L12 8 L10 10" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" fill="none"/>
        </svg>
        Go
      </button>
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
      <div class="webpage-content" class:fullscreen-game-content={window.isFullscreenGame || window.isCommandPrompt || window.isWinAmp || window.isExplorer}>
      {#if window.currentUrl === 'https://www.msn.com/about'}
        <div class="about-page">
          <h1 class="about-title">About This Website</h1>
          <div class="about-content">
            <button class="about-btn" on:click={() => navigateToUrl(window.id, 'https://www.msn.com/')}>Profile</button>
            <button class="about-btn" on:click={() => navigateToUrl(window.id, 'https://www.msn.com/projects')}>Projects</button>
          </div>
        </div>
      {:else if window.currentUrl === 'https://www.msn.com/'}
        <h1 class="name-header">Hi<span class="light-char">,</span> I'm Matthew<span class="light-char">!</span></h1>
        <p class="subtitle-text">I'm a coder from California, homeschooled, and my favorite game is <span class="underline-text clickable-factorio" on:click={openFactorioPage}>Factorio</span>!</p>
        <p class="tech-stack-text"><i class="devicon-html5-plain tech-icon"></i><i class="devicon-css3-plain tech-icon"></i><i class="devicon-javascript-plain tech-icon"></i><i class="devicon-lua-plain tech-icon"></i><i class="devicon-python-plain tech-icon"></i><i class="devicon-react-plain tech-icon"></i><i class="devicon-svelte-plain tech-icon"></i><i class="devicon-nextjs-plain tech-icon"></i>I enjoy coding in Svelte and NextJS</p>
        <p class="social-text">I'm often on Vercel and Github.<i class="devicon-vercel-plain social-icon"></i><i class="devicon-github-plain social-icon"></i><i class="devicon-figma-plain social-icon"></i><i class="devicon-vscode-plain social-icon"></i><i class="devicon-slack-plain social-icon"></i><i class="devicon-ngrok-plain social-icon"></i><i class="devicon-pypi-plain social-icon"></i></p>
        <p class="reading-text"><img src="https://upload.wikimedia.org/wikipedia/en/d/d9/Foundation_-_Isaac_Asimov_%28Gnome_1951%29.jpg" alt="Foundation by Isaac Asimov" class="book-icon"><img src="https://upload.wikimedia.org/wikipedia/en/d/d5/I_robot.jpg" alt="I, Robot by Isaac Asimov" class="book-icon">I love reading Isaac Asimov's Robot and Foundation series</p>
        <p class="ai-tools-text">My Favorite AI Tools are Cursor and Perplexity<svg class="ai-icon" viewBox="0 0 24 24" fill="currentColor"><path d="M22.106 5.68L12.5.135a.998.998 0 00-.998 0L1.893 5.68a.84.84 0 00-.419.726v11.186c0 .3.16.577.42.727l9.607 5.547a.999.999 0 00.998 0l9.608-5.547a.84.84 0 00.42-.727V6.407a.84.84 0 00-.42-.726zm-.603 1.176L12.228 22.92c-.063.108-.228.064-.228-.061V12.34a.59.59 0 00-.295-.51l-9.11-5.26c-.107-.062-.063-.228.062-.228h18.55c.264 0 .428.286.296.514z"/></svg><svg class="ai-icon" viewBox="0 0 24 24" fill="currentColor"><path d="M19.785 0v7.272H22.5V17.62h-2.935V24l-7.037-6.194v6.145h-1.091v-6.152L4.392 24v-6.465H1.5V7.188h2.884V0l7.053 6.494V.19h1.09v6.49L19.786 0zm-7.257 9.044v7.319l5.946 5.234V14.44l-5.946-5.397zm-1.099-.08l-5.946 5.398v7.235l5.946-5.234V8.965zm8.136 7.58h1.844V8.349H13.46l6.105 5.54v2.655zm-8.982-8.28H2.59v8.195h1.8v-2.576l6.192-5.62zM5.475 2.476v4.71h5.115l-5.115-4.71zm13.219 0l-5.115 4.71h5.115v-4.71z"/></svg><svg fill="currentColor" fill-rule="evenodd" height="64" viewBox="0 0 24 24" width="64" xmlns="http://www.w3.org/2000/svg" style="flex: 0 0 auto; line-height: 1;" class="ai-icon"><title>OpenAI</title><path d="M21.55 10.004a5.416 5.416 0 00-.478-4.501c-1.217-2.09-3.662-3.166-6.05-2.66A5.59 5.59 0 0010.831 1C8.39.995 6.224 2.546 5.473 4.838A5.553 5.553 0 001.76 7.496a5.487 5.487 0 00.691 6.5 5.416 5.416 0 00.477 4.502c1.217 2.09 3.662 3.165 6.05 2.66A5.586 5.586 0 0013.168 23c2.443.006 4.61-1.546 5.361-3.84a5.553 5.553 0 003.715-2.66 5.488 5.488 0 00-.693-6.497v.001zm-8.381 11.558a4.199 4.199 0 01-2.675-.954c.034-.018.093-.05.132-.074l4.44-2.53a.71.71 0 00.364-.623v-6.176l1.877 1.069c.02.01.033.029.036.05v5.115c-.003 2.274-1.87 4.118-4.174 4.123zM4.192 17.78a4.059 4.059 0 01-.498-2.763c.032.02.09.055.131.078l4.44 2.53c.225.13.504.13.73 0l5.42-3.088v2.138a.068.068 0 01-.027.057L9.9 19.288c-1.999 1.136-4.552.46-5.707-1.51h-.001zM3.023 8.216A4.15 4.15 0 015.198 6.41l-.002.151v5.06a.711.711 0 00.364.624l5.42 3.087-1.876 1.07a.067.067 0 01-.063.005l-4.489-2.559c-1.995-1.14-2.679-3.658-1.53-5.63h.001zm15.417 3.54l-5.42-3.088L14.896 7.6a.067.067 0 01.063-.006l4.489 2.557c1.998 1.14 2.683 3.662 1.529 5.633a4.163 4.163 0 01-2.174 1.807V12.38a.71.71 0 00-.363-.623zm1.867-2.773a6.04 6.04 0 00-.132-.078l-4.44-2.53a.731.731 0 00-.729 0l-5.42 3.088V7.325a.068.068 0 01.027-.057L14.1 4.713c2-1.137 4.555-.46 5.707 1.513.487.833.664 1.809.499 2.757h.001zm-11.741 3.81l-1.877-1.068a.065.065 0 01-.036-.051V6.559c.001-2.277 1.873-4.122 4.181-4.12.976 0 1.92.338 2.671.954-.034.018-.092.05-.131.073l-4.44 2.53a.71.71 0 00-.365.623l-.003 6.173v.002zm1.02-2.168L12 9.25l2.414 1.375v2.75L12 14.75l-2.415-1.375v-2.75z"></path></svg><svg fill="currentColor" fill-rule="evenodd" height="64" viewBox="0 0 24 24" width="64" xmlns="http://www.w3.org/2000/svg" style="flex: 0 0 auto; line-height: 1;" class="ai-icon"><title>Grok</title><path d="M9.27 15.29l7.978-5.897c.391-.29.95-.177 1.137.272.98 2.369.542 5.215-1.41 7.169-1.951 1.954-4.667 2.382-7.149 1.406l-2.711 1.257c3.889 2.661 8.611 2.003 11.562-.953 2.341-2.344 3.066-5.539 2.388-8.42l.006.007c-.983-4.232.242-5.924 2.75-9.383.06-.082.12-.164.179-.248l-3.301 3.305v-.01L9.267 15.292M7.623 16.723c-2.792-2.67-2.31-6.801.071-9.184 1.761-1.763 4.647-2.483 7.166-1.425l2.705-1.25a7.808 7.808 0 00-1.829-1A8.975 8.975 0 005.984 5.83c-2.533 2.536-3.33 6.436-1.962 9.764 1.022 2.487-.653 4.246-2.34 6.022-.599.63-1.199 1.259-1.682 1.925l7.62-6.815"></path></svg><svg class="ai-icon" viewBox="0 0 24 24" fill="currentColor"><path d="M20.616 10.835a14.147 14.147 0 01-4.45-3.001 14.111 14.111 0 01-3.678-6.452.503.503 0 00-.975 0 14.134 14.134 0 01-3.679 6.452 14.155 14.155 0 01-4.45 3.001c-.65.28-1.318.505-2.002.678a.502.502 0 000 .975c.684.172 1.35.397 2.002.677a14.147 14.147 0 014.45 3.001 14.112 14.112 0 013.679 6.453.502.502 0 00.975 0c.172-.685.397-1.351.677-2.003a14.145 14.145 0 013.001-4.45 14.113 14.113 0 016.453-3.678.503.503 0 000-.975 13.245 13.245 0 01-2.003-.678z"></path></svg></p>
        <div style="text-align: center; margin-top: -50px;">
          <button style="padding: 8px 16px; background: #c0c0c0; border: 2px outset #c0c0c0; font-family: 'MS Sans Serif', sans-serif; font-size: 14px; cursor: pointer; color: #000;" on:click={() => navigateToUrl(window.id, 'https://www.msn.com/projects')}>View my Projects</button>
          <p style="font-family: 'MS Sans Serif', sans-serif; font-size: 11px; color: #666; margin: 4px 0 0 0;">Auto Updates!</p>
        </div>
      {:else if window.currentUrl === 'https://www.msn.com/projects'}
        <div class="projects-page">
          <h1 class="projects-title">My Projects</h1>

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
      {:else if window.isWinAmp}
        <div class="winamp-window">
          <iframe
            src="/winamp.html"
            class="winamp-iframe"
            title="WinAMP"
            sandbox="allow-scripts allow-same-origin"
          ></iframe>
        </div>
      {:else if window.isExplorer}
        <div class="explorer-window">
          <div class="explorer-toolbar">
            <div class="explorer-address-bar">
              <span class="address-label">Address:</span>
              <div class="address-display">My Computer</div>
            </div>
          </div>
          <div class="explorer-content">
            <div class="program-item" on:click={() => { openNewWindow('https://www.msn.com/'); closeWindow(window.id); }} on:keydown={(e) => { if (e.key === 'Enter' || e.key === ' ') { openNewWindow('https://www.msn.com/'); closeWindow(window.id); } }} role="button" tabindex="0">
              <img src="https://win98icons.alexmeub.com/icons/png/world-3.png" alt="Internet Explorer" class="program-icon">
              <span class="program-name">Internet Explorer</span>
            </div>
            <div class="program-item" on:click={() => { openNewWindow('https://98.js.org/programs/command/index.html'); closeWindow(window.id); }} on:keydown={(e) => { if (e.key === 'Enter' || e.key === ' ') { openNewWindow('https://98.js.org/programs/command/index.html'); closeWindow(window.id); } }} role="button" tabindex="0">
              <img src="https://upload.wikimedia.org/wikipedia/commons/e/e6/Windows_98_CONSOLE_PROMPT.png" alt="Command Prompt" class="program-icon">
              <span class="program-name">Command Prompt</span>
            </div>
            <div class="program-item" on:click={() => { winampOpen = true; closeWindow(window.id); }} on:keydown={(e) => { if (e.key === 'Enter' || e.key === ' ') { winampOpen = true; closeWindow(window.id); } }} role="button" tabindex="0">
              <img src="https://webamp.org/assets/favicon-BX6eJgec.ico" alt="WinAMP" class="program-icon">
              <span class="program-name">WinAMP</span>
            </div>
          </div>
        </div>
      {:else}
        <iframe
          src={window.currentUrl}
          class="web-iframe"
          title="Web content"
          tabindex="0"
          sandbox={window.isCommandPrompt ? "allow-scripts allow-same-origin allow-forms allow-popups allow-modals allow-pointer-lock allow-top-navigation" : window.isMinesweeper ? "allow-scripts allow-same-origin allow-forms" : "allow-scripts allow-same-origin allow-forms allow-popups"}
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

{#if isIdle}
<div class="screensaver-overlay">
  <iframe
    src={screensaverUrl}
    class="screensaver-iframe"
    title="Screensaver"
    sandbox="allow-scripts allow-same-origin"
  ></iframe>
</div>
{/if}

{#if winampOpen}
<div class="winamp-overlay">
  <iframe
    src="/winamp.html"
    class="winamp-overlay-iframe"
    title="WinAMP"
    sandbox="allow-scripts allow-same-origin"
  ></iframe>
</div>
{/if}

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
    font-size: 16px;
    text-align: center;
    line-height: 18px;
  }

  .nav-btn:disabled .nav-icon {
    color: #808080;
  }

  .nav-text {
    font-size: 11px;
    font-weight: normal;
    color: #333;
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
    display: flex;
    align-items: center;
    gap: 4px;
  }

  .go-icon {
    width: 12px;
    height: 12px;
    display: inline-block;
    image-rendering: pixelated;
    image-rendering: -moz-crisp-edges;
    image-rendering: crisp-edges;
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
    justify-content: flex-start;
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
  .winamp-window {
    width: 100%;
    height: 100%;
    display: flex;
    align-items: center;
    justify-content: center;
  }
  .winamp-iframe {
    width: 100%;
    height: 100%;
    border: none;
    background: transparent;
  }
  .winamp-overlay {
    position: fixed;
    top: 50px;
    left: 50px;
    right: 50px;
    bottom: 50px;
    z-index: 9999;
    pointer-events: none;
  }
  .winamp-overlay-iframe {
    width: 100%;
    height: 100%;
    border: none;
    background: transparent;
    pointer-events: auto;
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
    position: relative;
    top: -5%;
    left: -35%;
    margin-bottom: 20px;
    text-align: center;
  }

  .subtitle-text {
    font-family: 'MS Sans Serif', sans-serif;
    font-size: 16px;
    color: #333;
    position: relative;
    top: -15%;
    left: -2.5%;
    margin-bottom: 20px;
    text-align: center;
    white-space: nowrap;
  }

  .underline-text {
    text-decoration: underline;
  }

  .clickable-factorio {
    cursor: pointer;
    user-select: none;
    transition: all 0.1s ease;
  }

  .clickable-factorio:active {
    transform: scale(0.95);
    opacity: 0.8;
  }

  .tech-stack-text {
    font-family: 'MS Sans Serif', sans-serif;
    font-size: 16px;
    color: #666;
    position: relative;
    top: -25%;
    left: -5%;
    margin-bottom: 20px;
    text-align: right;
    white-space: nowrap;
    max-width: 500px;
    margin-left: auto;
    display: flex;
    align-items: center;
    gap: 8px;
  }

  .tech-icon {
    font-size: 20px;
    color: #666;
  }

  .social-text {
    font-family: 'MS Sans Serif', sans-serif;
    font-size: 16px;
    color: #666;
    position: relative;
    top: -35%;
    left: -27%;
    margin-bottom: 20px;
    text-align: right;
    white-space: nowrap;
    max-width: 500px;
    margin-left: auto;
    display: flex;
    align-items: center;
    gap: 8px;
  }

  .social-icon {
    font-size: 20px;
    color: #666;
  }

  .reading-text {
    font-family: 'MS Sans Serif', sans-serif;
    font-size: 16px;
    color: #666;
    position: relative;
    top: -45%;
    left: -5%;
    margin-bottom: 20px;
    text-align: right;
    white-space: nowrap;
    max-width: 600px;
    margin-left: auto;
    display: flex;
    align-items: center;
    gap: 8px;
  }

  .book-icon {
    width: 24px;
    height: 36px;
    object-fit: cover;
    border: 1px solid #ccc;
    border-radius: 2px;
  }

  .ai-tools-text {
    font-family: 'MS Sans Serif', sans-serif;
    font-size: 16px;
    color: #666;
    position: relative;
    top: -55%;
    left: -17%;
    margin-bottom: 20px;
    text-align: right;
    white-space: nowrap;
    max-width: 600px;
    margin-left: auto;
    display: flex;
    align-items: center;
    gap: 8px;
  }

  .ai-icon {
    width: 20px;
    height: 20px;
  }

  .light-char {
    color: #666;
  }

  .context-menu {
    position: fixed;
    background: #c0c0c0;
    border: 1px solid #808080;
    border-right: 2px solid #000000;
    border-bottom: 2px solid #000000;
    border-top: 1px solid #ffffff;
    border-left: 1px solid #ffffff;
    box-shadow: 1px 1px 0px rgba(0, 0, 0, 0.3);
    z-index: 10000;
    min-width: 180px;
    font-family: 'MS Sans Serif', sans-serif;
    font-size: 11px;
    padding: 2px 0;
  }

  .context-menu-item {
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 2px 20px 2px 18px;
    cursor: pointer;
    color: #000000;
    user-select: none;
    white-space: nowrap;
  }

  .context-menu-item:hover:not(.disabled) {
    background: #000080;
    color: #ffffff;
  }

  .context-menu-item.disabled {
    color: #808080;
    cursor: default;
  }

  .context-menu-text {
    flex: 1;
  }

  .context-menu-arrow {
    margin-left: 20px;
    font-size: 8px;
    color: #000000;
    line-height: 1;
  }

  .context-menu-item:hover:not(.disabled) .context-menu-arrow {
    color: #ffffff;
  }

  .context-menu-separator {
    height: 1px;
    background: #808080;
    margin: 2px 1px;
    border-top: 1px solid #ffffff;
  }

  .screensaver-overlay {
    position: fixed;
    top: 0;
    left: 0;
    width: 100vw;
    height: 100vh;
    background: black;
    z-index: 10000;
    cursor: none;
  }

  .screensaver-iframe {
    width: 100%;
    height: 100%;
    border: none;
  }
  .explorer-window {
    width: 100%;
    height: 100%;
    display: flex;
    flex-direction: column;
    background: white;
  }
  .explorer-toolbar {
    background: #c0c0c0;
    border-bottom: 2px inset #c0c0c0;
    padding: 4px 8px;
  }
  .explorer-address-bar {
    display: flex;
    align-items: center;
    gap: 8px;
  }
  .address-label {
    font-family: 'MS Sans Serif', sans-serif;
    font-size: 12px;
    font-weight: bold;
    color: #000;
  }
  .address-display {
    flex: 1;
    padding: 2px 4px;
    border: 2px inset #c0c0c0;
    background: white;
    font-family: 'MS Sans Serif', sans-serif;
    font-size: 12px;
    color: #000;
  }
  .explorer-content {
    flex: 1;
    padding: 20px;
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(120px, 1fr));
    gap: 20px;
    overflow-y: auto;
  }
  .program-item {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 8px;
    padding: 12px 8px;
    cursor: pointer;
    border: 1px solid transparent;
    border-radius: 4px;
  }
  .program-item:focus {
    outline: 1px dotted #000;
  }
  .program-icon {
    width: 48px;
    height: 48px;
    image-rendering: pixelated;
    image-rendering: -moz-crisp-edges;
    image-rendering: crisp-edges;
  }
  .program-name {
    font-family: 'MS Sans Serif', sans-serif;
    font-size: 12px;
    color: #000;
    text-align: center;
    word-wrap: break-word;
    max-width: 100px;
  }

</style>
