# MDI Timer — "What's It?" Game Timer Project Plan

**Project:** What's It? Round Timer  
**Date:** 2026-05-22  
**Live URL:** http://www.movecraft.com/MDI/whatsit-timer  
**Delivery:** Single `index.html` (self-contained, no build step)  
**Design Spec:** `PROJECT-PLAN.md` (this document)  
**Figma:** https://www.figma.com/design/TbBFO6DK8XISy7ctRvpBUg/What-s-It?node-id=3638-3346

---

## Context

This is a round timer for the **MDI "What's It?"** party game. It counts down between rounds, signals game state changes (SWITCH teams, TIME UP, GO!), and supports audio muting. The timer is operator-controlled — a game host runs it on a screen visible to players.

---

## UX & State Machine

### Full Round Cycle (loops indefinitely, round number increments each cycle)

```
[ROUND STATE]  3:00 → 0:00 countdown
    │
    ├─ :30 remaining → blue↔green crossfade begins on timer digits
    ├─ :20 remaining → tick audio starts (every 1000ms)
    ├─ :10 remaining → tick audio speeds up (every 500ms)
    ├─ :05 remaining → tick audio speeds up (every 250ms)
    │
    ↓ 0:00
[TIMEUP STATE]  "TIME UP" displayed (pink), fanfare audio fires
    │  holds for 7 seconds
    ↓
[SWITCH STATE]  "SWITCH" displayed (green), fades in/out in opacity
    │  status label: "NEXT ROUND BEGINS IN :30" → counts down to :00
    ↓
[GO STATE]  "GO!" displayed (blue), holds for 3 seconds
    │  round number increments
    ↓
[ROUND STATE]  auto-starts 3:00 countdown → loop
```

---

### State Details

#### ROUND — Countdown (3:00 → 0:00)
- Status label: `"Round 1"` → `"Round 2"` etc. The round number is **bold** (Instrument Sans SemiBold), the word "Round" is regular weight.
- Timer displays `M:SS` in blue (`#4e8dbc`), Holtwood One SC, 196px.
- At **:30 remaining**: smooth blue↔green color crossfade begins on the timer digits.
  - :30–:20 → crossfade cycle period: **3000ms** (slow pulse)
  - :20–:10 → crossfade cycle period: **2000ms** (medium pulse)
  - :10–:00 → crossfade cycle period: **1000ms** (fast pulse)
  - Implementation: CSS `animation` on `color`, keyframes `#4e8dbc → #74c486 → #4e8dbc`, `animation-duration` updated in JS at each threshold.
- At **:20 remaining**: `tick.mp3` begins playing on a repeating interval.
  - :20–:10 → tick interval: **1000ms**
  - :10–:05 → tick interval: **500ms**
  - :05–:00 → tick interval: **250ms**
  - Tick playback respects the MUTE toggle.

#### TIMEUP — 7 seconds
- Timer display swaps to `"TIME UP"` in pink (Figma timeup color).
- Crossfade animation stops. Color locks to pink.
- `fanfare.mp3` plays immediately on entry (respects MUTE).
- Status label retains current round text (e.g. `"Round 1"`).
- No user interaction needed; auto-advances after 7 seconds.

#### SWITCH — 30-second inter-round break
- Timer display shows `"SWITCH"` in green (`#74c486`).
- Opacity pulses: fades in and out continuously at a slow, calm pace — `animation: pulse 1.5s ease-in-out infinite alternate` (fades between full opacity and ~0.15).
- Status label: `"NEXT ROUND BEGINS IN "` (regular weight) + `":30"` counting down to `":00"` (bold/SemiBold). Updates every second.
- Round number has **not** incremented yet — it updates only when the new 3:00 countdown begins.
- No audio during SWITCH state.
- Auto-advances when the :30 countdown reaches :00.

#### GO — 3 seconds
- Timer display shows `"GO!"` in blue (`#4e8dbc`).
- Opacity animation stops; GO! is fully opaque.
- No audio.
- After 3 seconds: auto-resets timer to 3:00, **increments the round number**, and immediately starts the ROUND countdown. The round number updates at the moment the new countdown begins, not before.

---

### Audio Files

| File | Trigger | Behavior | Respects MUTE |
|---|---|---|---|
| `tick-1.mp3` | :20 remaining in round | Alternates with tick-2 | Yes |
| `tick-2.mp3` | :20 remaining in round | Alternates with tick-1 | Yes |
| `fanfare.mp3` | On entry to TIMEUP state | Plays once, 5s duration | Yes |

