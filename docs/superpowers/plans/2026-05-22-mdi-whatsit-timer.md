# MDI What's It? Timer — Implementation Plan

> **For agentic workers:** REQUIRED: Use superpowers:subagent-driven-development (if subagents available) or superpowers:executing-plans to implement this plan. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a single self-contained `index.html` round timer for the MDI "What's It?" party game that cycles through ROUND → TIMEUP → SWITCH → GO states indefinitely.

**Architecture:** Everything lives in one `index.html` file — inline `<style>` for CSS (with custom properties), inline `<script>` for vanilla JS state machine, and `<audio>` elements for sound. No build step, no dependencies beyond Google Fonts. The state machine is driven by a single `setState(name)` function that swaps classes and manages intervals.

**Tech Stack:** HTML5, CSS3 custom properties, Vanilla JS (ES6), Google Fonts (Holtwood One SC + Instrument Sans), HTML5 Audio API.

**Spec:** `PROJECT-PLAN.md` in the root directory — read it before implementing any task.

---

## Chunk 1: Foundation & Static UI

### Task 1: HTML Boilerplate + Fonts + CSS Tokens

**Files:**
- Create: `index.html`

- [ ] **Step 1: Create the HTML shell with meta tags and Google Fonts import**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>What's It? Timer</title>
  <style>
    @import url('https://fonts.googleapis.com/css2?family=Holtwood+One+SC&family=Instrument+Sans:wght@400;500;600&display=swap');
  </style>
</head>
<body>
</body>
</html>
```

- [ ] **Step 2: Add all CSS custom properties inside `<style>`**

Add directly after the `@import` line:

```css
    :root {
      --bg-screen:        #fcf7f2;
      --itsit-blue:       #4e8dbc;
      --itsit-green:      #74c486;
      --itsit-green-dark: #033f2c;
      --shadow:           #e5e5e6;
      --timeup-pink:      #f178b6;
      --font-display:     'Holtwood One SC', serif;
      --font-ui:          'Instrument Sans', sans-serif;
      --timer-size:       clamp(100px, 16vw, 260px);
      --status-size:      16px;
      --button-size:      14.7px;
      --branding-size:    20.668px;
      --timer-tracking:   9.8px;
      --status-tracking:  2px;
      --brand-tracking:   3.875px;
    }

    *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

    html, body {
      width: 100vw;
      height: 100vh;
      overflow: hidden;
      background: var(--bg-screen);
      font-family: var(--font-ui);
    }
```

- [ ] **Step 3: Open `index.html` in browser — verify warm cream background loads, no scroll bar**

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: html boilerplate, fonts, css tokens"
```

---

### Task 2: Five Layout Zones (Stub)

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Add the five structural zones to `<body>`**

```html
<body>
  <!-- Zone 1: Header bar -->
  <header id="header-bar"></header>

  <!-- Zone 2: Branding row -->
  <div id="branding-row">
    <span id="brand-mdi">MDI</span>
    <span id="brand-it-wrapper">
      <svg id="brand-it-circle" viewBox="0 0 40 40" xmlns="http://www.w3.org/2000/svg">
        <ellipse cx="20" cy="20" rx="19" ry="19" fill="none" stroke="#e5e5e6" stroke-width="1.5"/>
      </svg>
      <span id="brand-it">IT</span>
    </span>
  </div>

  <!-- Zone 3: Status label -->
  <div id="status-label">
    <span id="status-text">Round <strong id="round-number">1</strong></span>
  </div>

  <!-- Zone 4: Timer band -->
  <main id="timer-band">
    <div id="timer-display">3:00</div>
  </main>

  <!-- Zone 5: Controls bar -->
  <footer id="controls-bar">
    <div id="controls-rule"></div>
    <div id="controls-buttons">
      <button id="btn-play">PLAY</button>
      <button id="btn-reset">RESET</button>
      <button id="btn-mute">MUTE</button>
    </div>
  </footer>

  <!-- Audio elements -->
  <audio id="audio-tick1" src="tick-1.mp3" preload="auto"></audio>
  <audio id="audio-tick2" src="tick-2.mp3" preload="auto"></audio>
  <audio id="audio-fanfare" src="fanfare.mp3" preload="auto"></audio>
</body>
```

- [ ] **Step 2: Add stub zone CSS to `<style>` so zones are visible as colored blocks**

