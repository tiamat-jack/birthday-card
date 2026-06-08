# 生日电子贺卡 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a single-page birthday e-card with swipe-driven wish messages, CSS-animated cake drop, and Web Audio birthday song.

**Architecture:** Single `index.html` file containing all HTML structure, CSS styles, and JS logic. Two animation phases: Phase 1 shows 6 birthday wishes (swipe-driven), Phase 2 plays a CSS-animated cake drop sequence with synthesized birthday song and title animation.

**Tech Stack:** HTML5, CSS3 (@keyframes + transform), vanilla JS, Web Audio API, Google Fonts (ZCOOL KuaiLe)

---

## File Structure

| File | Responsibility |
|------|---------------|
| `index.html` | Everything — structure, styles, animations, audio, interaction logic |

States managed in JS:
- `currentWish` (int, 0-5): which wish is visible
- `phase` ("wishes" | "cake"): which animation phase
- `audioCtx` (AudioContext | null): lazy-init on first interaction

---

### Task 1: HTML scaffolding and global styles

**Files:**
- Create: `index.html`

- [ ] **Step 1: Write HTML structure**

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
  <title>生日快乐</title>
  <link href="https://fonts.googleapis.com/css2?family=ZCOOL+KuaiLe&display=swap" rel="stylesheet">
  <style>
    *, *::before, *::after { margin: 0; padding: 0; box-sizing: border-box; }
    html, body { width: 100%; height: 100%; overflow: hidden; }
    body {
      font-family: 'ZCOOL KuaiLe', 'Microsoft YaHei', sans-serif;
      background: linear-gradient(180deg, #1a0033 0%, #4a154b 30%, #ff6b9d 100%);
      display: flex; justify-content: center; align-items: center;
    }
    .card {
      width: 100%; max-width: 480px; height: 100vh;
      position: relative; overflow: hidden;
      display: flex; flex-direction: column; align-items: center;
    }
    /* Title zone */
    .title-zone {
      position: absolute; top: 8vh; left: 0; right: 0;
      display: flex; justify-content: center; gap: 12px;
      opacity: 0; pointer-events: none; z-index: 10;
    }
    /* Main stage */
    .stage {
      flex: 1; width: 100%;
      display: flex; justify-content: center; align-items: center;
      position: relative;
    }
    /* Hint zone */
    .hint-zone {
      position: absolute; bottom: 6vh; left: 0; right: 0;
      display: flex; flex-direction: column; align-items: center;
      z-index: 5; transition: opacity 0.3s;
    }
  </style>
</head>
<body>
  <div class="card" id="card">
    <div class="title-zone" id="titleZone">
      <span class="title-char">生</span>
      <span class="title-char">日</span>
      <span class="title-char">快</span>
      <span class="title-char">乐</span>
    </div>
    <div class="stage" id="stage">
      <div class="wish-container" id="wishContainer"></div>
      <div class="cake-stage" id="cakeStage"></div>
    </div>
    <div class="hint-zone" id="hintZone">
      <span class="hint-text">向上滑动</span>
      <span class="hint-arrow">↑</span>
    </div>
  </div>
  <script>
    // JS goes here
  </script>
</body>
</html>
```

- [ ] **Step 2: Add base responsive styles for .title-char, .wish-container, .hint-text, .hint-arrow**

Add to `<style>` section:

```css
.title-char {
  font-size: clamp(2.5rem, 8vw, 3.5rem);
  font-weight: bold;
  opacity: 0; transform: scale(0) rotate(-10deg);
}
.hint-text { color: rgba(255,255,255,0.7); font-size: 0.9rem; margin-bottom: 4px; }
.hint-arrow { color: rgba(255,255,255,0.7); font-size: 1.5rem; animation: hintBounce 1.5s infinite; }
@keyframes hintBounce {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-10px); }
}
.wish-container, .cake-stage { position: absolute; width: 100%; display: flex; justify-content: center; align-items: center; }
.cake-stage { opacity: 0; pointer-events: none; flex-direction: column; }
```

- [ ] **Step 3: Verify layout in browser**

Run: open `index.html` in a browser. Confirm dark gradient background shows, title zone is hidden, hint arrow bounces at bottom.

---

### Task 2: Birthday wishes with cartoon art text

**Files:**
- Modify: `index.html` (add wish styles + HTML content)

- [ ] **Step 1: Add cartoon art-text CSS**

Add before `</style>`:

```css
.wish-text {
  font-size: clamp(1.5rem, 5vw, 2.2rem);
  color: #ffffff;
  text-align: center;
  padding: 0 24px;
  line-height: 1.8;
  letter-spacing: 0.08em;
  /* Cartoon 3D bubble effect */
  text-shadow:
    0 0 10px rgba(255,255,255,0.9),
    0 0 20px rgba(255,180,200,0.5),
    0 2px 0 #ff3388,
    0 4px 0 #cc1166,
    0 6px 0 #990044,
    0 8px 12px rgba(0,0,0,0.3);
  opacity: 0;
  transform: translateY(40px);
  transition: opacity 0.5s cubic-bezier(0.34, 1.56, 0.64, 1),
              transform 0.5s cubic-bezier(0.34, 1.56, 0.64, 1);
  will-change: transform, opacity;
}
.wish-text.active {
  opacity: 1;
  transform: translateY(0);
}
.wish-text.exit-up {
  opacity: 0;
  transform: translateY(-60px);
  transition: opacity 0.3s ease-in, transform 0.3s ease-in;
}
.wish-text.exit-down {
  opacity: 0;
  transform: translateY(60px);
  transition: opacity 0.3s ease-in, transform 0.3s ease-in;
}
```

- [ ] **Step 2: Add wish text content to wish-container**

Replace `<div class="wish-container" id="wishContainer"></div>` with:

```html
<div class="wish-container" id="wishContainer">
  <p class="wish-text active" data-index="0">愿你的每一天<br>都如星光般闪耀</p>
  <p class="wish-text" data-index="1">岁月流转<br>你依然是那束最美的光</p>
  <p class="wish-text" data-index="2">愿所有美好<br>都向你奔赴而来</p>
  <p class="wish-text" data-index="3">今天是你的节日<br>全世界都在为你庆祝</p>
  <p class="wish-text" data-index="4">许一个愿吧<br>它一定会实现</p>
  <p class="wish-text" data-index="5">准备好了吗？<br>🎂</p>
