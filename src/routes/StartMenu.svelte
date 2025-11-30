<script>
  export let openNewWindow;
  export let openRunMenu;

  let open = false;
  const items = [
    ['windows_update_large-2', 'Windows Update'],
    ['appwizard-4', 'Programs'],
    ['joystick-2', 'Games'],
    ['directory_favorites-3', 'Favorites'],
    ['directory_open_file_mydocs_cool-3', 'Documents'],
    ['settings_gear-2', 'Settings'],
    ['search_file_2-2', 'Find'],
    ['help_book_cool-4', 'Help'],
    ['application_hourglass_small-2', 'Run...'],
    ['key_win-2', 'Log Off'],
  ];
</script>

<div class="bottom-bar">
  <button class="b" class:pressed={open} on:click={() => open = !open}>
    <img src="https://win98icons.alexmeub.com/icons/png/windows-0.png" alt="Windows" class="i" />
    <span style="display:flex; align-items:center; gap:6px;">
      Start
    </span>
  </button>
</div>

{#if open}
  <div class="o" on:click={() => open = false} on:keydown={() => open = false} role="button" tabindex="0"></div>

  <div class="m">
    <div class="h">
      <span class="t">Windows 98</span>
    </div>
    <div class="s">
      {#each items as [icon, text], i}
        {#if i === 8}
          <div class="separator"></div>
        {/if}
        <div
          class="item"
          on:click={() => {
            if (text === 'Log Off') {
              window.location.reload();
            } else if (text === 'Programs') {
              openNewWindow('explorer://');
              open = false;
            } else if (text === 'Games') {
              openNewWindow('https://98plus.js.org/programs/minesweeper/index.html');
              open = false;
            } else if (text === 'Windows Update') {
              openNewWindow('https://www.msn.com/update');
              open = false;
            } else if (text === 'Help') {
              openNewWindow('/help');
              open = false;
            } else if (text === 'Run...') {
              openRunMenu();
              open = false;
            } else if (text === 'Find') {
              openRunMenu();
              open = false;
            } else if (text === 'Favorites') {
              openNewWindow('https://www.internetexplorer.com/');
              open = false;
            } else {
              open = false;
            }
          }}
          on:keydown={() => {
            if (text === 'Log Off') {
              window.location.reload();
            } else if (text === 'Programs') {
              openNewWindow('explorer://');
              open = false;
            } else if (text === 'Games') {
              openNewWindow('https://98plus.js.org/programs/minesweeper/index.html');
              open = false;
            } else if (text === 'Windows Update') {
              openNewWindow('https://www.msn.com/update');
              open = false;
            } else if (text === 'Help') {
              openNewWindow('/help');
              open = false;
            } else if (text === 'Run...') {
              openRunMenu();
              open = false;
            } else if (text === 'Find') {
              openRunMenu();
              open = false;
            } else if (text === 'Favorites') {
              openNewWindow('https://www.internetexplorer.com/');
              open = false;
            } else {
              open = false;
            }
          }}
          role="button"
          tabindex="0"
        >
          <img src="https://win98icons.alexmeub.com/icons/png/{icon}.png" alt="{text}" class="i2">
          {text}
        </div>
      {/each}
    </div>
  </div>

{/if}


<style>
  @font-face {
    font-family: 'Win98';
    src: url('/font.otf') format('opentype');
  }
  .bottom-bar {
    position: fixed;
    bottom: 0;
    left: 0;
    right: 0;
    height: 42px;
    background: rgb(195, 198, 200);
    z-index: 3000;
  }
  .b {
    position: absolute;
    bottom: 0;
    left: 0.2%;
    height: 42px;
    padding: 0 15px 0 12px;
    background: #c0c0c0;
    font-family: 'Win98', 'MS Sans Serif', Tahoma, sans-serif;
    font-size: 17px;
    letter-spacing: 0.1em;
    color: black;
    cursor: pointer;
    display: flex;
    align-items: center;
    gap: 9px;
    border: 2px solid;
    border-color: #ffffff #404040 #404040 #ffffff;
    outline: 1px solid #808080;
    outline-offset: -2px;
    user-select: none;
  }
  .b:active,
  .b.pressed {
    border-color: #404040 #ffffff #ffffff #404040;
    outline-color: #808080;
  }
  .i {
    width: 32px;
    height: 32px;
    image-rendering: pixelated;
  }
  .i2 {
    width: 20px;
    height: 20px;
    image-rendering: pixelated;
  }
  .o {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    z-index: 2999;
  }
  .m {
    position: fixed;
    bottom: 42px;
    left: 0.7%;
    width: 196px;
    background: #c0c0c0;
    border: 2px outset #c0c0c0;
    box-shadow: 2px 2px 4px rgba(0, 0, 0, 0.5);
    z-index: 3001;
  }
  .h {
    background: linear-gradient(to right, #000080, #1084d0);
    padding: 3px 8px;
    border-bottom: 1px solid #808080;
  }
  .t {
    font-family: 'Win98', 'MS Sans Serif', sans-serif;
    font-size: 18px;
    font-weight: bold;
    color: white;
  }
  .s {
    padding: 2px 0;
  }
  .item {
    display: flex;
    align-items: center;
    gap: 6px;
    padding: 6px 8px;
    cursor: pointer;
    font-family: 'Win98', 'MS Sans Serif', sans-serif;
    font-size: 16px;
    color: black;
  }
  .item:hover {
    background: #000080;
    color: white;
  }
  .separator {
    height: 1px;
    background: #808080;
    margin: 4px 0;
  }
</style>