```css
    #header-bar {
      position: fixed;
      top: 0; left: 0; right: 0;
      height: 52px;
      background: linear-gradient(to bottom, #74c486 9.3%, #033f2c 255%);
      z-index: 10;
    }

    #timer-band {
      position: fixed;
      top: 52px; bottom: 80px;
      left: 0; right: 0;
      background: #fff;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
    }

    #controls-bar {
      position: fixed;
      bottom: 0; left: 0; right: 0;
      height: 80px;
      background: var(--bg-screen);
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
    }
```

- [ ] **Step 3: Open in browser — confirm green header at top, white timer band in middle, cream footer at bottom. No overflow.**

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: five layout zone stubs"
```

---

### Task 3: Header Bar + Branding Row

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Add branding row CSS**

```css
    #branding-row {
      position: fixed;
      top: 73px;
      left: 0; right: 0;
      height: 57px;
      display: flex;
      align-items: center;
      justify-content: space-between;
      padding: 0 32px;
      z-index: 9;
    }

    #brand-mdi {
      font-family: var(--font-display);
      font-size: var(--branding-size);
      color: var(--shadow);
      letter-spacing: var(--brand-tracking);
    }

    #brand-it-wrapper {
      position: relative;
      display: inline-flex;
      align-items: center;
      justify-content: center;
      width: 40px;
      height: 40px;
    }

    #brand-it-circle {
      position: absolute;
      top: 0; left: 0;
      width: 40px; height: 40px;
    }

    #brand-it {
      position: relative;
      font-family: var(--font-display);
      font-size: var(--branding-size);
      color: var(--shadow);
      letter-spacing: var(--brand-tracking);
    }
```

- [ ] **Step 2: Open in browser — confirm "MDI" appears left, "IT" in circle appears right, both in light gray, overlaid on the page (above white timer band).**

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: header bar and branding row"
```

---

### Task 4: Status Label

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Add status label CSS**

```css
    #status-label {
      position: fixed;
      top: 130px;
      left: 0; right: 0;
      text-align: center;
      font-family: var(--font-ui);
      font-size: var(--status-size);
      color: var(--itsit-green-dark);
      letter-spacing: var(--status-tracking);
      text-transform: uppercase;
      z-index: 8;
    }

    #status-label strong,
    #status-label #round-number {
      font-weight: 600;
    }
```

- [ ] **Step 2: Open in browser — confirm "Round 1" appears below branding row, centered, in dark green, uppercase.**

Note: "Round" should be regular weight, "1" should be SemiBold. Verify visually.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: status label"
```

---

### Task 5: Timer Display — All Four States

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Add timer display base CSS**

```css
    #timer-display {
      max-width: 784px;
      width: 100%;
      text-align: center;
      font-family: var(--font-display);
      font-size: var(--timer-size);
      letter-spacing: var(--timer-tracking);
      color: var(--itsit-blue);
      line-height: 1;
      user-select: none;
    }
```

- [ ] **Step 2: Add state-specific color overrides**

```css
    /* TIMEUP state */
    #timer-display.state-timeup {
      color: var(--timeup-pink);
    }

    /* SWITCH state */
    #timer-display.state-switch {
      color: var(--itsit-green);
    }

    /* GO state */
    #timer-display.state-go {
      color: var(--itsit-blue);
    }

    /* Crossfade animation (applied during ROUND :30–:00) */
    @keyframes crossfade {
      0%   { color: var(--itsit-blue); }
      50%  { color: var(--itsit-green); }
      100% { color: var(--itsit-blue); }
    }

    #timer-display.crossfade {
      animation-name: crossfade;
      animation-iteration-count: infinite;
      animation-timing-function: ease-in-out;
      /* animation-duration set by JS */
    }

    /* SWITCH opacity pulse — intentionally a second .state-switch block;
       CSS cascade merges it with the color rule above. */
    @keyframes pulse {
      from { opacity: 1; }
      to   { opacity: 0.15; }
    }

    #timer-display.state-switch {
      animation: pulse 1.5s ease-in-out infinite alternate;
    }

    /* GO state resets opacity to 1 explicitly (clearAllAnimations removes
       state-switch, but an explicit reset guards against future regressions) */
    #timer-display.state-go {
      opacity: 1;
    }
```

- [ ] **Step 3: Manually test all four states by temporarily adding classes to `#timer-display` in HTML**

Test sequence (add/remove class in HTML to verify visually):
- Default (no class): `3:00` in blue ✓
- `state-timeup`: `TIME UP` in pink ✓
- `state-switch`: `SWITCH` in green with pulse ✓
- `state-go`: `GO!` in blue ✓