</div>
```

- [ ] **Step 3: Verify wishes render with cartoon text effect**

Open `index.html` in browser. First wish should be visible with white bubble text and pink layered shadow. Other wishes hidden.

---

### Task 3: Swipe interaction for wishes

**Files:**
- Modify: `index.html` (replace `<script>` section)

- [ ] **Step 1: Write JS state and touch/wheel handlers**

Replace `// JS goes here` with:

```javascript
let currentWish = 0;
let phase = 'wishes';
let touchStartY = 0;
let isTransitioning = false;
const totalWishes = 6;

const wishEls = document.querySelectorAll('.wish-text');
const hintZone = document.getElementById('hintZone');
const titleZone = document.getElementById('titleZone');
const cakeStage = document.getElementById('cakeStage');

function showWish(index, direction) {
  if (index < 0 || index >= totalWishes || isTransitioning) return;
  if (phase !== 'wishes') return;

  isTransitioning = true;
  const exitClass = direction === 'up' ? 'exit-up' : 'exit-down';

  wishEls[currentWish].classList.remove('active');
  wishEls[currentWish].classList.add(exitClass);

  wishEls[index].classList.remove('exit-up', 'exit-down');
  // Force reflow
  void wishEls[index].offsetWidth;
  wishEls[index].classList.add('active');

  currentWish = index;

  setTimeout(() => {
    wishEls.forEach(el => el.classList.remove('exit-up', 'exit-down'));
    isTransitioning = false;

    // Last wish → start cake phase after short delay
    if (currentWish === totalWishes - 1) {
      setTimeout(startCakePhase, 800);
    }
  }, 500);

  updateHint();
}

function updateHint() {
  if (currentWish === totalWishes - 1) {
    hintZone.innerHTML = '<span class="hint-text">向上滑动查看惊喜</span><span class="hint-arrow">↑</span>';
  } else {
    hintZone.innerHTML = '<span class="hint-text">向上滑动</span><span class="hint-arrow">↑</span>';
  }
}

// Touch events
document.getElementById('card').addEventListener('touchstart', e => {
  touchStartY = e.touches[0].clientY;
}, { passive: true });

document.getElementById('card').addEventListener('touchend', e => {
  if (phase !== 'wishes') return;
  const deltaY = touchStartY - e.changedTouches[0].clientY;
  if (Math.abs(deltaY) < 40) return; // threshold
  if (deltaY > 0) {
    showWish(currentWish + 1, 'up');
  } else {
    showWish(currentWish - 1, 'down');
  }
});

// Wheel events (desktop)
document.getElementById('card').addEventListener('wheel', e => {
  if (phase !== 'wishes') return;
  e.preventDefault();
  if (e.deltaY > 0) {
    showWish(currentWish + 1, 'up');
  } else {
    showWish(currentWish - 1, 'down');
  }
}, { passive: false });
```