- All files live in the same directory as `index.html`.
- **Tick alternation** — maintain a boolean or index that flips each tick: odd ticks play `tick-1.mp3`, even ticks play `tick-2.mp3` (or vice versa). Use two `<audio>` elements, one per file, and call `.currentTime = 0; .play()` on the active one each interval.
- **Fanfare** — plays once on TIMEUP entry, runs for its full 5-second duration. TIMEUP state holds for 7 seconds total, so fanfare completes before the state transitions to SWITCH.
- MUTE sets a global `muted` flag; all audio calls check this flag before firing.

---

### Controls Behavior

#### MUTE / UNMUTE
- Toggles global audio volume on and off.
- Button label switches between `"MUTE"` and `"UNMUTE"` to reflect current state.
- Active in all states.

#### PLAY / PAUSE
- Toggles the countdown on and off.
- Button label switches between `"PLAY"` and `"PAUSE"` to reflect current state.
- Only active during the ROUND (countdown) state — disabled/inert during TIMEUP, SWITCH, and GO states (those phases are time-driven and not operator-paused).

#### RESET
- Returns the entire app to its initial state: round countdown at `ROUND_DURATION`, Round 1, paused.
- Clears all animations, cancels all audio, resets all intervals.
- Active in all states.

| Button | ROUND state | TIMEUP state | SWITCH state | GO state |
|---|---|---|---|---|
| MUTE/UNMUTE | Active | Active | Active | Active |
| PLAY/PAUSE | Active (toggles countdown) | Disabled | Disabled | Disabled |
| RESET | Active | Active | Active | Active |

---

## Goals & Requirements

### Primary Goal
Build a single-page countdown timer that guides players through rounds of the "What's It?" game. The design is clean, large-format, and readable from a distance.

### Functional Requirements
- 3-minute countdown timer, hard-coded (not user-configurable)
- Loops indefinitely; round number increments each cycle
- Four display states: `timer` → `timeup` → `switch` → `go!` → back to `timer`
- Animated blue↔green color crossfade on timer digits from :30–:00
- Tick audio (`tick.mp3`) from :20–:00, accelerating in three stages
- Fanfare audio (`fanfare.mp3`) on time-up
- 30-second inter-round break with live countdown in status label
- Three control buttons: **PLAY** (start/pause), **RESET**, **MUTE**
- RESET returns to 3:00, Round 1, paused

### Timer Display States

| State | Display Text | Color | Duration |
|---|---|---|---|
| `timer` | `3:00` counting down | Blue `#4e8dbc` (pulses to green from :30) | 3 minutes |
| `timeup` | `TIME UP` | Pink (Figma timeup color) | 7 seconds |
| `switch` | `SWITCH` | Green `#74c486`, opacity pulse | 30 seconds |
| `go` | `GO!` | Blue `#4e8dbc` | 3 seconds |

### Non-Functional Requirements
- Single `index.html` file — no separate CSS or JS files
- No build pipeline; edit and open directly in browser
- Deployed via SFTP to `72.167.57.128 → /home/sm2tdtbtphn3/public_html/MDI/whatsit-timer/`
- Must work on Chrome, Safari, Firefox; desktop priority
- **Must support full-screen** — the layout should fill 100vw × 100vh at any display size with no scrolling and no overflow

---

## Tech Stack & Architecture

| Layer | Choice | Notes |
|---|---|---|
| Structure | HTML5 | Single file, semantic markup |
| Styling | CSS3 (inline `<style>`) | CSS custom properties for all tokens |
| Logic | Vanilla JS (inline `<script>`) | No frameworks, no bundlers |
| Fonts | Google Fonts (`@import`) | Holtwood One SC (timer/display), Instrument Sans (UI) |
| Audio | HTML5 `<audio>` | Triggered on state transitions; mutable |
| Hosting | Shared hosting via SFTP | Manual deploy by user |

### File Structure
```
/whatsit-timer/
└── index.html        ← entire app (HTML + CSS + JS inline)
```
> Note: No image assets needed based on Figma design — the layout uses pure CSS/text.

---

## Design Tokens

**Source of truth: Figma** (values below supersede the original .md spec where they differ)