Remove any manually added test classes before committing.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: timer display four state styles"
```

---

### Task 6: Controls Bar — Rule + Buttons

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Add controls bar detailed CSS**

```css
    #controls-rule {
      width: 690px;
      max-width: 100%;
      height: 4px;
      background: var(--itsit-green);
      margin-bottom: 8px;
    }

    #controls-buttons {
      display: flex;
      gap: 11px;
      align-items: center;
    }

    #controls-buttons button {
      width: 117.6px;
      height: 29.4px;
      font-family: var(--font-ui);
      font-size: var(--button-size);
      font-weight: 500;
      color: var(--itsit-green-dark);
      background: none;
      border: none;
      cursor: pointer;
      letter-spacing: 1px;
      text-transform: uppercase;
    }

    #controls-buttons button:hover {
      background: rgba(255,255,255,0.2);
      border-bottom: 0.4px solid var(--itsit-green-dark);
    }

    #controls-buttons button:active {
      background: rgba(255,255,255,0.6);
      border-bottom: 0.4px solid var(--itsit-green-dark);
    }

    #controls-buttons button:disabled {
      opacity: 0.35;
      cursor: not-allowed;
    }

    /* Active PLAY state — green underline to indicate "running" */
    #btn-play.is-playing {
      border-bottom: 0.4px solid var(--itsit-green);
    }
```

- [ ] **Step 2: Open in browser — confirm green rule above three buttons, buttons are same dark green text, hover shows underline.**

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: controls bar rule and buttons"
```

---

## Chunk 2: Timer Logic & State Machine

### Task 7: Global Variables + Utility Functions

**Files:**
- Modify: `index.html` — add `<script>` block before `</body>`

- [ ] **Step 1: Add the `<script>` block with configurable constants and DOM refs**

```js
<script>
  // ─── CONFIGURABLE DURATIONS ──────────────────────────────────────────────────
  const ROUND_DURATION = 180;  // Round countdown in seconds (default: 3:00)
  const BREAK_DURATION = 30;   // Inter-round break in seconds (default: :30)
  // ─────────────────────────────────────────────────────────────────────────────

  // Derived thresholds — do not edit these directly
  const CROSSFADE_START = 30;
  const TICK_START      = 20;
  const TICK_FAST       = 10;
  const TICK_FASTER     = 5;

  // DOM refs
  const timerDisplay  = document.getElementById('timer-display');
  const statusText    = document.getElementById('status-text');
  const roundNumber   = document.getElementById('round-number');
  const btnPlay       = document.getElementById('btn-play');
  const btnReset      = document.getElementById('btn-reset');
  const btnMute       = document.getElementById('btn-mute');
  const audioTick1    = document.getElementById('audio-tick1');
  const audioTick2    = document.getElementById('audio-tick2');
  const audioFanfare  = document.getElementById('audio-fanfare');

  // ─── App State ───────────────────────────────────────────────────────────────
  let state          = 'round';   // 'round' | 'timeup' | 'switch' | 'go'
  let timeLeft       = ROUND_DURATION;
  let breakLeft      = BREAK_DURATION;
  let roundNum       = 1;
  let isPlaying      = false;
  let isMuted        = false;
  let mainInterval   = null;
  let tickInterval   = null;
  let switchInterval = null;   // module-level so RESET can clear it
  let stateTimeout   = null;   // module-level so RESET can clear TIMEUP/GO timeouts
  let tickToggle     = false;

  // ─── Helpers ─────────────────────────────────────────────────────────────────
  function formatTime(secs) {
    const m = Math.floor(secs / 60);
    const s = secs % 60;
    return `${m}:${String(s).padStart(2, '0')}`;
  }

  function playAudio(el) {
    if (isMuted) return;
    el.currentTime = 0;
    el.play().catch(() => {});
  }

  function stopAllAudio() {
    [audioTick1, audioTick2, audioFanfare].forEach(a => {
      a.pause();
      a.currentTime = 0;
    });
  }
</script>
```