- [ ] **Step 2: Test swipe interaction**

Open `index.html` in mobile viewport (Chrome DevTools device mode). Swipe up to advance through wishes, swipe down to go back. Confirm smooth spring animation and hint text updates.

---

### Task 4: CSS cake components

**Files:**
- Modify: `index.html` (add cake HTML + styles)

- [ ] **Step 1: Add cake HTML inside cake-stage**

Replace `<div class="cake-stage" id="cakeStage"></div>` with:

```html
<div class="cake-stage" id="cakeStage">
  <div class="cake-assembly">
    <div class="cake-part plate">
      <div class="plate-top"></div>
      <div class="plate-shadow"></div>
    </div>
    <div class="cake-part cake-body">
      <div class="cake-body-top"></div>
      <div class="cake-body-front"></div>
    </div>
    <div class="cake-part cream-layer">
      <div class="cream-top"></div>
      <div class="cream-drip drip1"></div>
      <div class="cream-drip drip2"></div>
      <div class="cream-drip drip3"></div>
    </div>
    <div class="cake-part candle">
      <div class="candle-body"></div>
      <div class="candle-stripe stripe1"></div>
      <div class="candle-stripe stripe2"></div>
      <div class="flame">
        <div class="flame-inner"></div>
        <div class="flame-glow"></div>
      </div>
    </div>
  </div>
</div>
```

- [ ] **Step 2: Add cake CSS styles**

Add before `</style>`:

```css
.cake-assembly {
  position: relative;
  width: 240px; height: 320px;
  display: flex; flex-direction: column; align-items: center;
  justify-content: flex-end;
}
.cake-part {
  position: absolute;
  opacity: 0;
  transform: translateY(-80px);
}

/* Plate */
.plate { bottom: 20px; left: 50%; transform: translateX(-50%) translateY(-80px); width: 220px; height: 40px; }
.plate-top {
  width: 220px; height: 20px;
  background: radial-gradient(ellipse at center, #f0f0f0 0%, #d0d0d0 60%, #b0b0b0 100%);
  border-radius: 50%;
  box-shadow: 0 4px 12px rgba(0,0,0,0.3);
}
.plate-shadow {
  position: absolute; bottom: 0; left: 10px;
  width: 200px; height: 12px;
  background: radial-gradient(ellipse, rgba(0,0,0,0.2) 0%, transparent 70%);
  border-radius: 50%;
}

/* Cake body */
.cake-body { bottom: 55px; left: 50%; transform: translateX(-50%) translateY(-80px); width: 180px; height: 70px; }
.cake-body-top {
  width: 180px; height: 18px;
  background: linear-gradient(180deg, #fceabb 0%, #f5d47a 100%);
  border-radius: 10px;
}
.cake-body-front {
  width: 180px; height: 54px;
  background: linear-gradient(180deg, #f5d47a 0%, #e8b84b 50%, #d4a030 100%);
  border-radius: 0 0 8px 8px;
  border: 3px solid #c48a20;
  border-top: none;
}

/* Cream */
.cream-layer { bottom: 120px; left: 50%; transform: translateX(-50%) translateY(-80px); width: 190px; height: 30px; }
.cream-top {
  width: 190px; height: 16px;
  background: linear-gradient(180deg, #ffffff 0%, #fffcf0 100%);
  border-radius: 12px 12px 4px 4px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}
.cream-drip {
  position: absolute;
  width: 16px;
  background: linear-gradient(180deg, #fffcf0 0%, #fef5e7 100%);
  border-radius: 0 0 8px 8px;
}
.drip1 { height: 18px; bottom: -14px; left: 20px; }
.drip2 { height: 24px; bottom: -20px; left: 60px; }
.drip3 { height: 16px; bottom: -12px; right: 30px; }

/* Candle */
.candle { bottom: 168px; left: 50%; transform: translateX(-50%) translateY(-80px); width: 28px; height: 70px; }
.candle-body {
  width: 28px; height: 56px;
  background: linear-gradient(90deg, #ff6b8a 0%, #ffa0b4 30%, #ff6b8a 60%, #ffa0b4 100%);
  border-radius: 4px 4px 2px 2px;
  border: 2px solid #e05570;
}
.candle-stripe {
  position: absolute; left: 4px; right: 4px; height: 8px;
  background: rgba(255,220,100,0.7);
  border-radius: 2px;
}
.stripe1 { top: 14px; }
.stripe2 { top: 34px; }

/* Flame */
.flame {
  position: absolute; top: -32px; left: 50%;
  transform: translateX(-50%);
  opacity: 0;
  transition: opacity 0.3s;
}
.flame.lit { opacity: 1; animation: flicker 0.15s infinite alternate; }
.flame-inner {
  width: 20px; height: 28px;
  background: radial-gradient(ellipse at bottom, #fff 0%, #ffeb3b 30%, #ff9800 70%, transparent 100%);
  border-radius: 50% 50% 50% 50% / 60% 60% 40% 40%;
  box-shadow: 0 0 14px 6px rgba(255,150,0,0.6);
}
.flame-glow {
  position: absolute; top: 0; left: -6px;
  width: 32px; height: 38px;
  background: radial-gradient(circle, rgba(255,200,50,0.4) 0%, transparent 70%);
  border-radius: 50%;
}

@keyframes flicker {
  0% { transform: translateX(-50%) scale(1, 1) rotate(-2deg); }
  25% { transform: translateX(-50%) scale(1.05, 0.9) rotate(2deg); }
  50% { transform: translateX(-50%) scale(0.95, 1.05) rotate(-1deg); }
  75% { transform: translateX(-50%) scale(1.05, 0.95) rotate(3deg); }
  100% { transform: translateX(-50%) scale(1, 1) rotate(-2deg); }
}
```

- [ ] **Step 3: Verify cake renders (static, invisible until phase 2)**

Open `index.html`. Cake should be hidden (all parts opacity: 0). No visual change from previous state.

---

### Task 5: Cake drop animation sequence

**Files:**
- Modify: `index.html` (add JS + CSS keyframes)

- [ ] **Step 1: Add drop keyframes CSS**

Add before `</style>`:

```css
@keyframes dropIn {
  from { transform: translateY(-380px); opacity: 0; }
  to { opacity: 1; }
}
@keyframes dropPlate {
  0% { transform: translateX(-50%) translateY(-380px); opacity: 0; }
  60% { transform: translateX(-50%) translateY(10px); opacity: 1; }
  80% { transform: translateX(-50%) translateY(-6px); opacity: 1; }
  100% { transform: translateX(-50%) translateY(0); opacity: 1; }
}
@keyframes dropCake {
  0% { transform: translateX(-50%) translateY(-380px); opacity: 0; }
  55% { transform: translateX(-50%) translateY(8px); opacity: 1; }
  75% { transform: translateX(-50%) translateY(-4px); opacity: 1; }
  100% { transform: translateX(-50%) translateY(0); opacity: 1; }
}
@keyframes dropCream {
  0% { transform: translateX(-50%) translateY(-380px); opacity: 0; }
  50% { transform: translateX(-50%) translateY(5px); opacity: 1; }
  70% { transform: translateX(-50%) translateY(-3px); opacity: 1; }
  100% { transform: translateX(-50%) translateY(0); opacity: 1; }
}
@keyframes dropCandle {
  0% { transform: translateX(-50%) translateY(-380px); opacity: 0; }
  40% { transform: translateX(-50%) translateY(3px); opacity: 1; }
  60% { transform: translateX(-50%) translateY(-2px); opacity: 1; }
  100% { transform: translateX(-50%) translateY(0); opacity: 1; }
}
.plate.dropped  { animation: dropPlate 0.7s cubic-bezier(0.22, 0.61, 0.36, 1) forwards; }
.cake-body.dropped  { animation: dropCake 0.65s 0.5s cubic-bezier(0.22, 0.61, 0.36, 1) forwards; }
.cream-layer.dropped  { animation: dropCream 0.6s 1.0s cubic-bezier(0.22, 0.61, 0.36, 1) forwards; }
.candle.dropped  { animation: dropCandle 0.5s 1.4s cubic-bezier(0.22, 0.61, 0.36, 1) forwards; }
```

- [ ] **Step 2: Verify keyframes added, no JS yet**

Open `index.html`, confirm CSS keyframes are present in styles. No functional change yet — `startCakePhase` will be wired in Task 9.

---

### Task 6: Web Audio API birthday song

**Files:**
- Modify: `index.html` (add JS function)

- [ ] **Step 1: Add `playBirthdaySong()` function**

Add to the `<script>` block (before any callsite, or at the end — JS hoisting works for function declarations):

```javascript
function playBirthdaySong() {
  try {
    const ctx = new (window.AudioContext || window.webkitAudioContext)();
    const notes = [
      // Happy Birthday melody in Hz (C4=262)
      { freq: 262, dur: 0.25 }, // C - Hap
      { freq: 262, dur: 0.25 }, // C - py
      { freq: 294, dur: 0.5  }, // D - Birth
      { freq: 262, dur: 0.5  }, // C - day
      { freq: 349, dur: 0.5  }, // F - to
      { freq: 330, dur: 0.75 }, // E - you

      { freq: 262, dur: 0.25 }, // C - Hap
      { freq: 262, dur: 0.25 }, // C - py
      { freq: 294, dur: 0.5  }, // D - Birth
      { freq: 262, dur: 0.5  }, // C - day
      { freq: 392, dur: 0.5  }, // G - to
      { freq: 349, dur: 0.75 }, // F - you

      { freq: 262, dur: 0.25 }, // C - Hap
      { freq: 262, dur: 0.25 }, // C - py
      { freq: 523, dur: 0.5  }, // C5- Birth
      { freq: 440, dur: 0.5  }, // A - day
      { freq: 349, dur: 0.5  }, // F - dear
      { freq: 330, dur: 0.5  }, // E - 
      { freq: 294, dur: 0.75 }, // D - 

      { freq: 466, dur: 0.25 }, // Bb- Hap
      { freq: 466, dur: 0.25 }, // Bb- py
      { freq: 440, dur: 0.5  }, // A - Birth
      { freq: 349, dur: 0.5  }, // F - day
      { freq: 392, dur: 0.5  }, // G - to
      { freq: 349, dur: 0.75 }, // F - you
    ];

    let time = ctx.currentTime + 0.1;
    notes.forEach(({ freq, dur }) => {
      const osc = ctx.createOscillator();
      const gain = ctx.createGain();
      osc.type = 'sine';
      osc.frequency.value = freq;
      gain.gain.setValueAtTime(0.3, time);
      gain.gain.exponentialRampToValueAtTime(0.01, time + dur - 0.02);
      osc.connect(gain);
      gain.connect(ctx.destination);
      osc.start(time);
      osc.stop(time + dur);
      time += dur + 0.02; // slight gap between notes
    });
  } catch (e) {
    console.warn('Audio playback failed:', e);
  }
}
```

