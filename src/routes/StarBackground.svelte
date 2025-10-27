<script>
  import { onMount } from 'svelte';

  let canvas;
  let ctx;
  let stars = [];
  let animationId;

  const numStars = 50;
  const maxStarSize = 2;
  const minStarSize = 0.5;

  class Star {
    constructor() {
      this.x = Math.random() * window.innerWidth;
      this.y = Math.random() * window.innerHeight;
      this.size = Math.random() * (maxStarSize - minStarSize) + minStarSize;
      this.opacity = Math.random() * 0.8 + 0.2;
      this.twinkleSpeed = Math.random() * 0.02 + 0.005;
      this.twinkleDirection = Math.random() > 0.5 ? 1 : -1;
    }

    update() {
      this.opacity += this.twinkleSpeed * this.twinkleDirection;
      if (this.opacity >= 1 || this.opacity <= 0.2) {
        this.twinkleDirection *= -1;
      }
    }

    draw() {
      ctx.save();
      ctx.globalAlpha = this.opacity;
      ctx.fillStyle = '#ffffff';
      ctx.beginPath();
      ctx.arc(this.x, this.y, this.size, 0, Math.PI * 2);
      ctx.fill();
      ctx.restore();
    }
  }

  function initStars() {
    stars = [];
    for (let i = 0; i < numStars; i++) {
      stars.push(new Star());
    }
  }

  function animate() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);

    ctx.fillStyle = '#000000';
    ctx.fillRect(0, 0, canvas.width, canvas.height);

    stars.forEach(star => {
      star.update();
      star.draw();
    });

    animationId = requestAnimationFrame(animate);
  }

  function resizeCanvas() {
    canvas.width = window.innerWidth;
    canvas.height = window.innerHeight;
    initStars();
  }

  onMount(() => {
    ctx = canvas.getContext('2d');
    resizeCanvas();

    window.addEventListener('resize', resizeCanvas);
    animate();

    return () => {
      window.removeEventListener('resize', resizeCanvas);
      cancelAnimationFrame(animationId);
    };
  });
</script>

<canvas
  bind:this={canvas}
  class="fixed top-0 left-0 w-full h-full pointer-events-none z-0"
></canvas>
