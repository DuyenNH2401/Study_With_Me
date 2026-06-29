# Study With Me Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a single-file HTML "Study With Me" page with analog Pomodoro timer (50/10), hidden YouTube audio embed, todo list with localStorage, cozy glassmorphism UI, and pixel art campfire GIF background.

**Architecture:** Single `index.html` file containing all CSS in `<style>` and all JS in `<script>`. Layout is a centered glassmorphism card over full-screen GIF background. Three vertical sections: analog clock with SVG ring progress, YouTube input + hidden iframe, CRUD todo list. Timer uses `Date` timestamps for cross-tab accuracy.

**Tech Stack:** Plain HTML5, CSS3 (flexbox, CSS variables, SVG), vanilla JavaScript (Web Audio API for chime), Google Fonts (Nunito).

## Global Constraints

- File must work when opened directly in browser (`file://` protocol)
- No build step, no dependencies, no npm
- YouTube embed via iframe (no API key needed)
- `localStorage` for todo persistence
- Font: Nunito from Google Fonts CDN
- Working color: `#e07a5f`, break color: `#81b29a`, text: `#fefae0`, card bg: `rgba(255,255,255,0.12)`
- Background GIF: `Lonely Camp Fire GIF.gif` with `background-size: cover`
- Fallback gradient if GIF fails to load
- Pomodoro: 50 min work / 10 min break

---

### Task 1: HTML Scaffold & Base CSS

**Files:**
- Create: `index.html`

**Interfaces:**
- Produces: HTML structure with all sections (clock, youtube, todo), CSS variables, glassmorphism card, background GIF, Google Fonts import

- [ ] **Step 1: Create index.html with full HTML structure and base CSS**

Write the complete file:

```html
<!DOCTYPE html>
<html lang="vi">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>☕ Study With Me</title>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Nunito:wght@300;400;600;700&display=swap" rel="stylesheet">
<style>
*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

:root {
  --color-work: #e07a5f;
  --color-break: #81b29a;
  --color-text: #fefae0;
  --color-text-muted: rgba(254, 250, 224, 0.7);
  --color-card: rgba(30, 20, 15, 0.55);
  --color-card-border: rgba(255, 255, 255, 0.1);
  --color-input-bg: rgba(255, 255, 255, 0.08);
  --color-danger: #e07a5f;
  --radius: 20px;
  --blur: 16px;
}

body {
  font-family: 'Nunito', sans-serif;
  color: var(--color-text);
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  background: linear-gradient(135deg, #1a0e06 0%, #2d1810 50%, #1a1208 100%);
  background-image: url('Lonely Camp Fire GIF.gif');
  background-size: cover;
  background-position: center;
  background-repeat: no-repeat;
  background-attachment: fixed;
}

body::before {
  content: '';
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.45);
  z-index: 0;
}

.card {
  position: relative;
  z-index: 1;
  width: 440px;
  max-width: 92vw;
  background: var(--color-card);
  backdrop-filter: blur(var(--blur));
  -webkit-backdrop-filter: blur(var(--blur));
  border: 1px solid var(--color-card-border);
  border-radius: var(--radius);
  padding: 36px 32px;
  display: flex;
  flex-direction: column;
  gap: 28px;
}

.title {
  text-align: center;
  font-size: 1.6rem;
  font-weight: 700;
  letter-spacing: 0.02em;
}

.section {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.section-label {
  font-size: 0.8rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.08em;
  color: var(--color-text-muted);
}
</style>
</head>
<body>
<div class="card">
  <h1 class="title">☕ Study With Me</h1>

  <!-- Pomodoro section -->
  <div class="section" id="pomodoro-section">
    <span class="section-label">⏱ Pomodoro</span>
    <!-- clock + controls go here in Task 2 -->
  </div>

  <!-- YouTube section -->
  <div class="section" id="youtube-section">
    <span class="section-label">🎵 Nhạc nền</span>
    <!-- input + player go here in Task 3 -->
  </div>

  <!-- Todo section -->
  <div class="section" id="todo-section">
    <span class="section-label">📝 Việc cần làm</span>
    <!-- todo list + input go here in Task 4 -->
  </div>
</div>

<!-- Hidden YouTube iframe (Task 3) -->
</body>
</html>
```

- [ ] **Step 2: Open index.html in browser to verify layout**