```
/* Colors */
--bg-screen:        #fcf7f2   /* page background — warm cream */
--itsit-blue:       #4e8dbc   /* timer digits, GO! state */
--itsit-green:      #74c486   /* header gradient top, rule line, SWITCH state */
--itsit-green-dark: #033f2c   /* header gradient bottom, button text, status text */
--shadow:           #e5e5e6   /* MDI/IT branding text color */
--timeup-pink:      #f178b6   /* TIME UP text — Fuschia/80, confirmed from Figma */

/* Typography */
--font-display:  'Holtwood One SC', serif    /* timer digits, state words, MDI/IT branding */
--font-ui:       'Instrument Sans', sans-serif  /* buttons, status label */

/* Font sizes */
--timer-size:    196px   /* main countdown digits */
--status-size:   16px    /* "NEXT ROUND BEGINS IN :30" */
--button-size:   14.7px  /* PLAY / RESET / MUTE */
--branding-size: 20.668px /* MDI / IT text */

/* Spacing / Layout */
--timer-tracking: 9.8px   /* letter-spacing on timer digits */
--status-tracking: 2px    /* letter-spacing on status label */
--brand-tracking:  3.875px /* letter-spacing on MDI/IT */
```

---

## Layout Structure

The page has **5 vertical zones** (top to bottom):

```
┌─────────────────────────────────────────┐
│  HEADER BAR  (green gradient, 52px)     │
├─────────────────────────────────────────┤
│  BRANDING ROW  (MDI left · IT right)    │  ~57px tall, 73px from top
├─────────────────────────────────────────┤
│                                         │
│  STATUS LABEL  ("NEXT ROUND BEGINS IN") │  centered
│                                         │
│  ████████████████████████████████       │
│        TIMER DISPLAY  (3:00)            │  white band, vertically centered
│  ████████████████████████████████       │
│                                         │
├─────────────────────────────────────────┤
│  CONTROLS  (rule + PLAY/RESET/MUTE)     │  80px, pinned to bottom
└─────────────────────────────────────────┘
```

**Key layout notes:**
- Header bar: full-width, `background: linear-gradient(to bottom, #74c486, #033f2c)`
- Timer band: white background, full-width, vertically centered in viewport
- Timer digits: `max-width: 784px`, centered, `font-size: 196px`, `letter-spacing: 9.8px`
- Controls bar: `position: fixed; bottom: 44px` — always visible
- Green rule: `height: ~4px`, full width of control container (690px), above buttons
- Buttons: 3 × 117.6px wide, spaced 11px apart, dark green text on transparent bg, bottom border only (underline style)
- Active/selected button (PLAY when active): bottom border highlighted in green

---

## Component Specs

### Header Bar (`#3638:3365`)
- Full-width rectangle, `height: 52px`, pinned to top
- Background: `linear-gradient(to bottom, #74c486 9.3%, #033f2c 255%)`

### Branding Row (`#3638:3359`)
- Left: "MDI" — Holtwood One SC, 20.668px, `#e5e5e6`, `letter-spacing: 3.875px`
- Right: "IT" inside a circle badge — same font/size/color, circle outlined with SVG ellipse

### Status Label (`#3638:3357 / #3638:3358`)
- Text: `"NEXT ROUND BEGINS IN "` (Instrument Sans Regular) + `":30"` (Instrument Sans SemiBold)
- Size: 16px, color: `#033f2c`, `letter-spacing: 2px`, uppercase, centered

### Timer Display (`#3638:3356`)
- Component with 4 swap states: `timer`, `timeup`, `switch`, `go!`
- Container: 784px wide × 157px tall, centered in white band
- Font: Holtwood One SC Regular, 196px, `letter-spacing: 9.8px`, centered

### Controls Bar (`#3638:3348 / #3638:3349`)
- Green rule line: `height: 4px`, `background: #74c486`, full container width
- Button row: flex, centered, gap 11px
- Each button: 117.6px × 29.4px, Instrument Sans Medium 14.7px, color `#033f2c`

**Button states (from Figma component `#3638:3367`):**

| State | Background | Border |
|---|---|---|
| Default | none | none |
| Hover (`:hover`) | `rgba(255,255,255,0.2)` | `border-bottom: 0.4px solid #033f2c` |
| Press (`:active`) | `rgba(255,255,255,0.6)` | `border-bottom: 0.4px solid #033f2c` |

All three states share the same text color (`#033f2c`), font (Instrument Sans Medium), and dimensions. Implement with CSS `:hover` and `:active` pseudo-classes.

---

## Phases & Milestones

### Phase 1 — Foundation
Set up the HTML skeleton, fonts, CSS tokens, and layout zones as empty colored blocks.