- [ ] **Step 2: Open browser console — confirm no errors on load.**

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: global constants, dom refs, helper functions"
```

---

### Task 8: setState — Central State Machine Function

**Files:**
- Modify: `index.html` — append to `<script>` block

- [ ] **Step 1: Add `clearAllAnimations`, `clearTickAudio`, and `setState` to the script**

```js
  function clearTickAudio() {
    if (tickInterval) { clearInterval(tickInterval); tickInterval = null; }
  }

  function clearAllAnimations() {
    timerDisplay.classList.remove('crossfade', 'state-timeup', 'state-switch', 'state-go');
    timerDisplay.style.animationDuration = '';
  }

  function setCrossfade(duration) {
    timerDisplay.classList.remove('crossfade');
    void timerDisplay.offsetWidth; // force reflow to restart animation
    timerDisplay.style.animationDuration = duration + 'ms';
    timerDisplay.classList.add('crossfade');
  }

  function setState(newState) {
    state = newState;
    clearAllAnimations();
    clearTickAudio();

    if (newState === 'round') {
      timerDisplay.textContent = formatTime(timeLeft);
      statusText.innerHTML = `Round <strong>${roundNum}</strong>`;
      btnPlay.disabled = false;

    } else if (newState === 'timeup') {
      timerDisplay.textContent = 'TIME UP';
      timerDisplay.classList.add('state-timeup');
      statusText.innerHTML = `Round <strong>${roundNum}</strong>`;
      btnPlay.disabled = true;
      playAudio(audioFanfare);

      stateTimeout = setTimeout(() => setState('switch'), 7000);

    } else if (newState === 'switch') {
      clearInterval(switchInterval); switchInterval = null; // defensive, in case of re-entry
      breakLeft = BREAK_DURATION;
      timerDisplay.textContent = 'SWITCH';
      timerDisplay.classList.add('state-switch');
      updateSwitchLabel();
      btnPlay.disabled = true;

      // stored at module level so RESET can clear it
      switchInterval = setInterval(() => {
        breakLeft--;
        if (breakLeft <= 0) {
          clearInterval(switchInterval);
          switchInterval = null;
          setState('go');
        } else {
          updateSwitchLabel();
        }
      }, 1000);

    } else if (newState === 'go') {
      timerDisplay.textContent = 'GO!';
      timerDisplay.classList.add('state-go');
      statusText.innerHTML = `Round <strong>${roundNum}</strong>`;
      btnPlay.disabled = true;

      stateTimeout = setTimeout(() => {
        roundNum++;
        timeLeft = ROUND_DURATION;
        isPlaying = true;
        setState('round');
        startRoundInterval();
        updatePlayButton();
      }, 3000);
    }
  }

  function updateSwitchLabel() {
    const formatted = `:${String(breakLeft).padStart(2, '0')}`;
    statusText.innerHTML = `NEXT ROUND BEGINS IN <strong>${formatted}</strong>`;
  }
```

- [ ] **Step 2: Open browser console — no errors. Manually call `setState('timeup')` in console, confirm display changes.**

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: setState state machine"
```

---

### Task 9: Round Countdown Logic

**Files:**
- Modify: `index.html` — append to `<script>` block

- [ ] **Step 1: Add `startRoundInterval` and `tick` functions**

```js
  function startRoundInterval() {
    if (mainInterval) clearInterval(mainInterval);
    mainInterval = setInterval(tick, 1000);
  }

  function tick() {
    timeLeft--;

    // Update display
    timerDisplay.textContent = formatTime(timeLeft);

    // Crossfade thresholds
    if (timeLeft === CROSSFADE_START) {
      setCrossfade(3000);
    } else if (timeLeft === TICK_START) {
      setCrossfade(2000);
      setTickInterval(1000);
    } else if (timeLeft === TICK_FAST) {
      setCrossfade(1000);
      setTickInterval(500);
    } else if (timeLeft === TICK_FASTER) {
      setTickInterval(250);
    }

    if (timeLeft <= 0) {
      clearInterval(mainInterval);
      mainInterval = null;
      clearTickAudio();
      isPlaying = false;
      updatePlayButton();
      setState('timeup');
    }
  }

  // Clears any existing tick interval and starts a new one at the given ms rate.
  // Called at :20, :10, and :05 thresholds with progressively shorter intervals.
  function setTickInterval(interval) {
    clearTickAudio();
    tickInterval = setInterval(() => {
      tickToggle = !tickToggle;
      playAudio(tickToggle ? audioTick1 : audioTick2);
    }, interval);
  }
```