Run: Open `index.html` in browser
Expected: See centered glassmorphism card over campfire GIF background, section labels visible, no errors in console

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add HTML scaffold with base CSS and glassmorphism card

Co-Authored-By: Claude <noreply@anthropic.com>"
```

---

### Task 2: Pomodoro Analog Clock

**Files:**
- Modify: `index.html` — add clock SVG/CSS and timer JS

**Interfaces:**
- Consumes: `#pomodoro-section` div from Task 1
- Produces: `startPomodoro()`, `pausePomodoro()`, `resetPomodoro()` functions; clock DOM with `.clock-face`, `.progress-ring__circle`, `.clock-hand`, `.clock-center`; mode indicator `.mode-badge`

- [ ] **Step 1: Add clock CSS and HTML into index.html**

Add inside `<style>`, after existing CSS:

```css
/* ---- Pomodoro Clock ---- */
.pomo-container { display: flex; flex-direction: column; align-items: center; gap: 16px; }

.clock-wrapper { position: relative; width: 200px; height: 200px; }

.clock-face {
  width: 100%; height: 100%;
  border-radius: 50%;
  border: 3px solid rgba(255,255,255,0.15);
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
}

/* SVG progress ring */
.progress-ring {
  position: absolute; inset: -8px;
  transform: rotate(-90deg);
  width: calc(100% + 16px); height: calc(100% + 16px);
}

.progress-ring__circle {
  fill: none;
  stroke: var(--color-work);
  stroke-width: 4;
  stroke-linecap: round;
  transition: stroke-dashoffset 1s linear, stroke 0.4s;
}

.progress-ring__circle.break { stroke: var(--color-break); }

/* Clock hand */
.clock-hand {
  position: absolute;
  width: 3px; height: 44%;
  background: var(--color-text);
  border-radius: 2px;
  bottom: 50%;
  left: 50%;
  transform-origin: bottom center;
  transform: translateX(-50%) rotate(0deg);
  transition: transform 1s linear;
  opacity: 0.9;
}

.clock-center {
  position: absolute;
  width: 10px; height: 10px;
  background: var(--color-text);
  border-radius: 50%;
  z-index: 2;
}

.clock-time {
  font-size: 2.2rem;
  font-weight: 700;
  letter-spacing: 0.04em;
  font-variant-numeric: tabular-nums;
}

.mode-badge {
  font-size: 0.75rem;
  font-weight: 600;
  padding: 4px 14px;
  border-radius: 999px;
  background: var(--color-work);
  color: #fff;
  transition: background 0.4s;
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

.mode-badge.break { background: var(--color-break); }

.pomo-controls { display: flex; gap: 12px; }

.pomo-btn {
  font-family: inherit;
  font-size: 0.9rem;
  font-weight: 600;
  padding: 8px 22px;
  border: none;
  border-radius: 999px;
  cursor: pointer;
  transition: all 0.25s;
  letter-spacing: 0.03em;
}

.pomo-btn--start {
  background: var(--color-work);
  color: #fff;
}

.pomo-btn--start:hover { filter: brightness(1.12); transform: translateY(-1px); }

.pomo-btn--reset {
  background: transparent;
  color: var(--color-text-muted);
  border: 1px solid rgba(255,255,255,0.2);
}

.pomo-btn--reset:hover { background: rgba(255,255,255,0.08); color: var(--color-text); }
```

Replace the `<!-- Pomodoro section -->` comment and its contents with:

```html
  <!-- Pomodoro section -->
  <div class="section" id="pomodoro-section">
    <span class="section-label">⏱ Pomodoro</span>
    <div class="pomo-container">
      <div class="mode-badge" id="mode-badge">🎯 Tập trung</div>
      <div class="clock-wrapper">
        <svg class="progress-ring" viewBox="0 0 216 216">
          <circle class="progress-ring__circle" id="progress-circle"
            cx="108" cy="108" r="102"
            stroke-dasharray="640.88"
            stroke-dashoffset="0" />
        </svg>
        <div class="clock-face">
          <div class="clock-hand" id="clock-hand"></div>
          <div class="clock-center"></div>
          <div class="clock-time" id="clock-time">50:00</div>
        </div>
      </div>
      <div class="pomo-controls">
        <button class="pomo-btn pomo-btn--start" id="btn-start">▶ Bắt đầu</button>
        <button class="pomo-btn pomo-btn--reset" id="btn-reset">↺ Reset</button>
      </div>
    </div>
  </div>
```

