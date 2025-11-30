<script>
  // Languages page component
</script>

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
</div>

<style>
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
    grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
    gap: 15px;
  }

  .language-item {
    background: #c0c0c0;
    border: 1px outset #c0c0c0;
    padding: 15px;
    display: flex;
    flex-direction: column;
    align-items: center;
    text-align: center;
    font-family: 'MS Sans Serif', sans-serif;
  }

  .language-item:hover {
    background: #d0d0d0;
  }

  .language-icon {
    font-size: 32px;
    margin-bottom: 8px;
    color: #000;
  }

  .language-name {
    font-size: 14px;
    font-weight: bold;
    margin-bottom: 5px;
    color: #000;
  }

</style>
