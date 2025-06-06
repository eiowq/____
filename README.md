

<html lang="ar">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>انميشن خلفية مع صورة متحركة واسم وأيقونات تواصل</title>

<link rel="preconnect" href="https://fonts.googleapis.com" />
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
<link
  href="https://fonts.googleapis.com/css2?family=Martian+Mono:wght@100..800&display=swap"
  rel="stylesheet"
/>

<!-- إضافة Font Awesome -->
<link
  rel="stylesheet"
  href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css"
/>

<style>
  body, html {
    margin: 0;
    padding: 0;
    width: 100%;
    height: 100%;
    overflow: hidden;
    background: #141414;
    font-family: 'Martian Mono', monospace;
    color: white;
    position: relative;
    display: flex;
    flex-direction: column;
    align-items: center;
  }

  a-hole {
    position: fixed;
    top: 0;
    left: 0;
    width: 100vw;
    height: 100vh;
    z-index: 0;
    pointer-events: none;
    display: block;
  }

  /* العنوان فوق */
  .profile-info {
    margin-top: 2rem;
    font-size: 3rem;
    animation: colorShift 6s linear infinite;
    z-index: 10;
    user-select: none;
  }

  @keyframes colorShift {
    0%   { color: #00f8f1; }
    25%  { color: #ffbd1e; }
    50%  { color: #fe848f; }
    75%  { color: #a900ff; }
    100% { color: #00f8f1; }
  }

  /* الصورة المتحركة */
  .moving-image {
    margin-top: 3rem;
    width: 220px;
    height: 220px;
    border-radius: 50%;
    overflow: hidden;
    border: 4px solid #00f8f1;
    box-shadow: 0 0 15px #00f8f1aa;
    animation: floatMove 6s ease-in-out infinite;
    z-index: 5;
  }
  .moving-image img {
    width: 100%;
    height: 100%;
    object-fit: cover;
  }

  @keyframes floatMove {
    0%, 100% {
      transform: translateY(0);
    }
    50% {
      transform: translateY(20px);
    }
  }

  /* أيقونات التواصل الاجتماعي */
  .social-icons {
    margin-top: 3rem;
    display: flex;
    gap: 30px;
    font-size: 2.5rem;
    color: #00f8f1;
    cursor: pointer;
    transition: color 0.3s ease;
    z-index: 10;
  }
  .social-icons a:hover {
    color: #ffbd1e;
  }
</style>
</head>
<body>

<a-hole>
  <canvas class="js-canvas"></canvas>
  <div class="aura"></div>
  <div class="overlay"></div>
</a-hole>

<div class="profile-info">
  Ahmed Tarek
</div>

<div class="moving-image">
  <img src="s.jpg" alt="صورة شخصية" />
</div>

<div class="social-icons">
  <a href="https://www.facebook.com/ahmdtarq.bd.alm.bwd?locale=ar_AR" target="_blank" aria-label="Facebook">
    <i class="fab fa-facebook-f"></i>
  </a>
 

  </a>
  <a href="https://www.instagram.com/ahmed_tarek_772/" target="_blank" aria-label="Instagram">
    <i class="fab fa-instagram"></i>
  </a>

</div>

<script type="module">
  import easingUtils from "https://esm.sh/easing-utils";

  class AHole extends HTMLElement {
    connectedCallback() {
      this.canvas = this.querySelector(".js-canvas");
      this.ctx = this.canvas.getContext("2d");
      this.discs = [];
      this.lines = [];
      this.setSize();
      this.setDiscs();
      this.setLines();
      this.setParticles();
      window.addEventListener("resize", () => {
        this.setSize();
        this.setDiscs();
        this.setLines();
        this.setParticles();
      });
      requestAnimationFrame(this.tick.bind(this));
    }

    setSize() {
      this.rect = this.getBoundingClientRect();
      this.render = {
        width: this.rect.width,
        height: this.rect.height,
        dpi: window.devicePixelRatio,
      };
      this.canvas.width = this.render.width * this.render.dpi;
      this.canvas.height = this.render.height * this.render.dpi;
    }

    setDiscs() {
      const { width, height } = this.rect;
      this.discs = [];
      this.startDisc = {
        x: width * 0.5,
        y: height * 0.45,
        w: width * 0.75,
        h: height * 0.7,
      };
      this.endDisc = {
        x: width * 0.5,
        y: height * 0.95,
        w: 0,
        h: 0,
      };
      const totalDiscs = 100;
      let prevBottom = height;
      this.clip = {};
      for (let i = 0; i < totalDiscs; i++) {
        const p = i / totalDiscs;
        const disc = this.tweenDisc({ p });
        const bottom = disc.y + disc.h;
        if (bottom <= prevBottom) {
          this.clip = {
            disc: { ...disc },
            i,
          };
        }
        prevBottom = bottom;
        this.discs.push(disc);
      }
      this.clip.path = new Path2D();
      this.clip.path.ellipse(
        this.clip.disc.x,
        this.clip.disc.y,
        this.clip.disc.w,
        this.clip.disc.h,
        0,
        0,
        Math.PI * 2
      );
      this.clip.path.rect(
        this.clip.disc.x - this.clip.disc.w,
        0,
        this.clip.disc.w * 2,
        this.clip.disc.y
      );
    }

    setLines() {
      const { width, height } = this.rect;
      this.lines = [];
      const totalLines = 100;
      const linesAngle = (Math.PI * 2) / totalLines;
      for (let i = 0; i < totalLines; i++) {
        this.lines.push([]);
      }
      this.discs.forEach((disc) => {
        for (let i = 0; i < totalLines; i++) {
          const angle = i * linesAngle;
          const p = {
            x: disc.x + Math.cos(angle) * disc.w,
            y: disc.y + Math.sin(angle) * disc.h,
          };
          this.lines[i].push(p);
        }
      });
      this.linesCanvas = new OffscreenCanvas(width, height);
      const ctx = this.linesCanvas.getContext("2d");
      this.lines.forEach((line, i) => {
        ctx.save();
        let lineIsIn = false;
        line.forEach((p1, j) => {
          if (j === 0) return;
          const p0 = line[j - 1];
          if (
            !lineIsIn &&
            (ctx.isPointInPath(this.clip.path, p1.x, p1.y) ||
              ctx.isPointInStroke(this.clip.path, p1.x, p1.y))
          ) {
            lineIsIn = true;
          } else if (lineIsIn) {
            ctx.clip(this.clip.path);
          }
          ctx.beginPath();
          ctx.moveTo(p0.x, p0.y);
          ctx.lineTo(p1.x, p1.y);
          ctx.strokeStyle = "#444";
          ctx.lineWidth = 2;
          ctx.stroke();
          ctx.closePath();
        });
        ctx.restore();
      });
      this.linesCtx = ctx;
    }

    setParticles() {
      const { width, height } = this.rect;
      this.particles = [];
      this.particleArea = {
        sw: this.clip.disc.w * 0.5,
        ew: this.clip.disc.w * 2,
        h: height * 0.85,
      };
      this.particleArea.sx = (width - this.particleArea.sw) / 2;
      this.particleArea.ex = (width - this.particleArea.ew) / 2;
      const totalParticles = 100;
      for (let i = 0; i < totalParticles; i++) {
        this.particles.push(this.initParticle(true));
      }
    }

    initParticle(start = false) {
      const sx = this.particleArea.sx + this.particleArea.sw * Math.random();
      const ex = this.particleArea.ex + this.particleArea.ew * Math.random();
      const dx = ex - sx;
      const vx = 0.1 + Math.random() * 0.5;
      const y = start ? this.particleArea.h * Math.random() : this.particleArea.h;
      const r = 0.5 + Math.random() * 4;
      const vy = 0.5 + Math.random();
      return {
        x: sx,
        sx,
        dx,
        y,
        vy,
        p: 0,
        r,
        c: `rgba(255, 255, 255, ${Math.random()})`,
      };
    }

    tweenValue(start, end, p, ease = false) {
      const delta = end - start;
      const easeFn =
        easingUtils[
          ease ? "ease" + ease.charAt(0).toUpperCase() + ease.slice(1) : "linear"
        ];
      return start + delta * easeFn(p);
    }

    drawDiscs() {
      const { ctx } = this;
      ctx.strokeStyle = "#444";
      ctx.lineWidth = 2;
      const outerDisc = this.startDisc;
      ctx.beginPath();
      ctx.ellipse(outerDisc.x, outerDisc.y, outerDisc.w, outerDisc.h, 0, 0, Math.PI * 2);
      ctx.stroke();
      ctx.closePath();
      this.discs.forEach((disc, i) => {
        if (i % 5 !== 0) return;
        if (disc.w < this.clip.disc.w - 5) {
          ctx.save();
          ctx.clip(this.clip.path);
        }
        ctx.beginPath();
        ctx.ellipse(disc.x, disc.y, disc.w, disc.h, 0, 0, Math.PI * 2);
        ctx.stroke();
        ctx.closePath();
        if (disc.w < this.clip.disc.w - 5) {
          ctx.restore();
        }
      });
    }

    drawLines() {
      const { ctx, linesCanvas } = this;
      ctx.drawImage(linesCanvas, 0, 0);
    }

    drawParticles() {
      const { ctx } = this;
      ctx.save();
      ctx.clip(this.clip.path);
      this.particles.forEach((particle) => {
        ctx.fillStyle = particle.c;
        ctx.beginPath();
        ctx.rect(particle.x, particle.y, particle.r, particle.r);
        ctx.closePath();
        ctx.fill();
      });
      ctx.restore();
    }

    moveDiscs() {
      this.discs.forEach((disc) => {
        disc.p = (disc.p + 0.001) % 1;
        this.tweenDisc(disc);
      });
    }

    moveParticles() {
      this.particles.forEach((particle) => {
        particle.p = 1 - particle.y / this.particleArea.h;
        particle.x = particle.sx + particle.dx * particle.p;
        particle.y -= particle.vy;
        if (particle.y < 0) {
          particle.y = this.initParticle().y;
        }
      });
    }

    tweenDisc(disc) {
      disc.x = this.tweenValue(this.startDisc.x, this.endDisc.x, disc.p);
      disc.y = this.tweenValue(this.startDisc.y, this.endDisc.y, disc.p, "inExpo");
      disc.w = this.tweenValue(this.startDisc.w, this.endDisc.w, disc.p);
      disc.h = this.tweenValue(this.startDisc.h, this.endDisc.h, disc.p);
      return disc;
    }

    tick() {
      const { ctx } = this;
      ctx.clearRect(0, 0, this.canvas.width, this.canvas.height);
      ctx.save();
      ctx.scale(this.render.dpi, this.render.dpi);
      this.moveDiscs();
      this.moveParticles();
      this.drawDiscs();
      this.drawLines();
      this.drawParticles();
      ctx.restore();
      requestAnimationFrame(this.tick.bind(this));
    }
  }

  customElements.define("a-hole", AHole);
</script>

</body>
</html>