- [ ] **Step 2: Add Pomodoro JavaScript**

Add inside `<script>` at end of `<body>` (before `</body>`):

```html
<script>
// ==================== POMODORO ====================
const WORK_MINUTES = 50;
const BREAK_MINUTES = 10;

const clockTime = document.getElementById('clock-time');
const clockHand = document.getElementById('clock-hand');
const progressCircle = document.getElementById('progress-circle');
const modeBadge = document.getElementById('mode-badge');
const btnStart = document.getElementById('btn-start');
const btnReset = document.getElementById('btn-reset');

const CIRCUMFERENCE = 2 * Math.PI * 102; // ~640.88
progressCircle.style.strokeDasharray = CIRCUMFERENCE;
progressCircle.style.strokeDashoffset = '0';

let pomoState = {
  mode: 'work',        // 'work' | 'break'
  running: false,
  totalSeconds: WORK_MINUTES * 60,
  remainingSeconds: WORK_MINUTES * 60,
  endTime: null,       // Date.now() + remainingSeconds*1000 when running
  intervalId: null
};

function formatTime(seconds) {
  const m = Math.floor(seconds / 60);
  const s = seconds % 60;
  return `${String(m).padStart(2, '0')}:${String(s).padStart(2, '0')}`;
}

function updateDisplay() {
  clockTime.textContent = formatTime(pomoState.remainingSeconds);
  const ratio = pomoState.remainingSeconds / pomoState.totalSeconds;
  progressCircle.style.strokeDashoffset = CIRCUMFERENCE * (1 - ratio);
  // Clock hand: full circle = totalSeconds, map to 360 degrees
  const degrees = (ratio * 360);
  clockHand.style.transform = `translateX(-50%) rotate(${degrees}deg)`;
}

function updateModeUI() {
  const isWork = pomoState.mode === 'work';
  modeBadge.textContent = isWork ? '🎯 Tập trung' : '☕ Nghỉ ngơi';
  modeBadge.classList.toggle('break', !isWork);
  progressCircle.classList.toggle('break', !isWork);
  btnStart.style.background = isWork ? 'var(--color-work)' : 'var(--color-break)';
  document.querySelector('.pomo-container').style.animation = 'none';
  void document.querySelector('.pomo-container').offsetHeight;
  document.querySelector('.pomo-container').style.animation = 'shake 0.4s ease';
}

function tick() {
  if (!pomoState.running) return;
  const now = Date.now();
  const remaining = Math.max(0, Math.ceil((pomoState.endTime - now) / 1000));
  pomoState.remainingSeconds = remaining;
  updateDisplay();

  if (remaining <= 0) {
    switchMode();
    playChime();
  }
}

function startPomodoro() {
  if (pomoState.running) {
    // Pause
    pomoState.running = false;
    clearInterval(pomoState.intervalId);
    btnStart.textContent = '▶ Bắt đầu';
    return;
  }
  // Start or resume
  pomoState.running = true;
  pomoState.endTime = Date.now() + pomoState.remainingSeconds * 1000;
  pomoState.intervalId = setInterval(tick, 200);
  btnStart.textContent = '⏸ Tạm dừng';
}

function resetPomodoro() {
  pomoState.running = false;
  clearInterval(pomoState.intervalId);
  pomoState.mode = 'work';
  pomoState.totalSeconds = WORK_MINUTES * 60;
  pomoState.remainingSeconds = WORK_MINUTES * 60;
  pomoState.endTime = null;
  btnStart.textContent = '▶ Bắt đầu';
  updateModeUI();
  updateDisplay();
}

function switchMode() {
  pomoState.running = false;
  clearInterval(pomoState.intervalId);
  if (pomoState.mode === 'work') {
    pomoState.mode = 'break';
    pomoState.totalSeconds = BREAK_MINUTES * 60;
  } else {
    pomoState.mode = 'work';
    pomoState.totalSeconds = WORK_MINUTES * 60;
  }
  pomoState.remainingSeconds = pomoState.totalSeconds;
  pomoState.endTime = null;
  btnStart.textContent = '▶ Bắt đầu';
  updateModeUI();
  updateDisplay();
}

function playChime() {
  try {
    const ctx = new (window.AudioContext || window.webkitAudioContext)();
    [523.25, 659.25, 783.99].forEach((freq, i) => {
      const osc = ctx.createOscillator();
      const gain = ctx.createGain();
      osc.type = 'sine';
      osc.frequency.value = freq;
      gain.gain.setValueAtTime(0.15, ctx.currentTime + i * 0.18);
      gain.gain.exponentialRampToValueAtTime(0.001, ctx.currentTime + i * 0.18 + 0.4);
      osc.connect(gain); gain.connect(ctx.destination);
      osc.start(ctx.currentTime + i * 0.18);
      osc.stop(ctx.currentTime + i * 0.18 + 0.4);
    });
  } catch(e) { /* Web Audio not available */ }
}

// Add shake keyframe animation to <style>
const shakeStyle = document.createElement('style');
shakeStyle.textContent = `
  @keyframes shake {
    0%, 100% { transform: translateX(0); }
    20% { transform: translateX(-6px); }
    40% { transform: translateX(6px); }
    60% { transform: translateX(-4px); }
    80% { transform: translateX(4px); }
  }