- [ ] **Step 2: Open browser console, call `setState('round')`, then manually call `startRoundInterval()`. Watch the timer count down from 3:00 in the display.**

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: round countdown tick and audio scheduling"
```

---

### Task 10: PLAY / PAUSE + RESET + MUTE Buttons

**Files:**
- Modify: `index.html` — append to `<script>` block

- [ ] **Step 1: Add button handler functions**

```js
  function updatePlayButton() {
    btnPlay.textContent = isPlaying ? 'PAUSE' : 'PLAY';
    if (isPlaying) {
      btnPlay.classList.add('is-playing');
    } else {
      btnPlay.classList.remove('is-playing');
    }
  }

  function handlePlay() {
    if (state !== 'round') return;
    if (isPlaying) {
      // Pause
      clearInterval(mainInterval);
      mainInterval = null;
      clearTickAudio();
      isPlaying = false;
    } else {
      // Play
      isPlaying = true;
      startRoundInterval();
    }
    updatePlayButton();
  }

  function handleReset() {
    // Clear all timers — including SWITCH interval and TIMEUP/GO timeouts
    clearInterval(mainInterval);  mainInterval = null;
    clearInterval(switchInterval); switchInterval = null;
    clearTimeout(stateTimeout);   stateTimeout = null;
    clearTickAudio();
    stopAllAudio();
    timeLeft  = ROUND_DURATION;
    breakLeft = BREAK_DURATION;
    roundNum  = 1;
    isPlaying = false;
    isMuted   = false;
    btnMute.textContent = 'MUTE';
    btnMute.setAttribute('aria-label', 'Mute audio');
    updatePlayButton();
    setState('round');
  }

  function handleMute() {
    isMuted = !isMuted;
    btnMute.textContent = isMuted ? 'UNMUTE' : 'MUTE';
    if (isMuted) stopAllAudio();
  }

  // ─── Event Listeners ─────────────────────────────────────────────────────────
  btnPlay.addEventListener('click', handlePlay);
  btnReset.addEventListener('click', handleReset);
  btnMute.addEventListener('click', handleMute);

  // ─── Visibility change: pause on tab switch ───────────────────────────────────
  document.addEventListener('visibilitychange', () => {
    if (document.hidden && isPlaying && state === 'round') {
      handlePlay(); // pause
    }
  });

  // ─── Initial render ───────────────────────────────────────────────────────────
  setState('round');
  updatePlayButton();
```

- [ ] **Step 2: Open browser and run a full test cycle:**
  - Click PLAY → timer counts down ✓
  - Click PAUSE → timer stops ✓
  - Click PLAY again → resumes ✓
  - Click RESET → returns to 3:00, Round 1 ✓
  - Click MUTE → button reads UNMUTE ✓
  - Click UNMUTE → button reads MUTE ✓

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: play/pause/reset/mute button handlers and init"
```

---

### Task 11: End-to-End State Cycle Test

**Files:**
- Modify: `index.html` — temporarily set `ROUND_DURATION = 10` to speed up testing, then restore

- [ ] **Step 1: Set `ROUND_DURATION = 10` at top of script**

- [ ] **Step 2: Click PLAY and observe full cycle:**
  - Timer counts 0:10 → 0:00 ✓
  - "TIME UP" appears in pink, fanfare plays ✓
  - After 7 seconds: "SWITCH" appears in green with pulse ✓
  - Status label reads "NEXT ROUND BEGINS IN :10" counting down ✓
  - After :00: "GO!" appears in blue ✓
  - After 3 seconds: round 2 starts at 0:10 ✓
  - Status label reads "Round 2" ✓

- [ ] **Step 3: Restore `ROUND_DURATION = 180` and `BREAK_DURATION = 30`**

- [ ] **Step 4: Test MUTE during tick + fanfare:**
  - Set `ROUND_DURATION = 10`, start timer, click MUTE before :05
  - Verify no tick sounds play ✓
  - Verify fanfare does NOT play at TIMEUP ✓
  - Click UNMUTE, RESET, test again to confirm audio returns ✓

- [ ] **Step 5: Restore `ROUND_DURATION = 180`**

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "feat: full state cycle verified"
```

---

## Chunk 3: Polish & Accessibility

### Task 12: ARIA Labels + Keyboard Accessibility

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Add ARIA attributes to buttons and live region**

Update HTML button elements:

```html
<button id="btn-play" aria-label="Play timer">PLAY</button>
<button id="btn-reset" aria-label="Reset timer">RESET</button>
<button id="btn-mute" aria-label="Mute audio">MUTE</button>
```

Update status label to be a live region:

```html
<div id="status-label" aria-live="polite" aria-atomic="true">
  ...
