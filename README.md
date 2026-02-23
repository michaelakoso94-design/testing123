<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>Antigravity</title>
<style>
  * { margin: 0; padding: 0; box-sizing: border-box; }

  body {
    background: #0a0a0f;
    width: 100vw;
    height: 100vh;
    overflow: hidden;
    font-family: 'Georgia', serif;
  }

  canvas {
    display: block;
    position: fixed;
    top: 0; left: 0;
    width: 100%; height: 100%;
  }

  .label {
    position: fixed;
    bottom: 28px;
    width: 100%;
    text-align: center;
    color: rgba(255,255,255,0.18);
    font-size: 11px;
    letter-spacing: 4px;
    text-transform: uppercase;
    pointer-events: none;
  }
</style>
</head>
<body>

<canvas id="c"></canvas>
<div class="label">move cursor to disturb</div>

<script>
const canvas = document.getElementById('c');
const ctx = canvas.getContext('2d');

let W = canvas.width = window.innerWidth;
let H = canvas.height = window.innerHeight;

let mouse = { x: W / 2, y: H / 2 };

window.addEventListener('mousemove', e => {
  mouse.x = e.clientX;
  mouse.y = e.clientY;
});

window.addEventListener('touchmove', e => {
  mouse.x = e.touches[0].clientX;
  mouse.y = e.touches[0].clientY;
}, { passive: true });

window.addEventListener('resize', () => {
  W = canvas.width = window.innerWidth;
  H = canvas.height = window.innerHeight;
  init();
});

const COLORS = [
  'rgba(147, 112, 219, ',  // purple
  'rgba(100, 180, 255, ',  // blue
  'rgba(255, 160, 100, ',  // orange
  'rgba(100, 230, 180, ',  // teal
  'rgba(255, 220, 120, ',  // gold
  'rgba(220, 130, 200, ',  // pink
];

class Particle {
  constructor() {
    this.reset(true);
  }

  reset(init = false) {
    this.x = Math.random() * W;
    this.y = init ? Math.random() * H : H + 20;
    this.baseSize = Math.random() * 3 + 1;
    this.size = this.baseSize;
    this.speedY = -(Math.random() * 0.6 + 0.2);   // float upward
    this.speedX = (Math.random() - 0.5) * 0.4;
    this.wobble = Math.random() * Math.PI * 2;
    this.wobbleSpeed = (Math.random() * 0.02 + 0.005);
    this.wobbleAmp = Math.random() * 0.6 + 0.1;
    this.color = COLORS[Math.floor(Math.random() * COLORS.length)];
    this.alpha = Math.random() * 0.6 + 0.2;
    this.targetAlpha = this.alpha;
    this.life = 0;
    this.maxLife = Math.random() * 300 + 200;
    // shape: 0=circle, 1=cross, 2=square, 3=triangle
    this.shape = Math.random() < 0.6 ? 0 : Math.floor(Math.random() * 4);
    this.rotation = Math.random() * Math.PI * 2;
    this.rotSpeed = (Math.random() - 0.5) * 0.03;
  }

  update() {
    this.life++;
    this.wobble += this.wobbleSpeed;
    this.rotation += this.rotSpeed;

    // mouse repulsion
    const dx = this.x - mouse.x;
    const dy = this.y - mouse.y;
    const dist = Math.sqrt(dx * dx + dy * dy);
    const repulse = 120;
    if (dist < repulse) {
      const force = (repulse - dist) / repulse;
      this.x += dx / dist * force * 3;
      this.y += dy / dist * force * 3;
    }

    this.x += this.speedX + Math.sin(this.wobble) * this.wobbleAmp;
    this.y += this.speedY;

    // fade in/out
    if (this.life < 40) {
      this.alpha = (this.life / 40) * this.targetAlpha;
    } else if (this.life > this.maxLife - 40) {
      this.alpha = ((this.maxLife - this.life) / 40) * this.targetAlpha;
    }

    if (this.life >= this.maxLife || this.y < -20 || this.x < -20 || this.x > W + 20) {
      this.reset();
    }
  }

  draw() {
    ctx.save();
    ctx.globalAlpha = this.alpha;
    ctx.fillStyle = this.color + '1)';
    ctx.strokeStyle = this.color + '0.8)';
    ctx.lineWidth = 1;
    ctx.translate(this.x, this.y);
    ctx.rotate(this.rotation);

    const s = this.size;
    ctx.beginPath();

    if (this.shape === 0) {
      // circle
      ctx.arc(0, 0, s, 0, Math.PI * 2);
      ctx.fill();
    } else if (this.shape === 1) {
      // square
      ctx.rect(-s, -s, s * 2, s * 2);
      ctx.stroke();
    } else if (this.shape === 2) {
      // triangle
      ctx.moveTo(0, -s * 1.3);
      ctx.lineTo(s * 1.1, s * 0.8);
      ctx.lineTo(-s * 1.1, s * 0.8);
      ctx.closePath();
      ctx.stroke();
    } else {
      // cross / plus
      ctx.moveTo(-s * 1.5, 0); ctx.lineTo(s * 1.5, 0);
      ctx.moveTo(0, -s * 1.5); ctx.lineTo(0, s * 1.5);
      ctx.stroke();
    }

    ctx.restore();

    // glow
    const grd = ctx.createRadialGradient(this.x, this.y, 0, this.x, this.y, s * 4);
    grd.addColorStop(0, this.color + (this.alpha * 0.25) + ')');
    grd.addColorStop(1, this.color + '0)');
    ctx.beginPath();
    ctx.arc(this.x, this.y, s * 4, 0, Math.PI * 2);
    ctx.fillStyle = grd;
    ctx.fill();
  }
}

let particles = [];

function init() {
  const count = Math.floor((W * H) / 8000);
  particles = Array.from({ length: Math.min(count, 150) }, () => new Particle());
}

function draw() {
  // trail effect
  ctx.fillStyle = 'rgba(10, 10, 15, 0.18)';
  ctx.fillRect(0, 0, W, H);

  for (const p of particles) {
    p.update();
    p.draw();
  }

  requestAnimationFrame(draw);
}

init();
draw();
</script>
</body>
</html>