`;
document.head.appendChild(shakeStyle);

btnStart.addEventListener('click', startPomodoro);
btnReset.addEventListener('click', resetPomodoro);
updateDisplay();
</script>
```

- [ ] **Step 3: Open index.html in browser to verify Pomodoro**

Expected:
- Clock face visible with "50:00" centered
- Click "Bắt đầu" → timer counts down, hand rotates, ring shrinks
- Click again → pauses ("Tạm dừng")  
- Click again → resumes
- Timer hits 0 → chime plays, switches to "Nghỉ ngơi" 10:00, color changes to mint green
- Reset → back to 50:00 work mode

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add Pomodoro analog clock with SVG ring and countdown timer

Co-Authored-By: Claude <noreply@anthropic.com>"
```

---

### Task 3: YouTube Audio Player

**Files:**
- Modify: `index.html` — add YouTube section HTML, CSS, and JS

**Interfaces:**
- Consumes: `#youtube-section` div from Task 1, card layout
- Produces: `loadYoutube(url)` function; hidden iframe `#youtube-player`; input `#yt-input`; status text `#yt-status`

- [ ] **Step 1: Add YouTube CSS**

Add inside `<style>`, before `</style>`:

```css
/* ---- YouTube ---- */
.yt-form { display: flex; gap: 10px; }

.yt-input {
  flex: 1;
  font-family: inherit;
  font-size: 0.85rem;
  padding: 10px 14px;
  border: 1px solid rgba(255,255,255,0.15);
  border-radius: 12px;
  background: var(--color-input-bg);
  color: var(--color-text);
  outline: none;
  transition: border-color 0.25s;
}

.yt-input::placeholder { color: rgba(254,250,224,0.35); }

.yt-input:focus { border-color: rgba(255,255,255,0.35); }

.yt-btn {
  font-family: inherit;
  font-size: 0.85rem;
  font-weight: 600;
  padding: 10px 18px;
  border: none;
  border-radius: 12px;
  background: rgba(255,255,255,0.12);
  color: var(--color-text);
  cursor: pointer;
  transition: all 0.25s;
  white-space: nowrap;
}

.yt-btn:hover { background: rgba(255,255,255,0.2); }

.yt-status {
  font-size: 0.78rem;
  color: var(--color-text-muted);
  min-height: 1.2em;
}

.yt-status.error { color: #f4a261; }
```

- [ ] **Step 2: Add YouTube HTML**

Replace the `<!-- YouTube section -->` comment and its contents with:

```html
  <!-- YouTube section -->
  <div class="section" id="youtube-section">
    <span class="section-label">🎵 Nhạc nền</span>
    <div class="yt-form">
      <input class="yt-input" id="yt-input" type="text"
        placeholder="Dán link YouTube vào đây..."
        autocomplete="off" />
      <button class="yt-btn" id="yt-load-btn">🎧 Load</button>
    </div>
    <div class="yt-status" id="yt-status"></div>
  </div>
```

- [ ] **Step 3: Add hidden YouTube iframe**

Add right after the `</div>` closing `.card` and before `</body>`:

```html
<div id="yt-player-wrapper" style="position:fixed;top:0;left:0;width:1px;height:1px;opacity:0;pointer-events:none;z-index:-1;">
  <div id="yt-player"></div>
</div>
```

- [ ] **Step 4: Add YouTube JavaScript**

Add inside `<script>`, after Pomodoro code:

```javascript
// ==================== YOUTUBE ====================
const ytInput = document.getElementById('yt-input');
const ytLoadBtn = document.getElementById('yt-load-btn');
const ytStatus = document.getElementById('yt-status');

let ytPlayer = null;

function extractYoutubeId(url) {
  if (!url || !url.trim()) return null;
  const patterns = [
    /(?:youtube\.com\/watch\?v=)([a-zA-Z0-9_-]{11})/,
    /(?:youtu\.be\/)([a-zA-Z0-9_-]{11})/,
    /(?:youtube\.com\/embed\/)([a-zA-Z0-9_-]{11})/,
    /(?:youtube\.com\/live\/)([a-zA-Z0-9_-]{11})/,
  ];
  for (const p of patterns) {
    const match = url.match(p);
    if (match) return match[1];
  }
  return null;
}

function loadYoutube(url) {
  const videoId = extractYoutubeId(url);
  if (!videoId) {
    ytStatus.textContent = '⚠ Link YouTube không hợp lệ. Thử lại nhé!';
    ytStatus.className = 'yt-status error';
    return;
  }

  ytStatus.textContent = '⏳ Đang tải...';
  ytStatus.className = 'yt-status';

  if (!ytPlayer) {
    // YouTube IFrame API will call onYouTubeIframeAPIReady
    window.onYouTubeIframeAPIReady = function() {
      createPlayer(videoId);
    };
    // Load API if not already loaded
    if (!document.getElementById('yt-api-script')) {
      const tag = document.createElement('script');
      tag.id = 'yt-api-script';
      tag.src = 'https://www.youtube.com/iframe_api';
      const firstScript = document.getElementsByTagName('script')[0];
      firstScript.parentNode.insertBefore(tag, firstScript);
    } else {
      createPlayer(videoId);
    }
  } else {
    ytPlayer.loadVideoById(videoId);
    ytPlayer.playVideo();
    ytStatus.textContent = '🎵 Đang phát...';
    ytStatus.className = 'yt-status';
  }
}

function createPlayer(videoId) {
  ytPlayer = new YT.Player('yt-player', {
    videoId: videoId,
    playerVars: {
      autoplay: 1,
      controls: 0,
      modestbranding: 1,
      iv_load_policy: 3,
      rel: 0,
      showinfo: 0,
    },
    events: {
      onReady: function() {
        ytPlayer.playVideo();
        ytStatus.textContent = '🎵 Đang phát...';
        ytStatus.className = 'yt-status';
      },
      onError: function() {
        ytStatus.textContent = '⚠ Không thể phát video này.';
        ytStatus.className = 'yt-status error';
      },
    },
  });
}

ytLoadBtn.addEventListener('click', () => {
  const url = ytInput.value.trim();
  if (!url) {
    ytInput.focus();
    ytStatus.textContent = 'Vui lòng nhập link YouTube.';
    ytStatus.className = 'yt-status error';
    return;
  }
  loadYoutube(url);
});

ytInput.addEventListener('keydown', (e) => {
  if (e.key === 'Enter') {
    ytLoadBtn.click();
  }
});
```

- [ ] **Step 5: Open index.html in browser to verify YouTube**

Expected:
- Input field visible with placeholder text
- Paste a YouTube link (e.g. `https://youtu.be/jfKfPfyJRdk`) and click Load
- Status shows "Đang tải..." then "🎵 Đang phát..."
- Audio plays from the video (no video visible)
- Paste invalid link → shows error message
- Empty input + Load → focuses input, shows message

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "feat: add YouTube audio player with hidden iframe embed

Co-Authored-By: Claude <noreply@anthropic.com>"
```

---

### Task 4: Todo List with localStorage

**Files:**
- Modify: `index.html` — add todo HTML, CSS, and JS with localStorage persistence

**Interfaces:**
- Consumes: `#todo-section` div from Task 1
- Produces: `addTodo(text)` function; `renderTodos()` function; todo items array in `localStorage` under key `studywithme-todos`

- [ ] **Step 1: Add Todo CSS**

Add inside `<style>`, before `</style>`:

```css
/* ---- Todo ---- */
.todo-form { display: flex; gap: 10px; }

.todo-input {
  flex: 1;
  font-family: inherit;
  font-size: 0.85rem;
  padding: 10px 14px;
  border: 1px solid rgba(255,255,255,0.15);
  border-radius: 12px;
  background: var(--color-input-bg);
  color: var(--color-text);
  outline: none;
  transition: border-color 0.25s;
}

.todo-input::placeholder { color: rgba(254,250,224,0.35); }
.todo-input:focus { border-color: rgba(255,255,255,0.35); }

.todo-btn {
  font-family: inherit;
  font-size: 0.85rem;
  font-weight: 600;
  padding: 10px 18px;
  border: none;
  border-radius: 12px;
  background: rgba(255,255,255,0.12);
  color: var(--color-text);
  cursor: pointer;
  transition: all 0.25s;
  white-space: nowrap;
}

.todo-btn:hover { background: rgba(255,255,255,0.2); }

.todo-list {
  list-style: none;
  display: flex;
  flex-direction: column;
  gap: 6px;
  max-height: 200px;
  overflow-y: auto;
}

.todo-list:empty::after {
  content: 'Chưa có việc nào...';
  color: var(--color-text-muted);
  font-size: 0.82rem;
  text-align: center;
  padding: 12px 0;
  display: block;
}

.todo-item {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 8px 12px;
  background: rgba(255,255,255,0.05);
  border-radius: 10px;
  transition: background 0.2s;
}

.todo-item:hover { background: rgba(255,255,255,0.08); }

.todo-checkbox {
  appearance: none;
  -webkit-appearance: none;
  width: 18px; height: 18px;
  border: 2px solid rgba(255,255,255,0.3);
  border-radius: 6px;
  cursor: pointer;
  flex-shrink: 0;
  transition: all 0.2s;
  position: relative;
}

.todo-checkbox:checked {
  background: var(--color-work);
  border-color: var(--color-work);
}

.todo-checkbox:checked::after {
  content: '✓';
  position: absolute;
  top: 50%; left: 50%;
  transform: translate(-50%, -50%);
  font-size: 10px;
  font-weight: 700;
  color: #fff;
}

.todo-text {
  flex: 1;
  font-size: 0.88rem;
  transition: all 0.25s;
}

.todo-text.done {
  text-decoration: line-through;
  opacity: 0.45;
}

.todo-delete {
  background: none;
  border: none;
  color: rgba(255,255,255,0.25);
  cursor: pointer;
  font-size: 1rem;
  padding: 2px 6px;
  border-radius: 6px;
  transition: all 0.2s;
  line-height: 1;
}

.todo-delete:hover { color: var(--color-danger); background: rgba(255,255,255,0.1); }
```

- [ ] **Step 2: Add Todo HTML**

Replace the `<!-- Todo section -->` comment and its contents with:

```html
  <!-- Todo section -->
  <div class="section" id="todo-section">
    <span class="section-label">📝 Việc cần làm</span>
    <div class="todo-form">
      <input class="todo-input" id="todo-input" type="text"
        placeholder="Thêm việc mới..."
        autocomplete="off" maxlength="200" />
      <button class="todo-btn" id="todo-add-btn">+ Thêm</button>
    </div>
    <ul class="todo-list" id="todo-list"></ul>
  </div>
```

- [ ] **Step 3: Add Todo JavaScript**

Add inside `<script>`, after YouTube code:

```javascript
// ==================== TODO LIST ====================
const STORAGE_KEY = 'studywithme-todos';
const todoInput = document.getElementById('todo-input');
const todoAddBtn = document.getElementById('todo-add-btn');
const todoList = document.getElementById('todo-list');

let todos = [];

function loadTodos() {
  try {
    const raw = localStorage.getItem(STORAGE_KEY);
    if (raw) {
      todos = JSON.parse(raw);
    }
  } catch(e) {
    todos = [];
  }
}

function saveTodos() {
  try {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(todos));
  } catch(e) { /* storage full or unavailable, silently fail */ }
}

function renderTodos() {
  todoList.innerHTML = '';
  todos.forEach((todo, index) => {
    const li = document.createElement('li');
    li.className = 'todo-item';

    const cb = document.createElement('input');
    cb.type = 'checkbox';
    cb.className = 'todo-checkbox';
    cb.checked = todo.done;
    cb.addEventListener('change', () => {
      todos[index].done = cb.checked;
      saveTodos();
      renderTodos();
    });

    const span = document.createElement('span');
    span.className = 'todo-text' + (todo.done ? ' done' : '');
    span.textContent = todo.text;

    const delBtn = document.createElement('button');
    delBtn.className = 'todo-delete';
    delBtn.innerHTML = '✕';
    delBtn.title = 'Xóa';
    delBtn.addEventListener('click', () => {
      todos.splice(index, 1);
      saveTodos();
      renderTodos();
    });

    li.appendChild(cb);
    li.appendChild(span);
    li.appendChild(delBtn);
    todoList.appendChild(li);
  });
}

function addTodo(text) {
  const trimmed = text.trim();
  if (!trimmed) return;
  todos.push({ text: trimmed, done: false });
  saveTodos();
  renderTodos();
  todoInput.value = '';
  todoInput.focus();
}

todoAddBtn.addEventListener('click', () => {
  addTodo(todoInput.value);
});

todoInput.addEventListener('keydown', (e) => {
  if (e.key === 'Enter') {
    addTodo(todoInput.value);
  }
});

// Initialize
loadTodos();
renderTodos();
```

- [ ] **Step 4: Open index.html in browser to verify Todo**

Expected:
- Empty list shows "Chưa có việc nào..."
- Type a task + Enter → appears in list with checkbox
- Check checkbox → text crossed out, dimmed
- Uncheck → text restored
- Click ✕ → item removed
- Refresh page → todos persist from localStorage
- Multiple items scrollable (max-height 200px)

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: add todo list with localStorage persistence

Co-Authored-By: Claude <noreply@anthropic.com>"
```

---

### Task 5: Final Polish & Integration

**Files:**
- Modify: `index.html` — refine spacing, transitions, responsive tweaks, final visual polish

**Interfaces:**
- Consumes: All prior tasks' DOM elements and JS functions
- Produces: Polished, responsive final page

- [ ] **Step 1: Add responsive CSS and final polish styles**

Add inside `<style>`, before `</style>`:

```css
/* ---- Scrollbar ---- */
.todo-list::-webkit-scrollbar { width: 4px; }
.todo-list::-webkit-scrollbar-track { background: transparent; }
.todo-list::-webkit-scrollbar-thumb { background: rgba(255,255,255,0.15); border-radius: 4px; }

/* ---- Card hover lift ---- */
.card { transition: transform 0.3s ease, box-shadow 0.3s ease; }
.card:hover { box-shadow: 0 8px 40px rgba(0,0,0,0.3); }

/* ---- Responsive ---- */
@media (max-width: 480px) {
  .card { padding: 28px 20px; gap: 22px; }
  .clock-wrapper { width: 160px; height: 160px; }
  .clock-time { font-size: 1.8rem; }
  .title { font-size: 1.3rem; }
  .pomo-btn { padding: 7px 16px; font-size: 0.82rem; }
}
```

- [ ] **Step 2: Add keyboard shortcut for Pomodoro**

Add inside `<script>`, at the end but before `</script>`:

```javascript
// ==================== KEYBOARD SHORTCUTS ====================
document.addEventListener('keydown', (e) => {
  // Space to toggle timer (only when not typing in inputs)
  if (e.code === 'Space' && e.target === document.body) {
    e.preventDefault();
    startPomodoro();
  }
});
```

- [ ] **Step 3: Open index.html in browser for final verification**

Run through the full checklist:
- [ ] Background GIF loads and covers screen, overlay darkens it
- [ ] Glassmorphism card centered with proper blur
- [ ] Pomodoro: start/pause/reset all work, countdown accurate, clock hand + ring animate, mode switch with color change, chime sounds
- [ ] YouTube: valid link loads and plays audio, invalid link shows error, empty input handled
- [ ] Todo: add/check/uncheck/delete items, persist across page refresh
- [ ] Responsive layout on narrow screen (phone)
- [ ] Spacebar starts/pauses timer when not typing
- [ ] No console errors

- [ ] **Step 4: Final commit**

```bash
git add index.html
git commit -m "feat: polish UI, add responsive styles, keyboard shortcuts

Co-Authored-By: Claude <noreply@anthropic.com>"
```