```

Update `handleMute()` in JS to update aria-label:

```js
  function handleMute() {
    isMuted = !isMuted;
    btnMute.textContent = isMuted ? 'UNMUTE' : 'MUTE';
    btnMute.setAttribute('aria-label', isMuted ? 'Unmute audio' : 'Mute audio');
    if (isMuted) stopAllAudio();
  }
```

Update `updatePlayButton()` in JS (full replacement — adds the aria-label line):

```js
  function updatePlayButton() {
    btnPlay.textContent = isPlaying ? 'PAUSE' : 'PLAY';
    btnPlay.setAttribute('aria-label', isPlaying ? 'Pause timer' : 'Play timer');
    if (isPlaying) {
      btnPlay.classList.add('is-playing');
    } else {
      btnPlay.classList.remove('is-playing');
    }
  }
```

- [ ] **Step 2: Tab through buttons in browser — confirm focus is visible (browser default outline).**

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: aria labels and keyboard accessibility"
```

---

### Task 13: Full-Screen Layout Verification + Final QA

**Files:**
- Modify: `index.html` (fix any layout issues found)

- [ ] **Step 1: Open in Chrome, resize to several viewport sizes (1920×1080, 1440×900, 1280×800)**

Check each size:
- Header bar stays pinned to top ✓
- Controls bar stays pinned to bottom ✓
- Timer digits scale with `clamp()` ✓
- No scrollbar appears ✓
- Status label stays readable ✓

- [ ] **Step 2: Open in Safari — confirm fonts load (Holtwood One SC, Instrument Sans)**

- [ ] **Step 3: Open in Firefox — confirm same layout**

- [ ] **Step 4: Open on iOS Safari (iPhone or iPad, or Simulator) — confirm layout, fonts, and controls bar not hidden by safe-area insets**

- [ ] **Step 5: Run full timer cycle at short duration (`ROUND_DURATION = 10`) in each browser**

Restore to 180 after testing.

- [ ] **Step 6: Verify against Figma**

Open Figma: https://www.figma.com/design/TbBFO6DK8XISy7ctRvpBUg/What-s-It?node-id=3638-3346

Check:
- Header gradient matches ✓
- Branding row position and color ✓
- Timer digit color and size ✓
- Controls rule + button styling ✓
- All four display states (TIME UP pink, SWITCH green, GO! blue) ✓

- [ ] **Step 7: Commit any layout fixes found during QA (skip this step if no changes were needed)**

```bash
git add index.html
git commit -m "fix: cross-browser layout adjustments"
```

---

### Task 14: Final Verification Before Deploy

- [ ] **Step 1: Set `ROUND_DURATION = 180`, `BREAK_DURATION = 30` (production values)**

- [ ] **Step 2: Run a full cycle with shortened durations one last time for confidence:**

Set `ROUND_DURATION = 15, BREAK_DURATION = 10`, click PLAY, observe complete cycle.

- [ ] **Step 3: Restore production values, verify `index.html` has correct constants**

- [ ] **Step 4: Confirm all required files are present in the directory:**

```bash
ls -la /Users/colinsebestyen/Desktop/MDI/06-02-26-Fun_Whats_It/MDI-timer/
# Expected: index.html, tick-1.mp3, tick-2.mp3, fanfare.mp3
```

Also open the browser Network tab, reload `index.html`, and confirm no 404 errors for the three audio files.

- [ ] **Step 5: Final commit**

```bash
git add index.html
git commit -m "chore: production values verified, ready for deploy"
```

---

## Deploy (Manual — User Action)

Per `PROJECT-PLAN.md` — the user deploys manually via SFTP:

```
Host: 72.167.57.128
Path: /home/sm2tdtbtphn3/public_html/MDI/whatsit-timer/
Files: index.html, tick-1.mp3, tick-2.mp3, fanfare.mp3
Live URL: http://www.movecraft.com/MDI/whatsit-timer
```

**Do NOT attempt any network operations.** The user handles SFTP.

---

## Notes

- Every task builds on the previous one — complete in order.
- If a browser verification step fails, fix before committing.
- The `ROUND_DURATION` hack (setting to 10 for testing) should always be restored before the final commit.
- Audio autoplay policy on Chrome requires a user gesture before audio plays — the PLAY button click satisfies this. No special workaround needed.
- The `void timerDisplay.offsetWidth` reflow trick in `setCrossfade` is intentional — it forces CSS to restart the animation when switching durations.
