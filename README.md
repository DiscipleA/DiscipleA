<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=1584,height=396">
<style>
  * { margin: 0; padding: 0; box-sizing: border-box; }

  body {
    width: 1584px;
    height: 396px;
    background: #0d110d;
    font-family: 'JetBrains Mono', 'Cascadia Code', 'Fira Code', 'Consolas', monospace;
    overflow: hidden;
    position: relative;
  }

  /* Grid overlay */
  .grid {
    position: absolute;
    inset: 0;
    background-image:
      linear-gradient(rgba(130, 150, 110, 0.04) 1px, transparent 1px),
      linear-gradient(90deg, rgba(130, 150, 110, 0.04) 1px, transparent 1px);
    background-size: 36px 36px;
    z-index: 1;
  }

  /* Scanlines */
  .scanlines {
    position: absolute;
    inset: 0;
    background: repeating-linear-gradient(
      0deg,
      transparent,
      transparent 2px,
      rgba(0, 0, 0, 0.10) 2px,
      rgba(0, 0, 0, 0.10) 4px
    );
    z-index: 5;
    pointer-events: none;
  }

  /* Matrix rain canvas */
  #matrix-canvas {
    position: absolute;
    inset: 0;
    z-index: 2;
    opacity: 0.80;
  }

  /* Main content container */
  .content {
    position: absolute;
    inset: 0;
    z-index: 3;
    display: flex;
    align-items: center;
    padding-left: 440px;
  }

  /* Terminal window frame */
  .terminal-frame {
    border: 1px solid rgba(130, 148, 110, 0.30);
    border-radius: 8px;
    overflow: hidden;
    background: rgba(10, 13, 10, 0.92);
    box-shadow:
      0 0 80px rgba(130, 148, 110, 0.06),
      0 0 200px rgba(0, 0, 0, 0.6),
      inset 0 0 80px rgba(0, 0, 0, 0.5);
    width: 820px;
  }

  .terminal-header {
    background: rgba(20, 22, 20, 0.95);
    padding: 12px 18px;
    display: flex;
    align-items: center;
    gap: 9px;
    border-bottom: 1px solid rgba(130, 148, 110, 0.10);
  }

  .dot { width: 11px; height: 11px; border-radius: 50%; }
  .dot.red { background: #c4736b; box-shadow: 0 0 4px rgba(196,115,107,0.3); }
  .dot.yellow { background: #c4a84e; box-shadow: 0 0 4px rgba(196,168,78,0.3); }
  .dot.green { background: #7a9666; box-shadow: 0 0 4px rgba(122,150,102,0.3); }

  .terminal-title {
    color: rgba(200, 200, 195, 0.25);
    font-size: 11px;
    margin-left: 18px;
    letter-spacing: 0.6px;
  }

  .terminal-body {
    padding: 28px 32px;
  }

  .prompt-line {
    display: flex;
    align-items: baseline;
    margin-bottom: 8px;
  }

  .prompt {
    color: #8fa878;
    font-size: 14px;
    font-weight: 600;
    text-shadow: 0 0 8px rgba(143, 168, 120, 0.3);
  }

  .cmd {
    color: #d8d6cf;
    font-size: 14px;
    margin-left: 10px;
  }

  .output-name {
    color: #9ab885;
    font-size: 38px;
    font-weight: 700;
    letter-spacing: -1px;
    line-height: 1.4;
    text-shadow:
      0 0 40px rgba(154, 184, 133, 0.20),
      0 0 80px rgba(154, 184, 133, 0.08);
  }

  .output-title {
    color: rgba(228, 226, 218, 0.88);
    font-size: 15px;
    margin-top: 8px;
    line-height: 1.6;
  }

  .output-dim {
    color: rgba(210, 208, 200, 0.35);
    font-size: 12px;
    margin-top: 6px;
  }

  .divider {
    color: rgba(154, 184, 133, 0.25);
    margin: 0 6px;
  }

  .cursor-line {
    margin-top: 22px;
    display: flex;
    align-items: center;
  }

  .cursor {
    display: inline-block;
    width: 9px;
    height: 17px;
    background: #9ab885;
    margin-left: 2px;
    box-shadow: 0 0 10px rgba(154, 184, 133, 0.4);
  }

  /* Right side: tech tags */
  .tech-tags {
    position: absolute;
    right: 100px;
    top: 50%;
    transform: translateY(-54%);
    z-index: 4;
    display: flex;
    flex-direction: column;
    gap: 14px;
    align-items: flex-end;
  }

  .tag {
    color: rgba(150, 165, 130, 0.45);
    font-family: 'JetBrains Mono', 'Cascadia Code', 'Fira Code', monospace;
    font-size: 13px;
    letter-spacing: 1.5px;
  }

  .tag.highlight {
    color: rgba(170, 190, 145, 0.78);
    font-size: 14px;
    font-weight: 600;
    text-shadow: 0 0 10px rgba(154, 184, 133, 0.20);
  }

  /* Bottom status bar */
  .bottom-line {
    position: absolute;
    bottom: 28px;
    left: 530px;
    z-index: 4;
    color: rgba(200, 195, 185, 0.22);
    font-size: 10px;
    font-family: 'JetBrains Mono', 'Cascadia Code', 'Fira Code', monospace;
    letter-spacing: 0.3px;
  }

  /* Left side ambient glow */
  .left-glow {
    position: absolute;
    left: 0;
    top: 0;
    width: 500px;
    height: 396px;
    background: radial-gradient(ellipse at 20% 50%, rgba(140, 160, 120, 0.04) 0%, transparent 70%);
    z-index: 2;
    pointer-events: none;
  }
</style>
</head>
<body>

<canvas id="matrix-canvas"></canvas>
<div class="grid"></div>
<div class="left-glow"></div>
<div class="scanlines"></div>

<div class="content">
  <div class="terminal-frame">
    <div class="terminal-header">
      <span class="dot red"></span>
      <span class="dot yellow"></span>
      <span class="dot green"></span>
      <span class="terminal-title">dmitriy@dev — bash — 80×24</span>
    </div>
    <div class="terminal-body">
      <div class="prompt-line">
        <span class="prompt">❯</span>
        <span class="cmd">whoami</span>
      </div>
      <div class="output-name">Dmitriy Chernichenko</div>
      <div class="output-title">
        Fullstack Developer in Training
        <span class="divider">|</span>
        AI-Integrated Platforms
        <span class="divider">|</span>
        SQL &bull; Python &bull; React
      </div>
      <div class="output-dim">
        BASc App Development (Junior)  &bull;  Building Trading Systems &amp; Educational AI
      </div>
      <div class="cursor-line">
        <span class="prompt">❯</span>
        <span class="cmd">_</span>
        <span class="cursor"></span>
      </div>
    </div>
  </div>
</div>

<div class="tech-tags">
  <span class="tag">[ python ]</span>
  <span class="tag">[ react ]</span>
  <span class="tag highlight">[ sql ]</span>
  <span class="tag">[ llm ]</span>
  <span class="tag">[ javascript ]</span>
  <span class="tag">[ git ]</span>
  <span class="tag highlight">[ ai ]</span>
  <span class="tag">[ playwright ]</span>
  <span class="tag">[ docker ]</span>
</div>

<div class="bottom-line">
  Node 20  &bull;  Python 3.11  &bull;  MySQL  &bull;  React 19  &bull;  Docker
</div>

<script>
// Matrix rain — muted sage/olive tones
const canvas = document.getElementById('matrix-canvas');
const ctx = canvas.getContext('2d');

canvas.width = 1584;
canvas.height = 396;

const chars = 'アイウエオカキクケコサシスセソタチツテトナニヌネノハヒフヘホマミムメモヤユヨラリルレロワヲン0123456789ABCDEF{}[]()<>/\\|*&^%$#@!~`+-=_;:,.?';
const charArray = chars.split('');
const fontSize = 14;
const columns = Math.floor(canvas.width / fontSize);
const drops = [];

// Initialize
for (let i = 0; i < columns; i++) {
  drops[i] = Math.floor(Math.random() * -canvas.height / fontSize);
}

function draw() {
  ctx.fillStyle = 'rgba(11, 14, 11, 0.055)';
  ctx.fillRect(0, 0, canvas.width, canvas.height);

  ctx.font = fontSize + 'px "JetBrains Mono", monospace';

  for (let i = 0; i < drops.length; i++) {
    const text = charArray[Math.floor(Math.random() * charArray.length)];
    const y = drops[i] * fontSize;

    // Lead character — muted sage green (matching tree tones)
    ctx.fillStyle = 'rgba(175, 190, 150, 0.85)';
    ctx.fillText(text, i * fontSize, y);

    // Trail — fading olive/sage tones
    for (let t = 1; t < 8; t++) {
      const trailY = y - t * fontSize;
      if (trailY > 0) {
        const alpha = 0.65 - (t * 0.085);
        if (alpha > 0) {
          ctx.fillStyle = `rgba(140, 160, 120, ${alpha})`;
          ctx.fillText(charArray[Math.floor(Math.random() * charArray.length)], i * fontSize, trailY);
        }
      }
    }

    if (y > canvas.height && Math.random() > 0.975) {
      drops[i] = 0;
    }
    drops[i]++;
  }
}

setInterval(draw, 45);
</script>

</body>
</html>
[linkedin-banner.html](https://github.com/user-attachments/files/28489496/linkedin-banner.html)

## Hi there 👋

<!--
**DiscipleA/DiscipleA** is a ✨ _special_ ✨ repository because its `README.md` (this file) appears on your GitHub profile.

Here are some ideas to get you started:

- 🔭 I’m currently working on ...
- 🌱 I’m currently learning ...
- 👯 I’m looking to collaborate on ...
- 🤔 I’m looking for help with ...
- 💬 Ask me about ...
- 📫 How to reach me: ...
- 😄 Pronouns: ...
- ⚡ Fun fact: ...
-->