- [ ] **Step 2: Test audio**

Trigger cake phase, confirm melody plays after candle lights. On mobile, ensure it works after swipe interactions (AudioContext unlocked by touch).

---

### Task 7: Title "生日快乐" animation

**Files:**
- Modify: `index.html` (add JS + CSS)

- [ ] **Step 1: Add title character keyframes**

Add before `</style>`:

```css
@keyframes titleBounceIn {
  0% { opacity: 0; transform: scale(0) rotate(-15deg); }
  60% { opacity: 1; transform: scale(1.2) rotate(5deg); }
  80% { transform: scale(0.9) rotate(-2deg); }
  100% { opacity: 1; transform: scale(1) rotate(0deg); }
}
.title-char {
  color: #ffd700;
  text-shadow:
    0 0 10px rgba(255,215,0,0.8),
    0 0 20px rgba(255,180,0,0.5),
    0 3px 0 #e67700,
    0 6px 0 #cc5500,
    0 9px 12px rgba(0,0,0,0.3);
  opacity: 0;
  transform: scale(0);
}
.title-char.bounce-in {
  animation: titleBounceIn 0.6s cubic-bezier(0.34, 1.56, 0.64, 1) forwards;
}
```

- [ ] **Step 2: Add `animateTitle()` function**

Add to the `<script>` block:

```javascript
function animateTitle() {
  titleZone.style.opacity = '1';
  const chars = titleZone.querySelectorAll('.title-char');
  chars.forEach((char, i) => {
    setTimeout(() => {
      char.classList.add('bounce-in');
    }, i * 200);
  });
}
```

- [ ] **Step 3: Verify title animation**

Trigger cake phase. Confirm title zone appears and characters bounce in one by one: 生 → 日 → 快 → 乐.

---

### Task 8: Confetti and balloon effects

**Files:**
- Modify: `index.html` (add JS + CSS)

- [ ] **Step 1: Add confetti CSS**

Add before `</style>`:

```css
.confetti-piece {
  position: fixed;
  width: 12px; height: 12px;
  border-radius: 3px;
  pointer-events: none; z-index: 20;
  animation: confettiFall linear forwards;
}
@keyframes confettiFall {
  0% {
    transform: translateY(-120px) rotate(0deg) scale(1);
    opacity: 1;
  }
  80% { opacity: 1; }
  100% {
    transform: translateY(105vh) rotate(720deg) scale(0.3);
    opacity: 0;
  }
}
.balloon {
  position: fixed;
  width: 40px; height: 50px;
  border-radius: 50%;
  pointer-events: none; z-index: 15;
  animation: balloonRise linear forwards;
}
.balloon::before {
  content: '';
  position: absolute; bottom: -10px; left: 50%;
  transform: translateX(-50%);
  width: 0; height: 0;
  border-left: 4px solid transparent;
  border-right: 4px solid transparent;
  border-top: 8px solid inherit;
}
.balloon::after {
  content: '';
  position: absolute; bottom: -24px; left: 50%;
  width: 1px; height: 16px;
  background: rgba(255,255,255,0.5);
}
@keyframes balloonRise {
  0% {
    transform: translateY(0) translateX(0) rotate(0deg);
    opacity: 0.9;
  }
  100% {
    transform: translateY(-110vh) translateX(60px) rotate(20deg);
    opacity: 0;
  }
}
```

- [ ] **Step 2: Add `spawnConfetti()` function**

Add to the `<script>` block:

```javascript
function spawnConfetti() {
  const colors = ['#ff6b9d', '#ffd700', '#66ccff', '#66ff99', '#ff9966', '#cc66ff', '#ff6699'];
  const fragment = document.createDocumentFragment();

  // Confetti
  for (let i = 0; i < 60; i++) {
    const piece = document.createElement('div');
    piece.className = 'confetti-piece';
    piece.style.left = Math.random() * 100 + '%';
    piece.style.top = -(Math.random() * 40) + 'px';
    piece.style.backgroundColor = colors[Math.floor(Math.random() * colors.length)];
    piece.style.width = (8 + Math.random() * 14) + 'px';
    piece.style.height = (8 + Math.random() * 14) + 'px';
    piece.style.animationDuration = (2.5 + Math.random() * 3) + 's';
    piece.style.animationDelay = Math.random() * 2 + 's';
    fragment.appendChild(piece);
  }

  // Balloons
  const balloonColors = ['#ff6b9d', '#ffd700', '#66ccff', '#66ff99'];
  for (let i = 0; i < 8; i++) {
    const balloon = document.createElement('div');
    balloon.className = 'balloon';
    balloon.style.left = (5 + Math.random() * 90) + '%';
    balloon.style.bottom = -(60 + Math.random() * 40) + 'px';
    balloon.style.backgroundColor = balloonColors[Math.floor(Math.random() * balloonColors.length)];
    balloon.style.animationDuration = (6 + Math.random() * 5) + 's';
    balloon.style.animationDelay = Math.random() * 1.5 + 's';
    fragment.appendChild(balloon);
  }

  document.body.appendChild(fragment);

  // Clean up after animations
  setTimeout(() => {
    document.querySelectorAll('.confetti-piece, .balloon').forEach(el => el.remove());
  }, 12000);
}
```

- [ ] **Step 3: Verify confetti and balloons**

Trigger cake phase. Confirm multicolored confetti pieces fall from top, balloons rise from bottom, all clean up after 12 seconds.

---

### Task 9: Integration, timing, and edge cases

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Add `startCakePhase()` wiring function (calls all sub-functions from Tasks 6-8)**

Add after the `animateTitle` function (from Task 7):

```javascript
function startCakePhase() {
  phase = 'cake';
  hintZone.style.opacity = '0';

  wishEls.forEach(el => { el.classList.remove('active'); el.style.display = 'none'; });
  document.querySelector('.wish-container').style.display = 'none';

  cakeStage.style.opacity = '1';
  cakeStage.style.pointerEvents = 'auto';

  setTimeout(() => document.querySelector('.plate').classList.add('dropped'), 100);
  setTimeout(() => document.querySelector('.cake-body').classList.add('dropped'), 600);
  setTimeout(() => document.querySelector('.cream-layer').classList.add('dropped'), 1100);
  setTimeout(() => document.querySelector('.candle').classList.add('dropped'), 1600);

  setTimeout(() => {
    document.querySelector('.flame').classList.add('lit');
    playBirthdaySong();
    animateTitle();
    spawnConfetti();
  }, 2100);

  setTimeout(() => { hintZone.style.display = 'none'; }, 500);
}
```

- [ ] **Step 2: Prevent overscroll on mobile**

Add to `body` style:

```css
body {
  /* existing styles AND */
  touch-action: none;
  -webkit-overflow-scrolling: none;
  overscroll-behavior: none;
}
```

And add JS:

```javascript
// Prevent pull-to-refresh and overscroll
document.addEventListener('touchmove', e => e.preventDefault(), { passive: false });
```

- [ ] **Step 3: Handle desktop mouse drag (suppress text selection during swipe)**

Add CSS:

```css
.card { user-select: none; -webkit-user-select: none; }
```

- [ ] **Step 4: Full end-to-end test**

Checklist:
- [ ] Page loads, first wish visible with cartoon text effect
- [ ] Swipe up → next wish (elastic transition), swipe down → previous wish
- [ ] Hint arrow bounces at bottom, text updates on last wish
- [ ] Last wish swipe → cake drop sequence begins (plate → cake → cream → candle → flame)
- [ ] Flame flickers
- [ ] Birthday song plays (synthesized melody)
- [ ] Title "生日快乐" bounces in character by character
- [ ] Confetti falls, balloons rise
- [ ] All animations smooth (60fps)
- [ ] Works on mobile viewport (375px width)
- [ ] No overscroll / pull-to-refresh interference

- [ ] **Step 5: Final page complete, no further changes needed**
