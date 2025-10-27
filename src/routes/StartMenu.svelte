<script>
  let open = false;
  const items = [['keys-5', 'Restart']];
</script>

<div class="bottom-bar">
  <button class="b" class:pressed={open} on:click={() => open = !open}>
    <img src="https://win98icons.alexmeub.com/icons/png/windows-0.png" alt="" class="i">
    Start
  </button>
</div>

{#if open}
  <div class="o" on:click={() => open = false} on:keypress={() => open = false} role="button" tabindex="0"></div>
  <div class="m">
    <div class="h"><span class="t">Start Menu</span></div>
    <div class="s">
      {#each items as [icon, text], i}
        <div class="item" on:click={() => open = false} on:keypress={() => open = false} role="button" tabindex="0">
          <img src="https://win98icons.alexmeub.com/icons/png/{icon}.png" alt="" class="i">
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
  .bottom-bar { position: fixed; bottom: 0; left: 0; right: 0; height: 42px; background: rgb(195, 198, 200); z-index: 999; }
  .b { position: absolute; bottom: 0; left: 0.2%; height: 42px; padding: 0 15px 0 12px; background: #c0c0c0; font-family: 'Win98', 'MS Sans Serif', Tahoma, sans-serif; font-size: 17px; letter-spacing: 0.1em; color: black; cursor: pointer; display: flex; align-items: center; gap: 9px; border: 2px solid; border-color: #ffffff #404040 #404040 #ffffff; outline: 1px solid #808080; outline-offset: -2px; user-select: none; }
  .b:active, .b.pressed { border-color: #404040 #ffffff #ffffff #404040; outline-color: #808080; }
  .i { width: 26px; height: 26px; image-rendering: pixelated; }
  .o { position: fixed; top: 0; left: 0; right: 0; bottom: 0; z-index: 999; }
  .m { position: fixed; bottom: 86px; left: 20%; width: 600px; background: #c0c0c0; border: 6px outset #c0c0c0; box-shadow: 6px 6px 12px rgba(0,0,0,.5); z-index: 1000; }
  .h { background: linear-gradient(to right, #000080, #1084d0); padding: 6px 24px; border-bottom: 3px solid #808080; }
  .t { font-family: 'Win98', 'MS Sans Serif', sans-serif; font-size: 33px; font-weight: bold; letter-spacing: 0.1em; color: white; }
  .s { padding: 12px 0; }
  .item { display: flex; align-items: center; gap: 24px; padding: 12px 24px; cursor: pointer; font-family: 'Win98', 'MS Sans Serif', sans-serif; font-size: 33px; letter-spacing: 0.1em; color: black; }
  .item:hover { background: #000080; color: white; }
  
</style>