- `index.html` boilerplate (DOCTYPE, meta viewport, title)
- Google Fonts `@import` for Holtwood One SC + Instrument Sans
- CSS custom properties for all design tokens
- Five layout zones stubbed out with correct colors/heights

**Milestone:** Page loads with correct font and color zones, no content yet ✓

---

### Phase 2 — Static UI
Build all visual elements as static HTML/CSS. No JavaScript. All 4 timer states visible during dev by swapping a class.

- Header bar (green gradient)
- Branding row: "MDI" left, "IT" circle right
- Status label: "NEXT ROUND BEGINS IN :30"
- Timer band (white background, full width)
- Timer display — default state `timer` showing "3:00" in blue
- Timer display — `timeup` state ("TIME UP", pink)
- Timer display — `switch` state ("SWITCH", green)
- Timer display — `go!` state ("GO!", blue)
- Controls bar: green rule + PLAY / RESET / MUTE buttons
- Button active/hover states

**Milestone:** All layout zones and all 4 timer states render pixel-accurately ✓

---

### Phase 3 — Timer Logic & State Machine
Wire up JavaScript for a functioning timer with state transitions.

- Countdown logic from 3:00 to 0:00 (using `setInterval`, 1-second tick)
- PLAY button: starts/pauses the timer; label toggles PLAY ↔ PAUSE
- RESET button: returns to 3:00, resets to `timer` state
- On reaching 0:00: transition through states (`timeup` → `switch` → `go!`) on a defined sequence/timing
- Status label updates to match current state
- MUTE button: toggles audio flag; icon/label reflects state
- Audio: `<audio>` elements triggered on state transitions (files TBD)
- Edge cases: prevent double-start, `visibilitychange` pause on tab switch

**Milestone:** Full timer cycle runs correctly end-to-end ✓

---

### Phase 4 — Polish & QA

- Cross-browser test: Chrome, Safari, Firefox, iOS Safari
- Verify font loading (Holtwood One SC, Instrument Sans)
- Accessibility: ARIA labels on buttons, keyboard operability
- Final pixel comparison against Figma screenshot
- Audio file integration and mute behavior

**Milestone:** QA sign-off, matches Figma ✓

---

### Phase 5 — Deploy

- SFTP all files to `72.167.57.128 → /home/sm2tdtbtphn3/public_html/MDI/whatsit-timer/`
- Verify at http://www.movecraft.com/MDI/whatsit-timer
- Smoke test: run a full timer cycle on the live URL

**Milestone:** Live and verified ✓

---

## Task List

| # | Task | Phase | Status | Notes |
|---|---|---|---|---|
| 1 | `index.html` boilerplate | 1 | ⬜ Pending | DOCTYPE, viewport, title |
| 2 | Google Fonts `@import` | 1 | ⬜ Pending | Holtwood One SC + Instrument Sans |
| 3 | CSS custom properties | 1 | ⬜ Pending | All tokens from Design Tokens section |
| 4 | Five layout zones (stub) | 1 | ⬜ Pending | Colored blocks, correct heights |
| 5 | Header bar | 2 | ⬜ Pending | Green gradient, 52px |
| 6 | Branding row | 2 | ⬜ Pending | MDI left, IT circle right |
| 7 | Status label | 2 | ⬜ Pending | Mixed-weight text, tracking |
| 8 | Timer band | 2 | ⬜ Pending | White bg, full width, centered |
| 9 | Timer display — `timer` state | 2 | ⬜ Pending | "3:00", blue |
| 10 | Timer display — `timeup` state | 2 | ⬜ Pending | "TIME UP", pink |
| 11 | Timer display — `switch` state | 2 | ⬜ Pending | "SWITCH", green |
| 12 | Timer display — `go!` state | 2 | ⬜ Pending | "GO!", blue |
| 13 | Controls bar | 2 | ⬜ Pending | Rule + 3 buttons |
| 14 | Button states (active/hover) | 2 | ⬜ Pending | Underline highlight |
| 15 | Countdown logic | 3 | ⬜ Pending | setInterval, 1s tick, M:SS display |
| 16 | PLAY/PAUSE button handler | 3 | ⬜ Pending | Toggle label, start/stop interval |
| 17 | RESET button handler | 3 | ⬜ Pending | Back to 3:00, Round 1, paused |
| 18 | Blue↔green crossfade animation | 3 | ⬜ Pending | CSS anim, JS updates duration at :30/:20/:10 |
| 19 | Tick audio scheduling | 3 | ⬜ Pending | 3 intervals: 1000/500/250ms from :20 |
| 20 | TIMEUP state + fanfare audio | 3 | ⬜ Pending | 7s hold, fanfare.mp3 on entry |
| 21 | SWITCH state + opacity pulse | 3 | ⬜ Pending | CSS opacity anim, 30s countdown in label |
| 22 | GO! state + auto-restart | 3 | ⬜ Pending | 3s hold, then 3:00 auto-starts |
| 23 | Round counter increment | 3 | ⬜ Pending | Loops indefinitely |
| 24 | Status label — round & inter-round | 3 | ⬜ Pending | "Round N" and "NEXT ROUND BEGINS IN :SS" |
| 25 | MUTE toggle | 3 | ⬜ Pending | Global flag, affects tick + fanfare |
| 26 | Edge cases | 3 | ⬜ Pending | Double-start, visibilitychange pause |
| 27 | Cross-browser testing | 4 | ⬜ Pending | Chrome, Safari, Firefox, iOS |
| 28 | Audio integration test | 4 | ⬜ Pending | Both files, mute behavior |
| 29 | Animation timing review | 4 | ⬜ Pending | Crossfade feel, opacity pulse feel |
| 30 | Accessibility pass | 4 | ⬜ Pending | ARIA, keyboard |
| 31 | Figma pixel comparison | 4 | ⬜ Pending | Screenshot vs. spec |
| 32 | SFTP deploy | 5 | ⬜ Pending | index.html + audio files |
| 33 | Live smoke test | 5 | ⬜ Pending | Full cycle on production URL |

---

## Notes for Claude Code

- **Single file only** — all HTML, CSS, and JS goes in `index.html`. Do not create separate `.css` or `.js` files.
- **No build step** — no npm, no Webpack, no Vite, no framework. Pure HTML/CSS/JS.
- **Font loading** — use Google Fonts `@import` inside `<style>`, not a `<link>` tag in `<head>`.
- **Design tokens** — always use the CSS custom properties defined above. The Figma values supersede the original `.md` spec.
- **Timer display** — implement as a single element whose content and color class swap based on state. Four states: `timer`, `timeup`, `switch`, `go`.
- **No image assets** — the design is pure CSS/text. Do not reference or create any `.png` files.
- **Deploy is manual** — do not attempt SFTP or any network operations. The user handles deployment.
- **Figma reference** — use https://www.figma.com/design/TbBFO6DK8XISy7ctRvpBUg/What-s-It?node-id=3638-3346 as the visual source of truth.
- **Full-screen layout** — the Figma is designed at 1222×755. All zones must scale to fill any viewport:
  - `body` and wrapper: `width: 100vw; height: 100vh; overflow: hidden`
  - Header bar: `position: fixed; top: 0; left: 0; right: 0` — fixed height (52px)
  - Controls bar: `position: fixed; bottom: 0; left: 0; right: 0` — fixed height
  - Branding (MDI / IT): absolutely positioned, pinned to left and right edges of viewport
  - Timer band: fills remaining vertical space between header and controls; timer content is vertically centered
  - Timer digits: use `clamp()` or `vw`-based sizing so the 196px spec scales up on large displays — e.g. `font-size: clamp(100px, 16vw, 260px)`
  - Status label and all centered elements: `width: 100%; text-align: center`

### Global Variables (top of `<script>`, user-adjustable)

Place these as the very first declarations in the `<script>` block, clearly commented, so the operator can tweak durations without reading the rest of the code:

```js
// ─── CONFIGURABLE DURATIONS ──────────────────────────────────────────────────
const ROUND_DURATION    = 180;  // Round countdown in seconds (default: 3:00)
const BREAK_DURATION    = 30;   // Inter-round break in seconds (default: :30)
// ─────────────────────────────────────────────────────────────────────────────
```

All timer logic must reference these constants — never hard-code `180` or `30` elsewhere in the script. The crossfade and tick thresholds below are expressed as offsets from `ROUND_DURATION` and should scale accordingly:

```js
// Derived thresholds — do not edit these directly
const CROSSFADE_START   = 30;   // seconds remaining when crossfade begins
const TICK_START        = 20;   // seconds remaining when tick audio begins
const TICK_FAST         = 10;   // seconds remaining when tick interval halves
const TICK_FASTER       = 5;    // seconds remaining when tick interval quarters
```
