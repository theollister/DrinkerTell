# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

**Drink & Tell** is a single-file, self-contained browser-based drinking party game. Players take turns answering taboo/personality questions in multiple-choice format; other players guess the active player's answer, and wrong guessers earn a drink.

The entire application lives in one file: `drink-and-tell.html`.

---

## Repository Structure

```
DrinkerTell/
├── drink-and-tell.html       # Complete application (HTML + CSS + JS, ~567 lines)
├── .github/workflows/
│   └── deploy.yml            # GitHub Pages deployment on push to main
└── CLAUDE.md                 # This file
```

There is no build tooling, no package manager, no backend, and no tests.

---

## Technology Stack

| Layer       | Technology                          |
|-------------|-------------------------------------|
| Markup      | HTML5                               |
| Styling     | CSS3 (embedded `<style>`)           |
| Logic       | Vanilla ES6 JavaScript (IIFE + global scope) |
| Font        | Cabinet Grotesk via Fontshare CDN   |
| State       | In-memory JS object (no persistence)|
| Storage     | None (localStorage not used)        |

No frameworks, no build step, no npm, no backend. Open the file in a browser and it works.

---

## Application Architecture

### File Layout (drink-and-tell.html)

| Lines     | Section                                      |
|-----------|----------------------------------------------|
| 1–158     | `<head>`: meta tags + all embedded CSS       |
| 160–222   | `<body>`: four screen divs + confetti layer  |
| 223–228   | Theme toggle IIFE                            |
| 230–300   | `CATEGORIES` array + `QUESTIONS` object      |
| 302–303   | Global `state` object                        |
| 304–305   | Screen map + `showScreen()` utility          |
| 307–503   | All game logic and event handlers            |

### Screen State Machine

```
home ──[start]──► game ──[reveal]──► reveal ──[next]──► game (loop)
  ▲                 │                                       │
  └──[play again]──scores ◄────────────────────────────────┘
                                                  (after round 20)
```

Screens are plain `<div class="screen">` elements. Only one has `class="active"` at a time (`showScreen(name)` toggles this).

### State Object

```javascript
state = {
  players: [],              // string[] — player names
  selectedCategories: [],   // string[] — category IDs
  currentPlayerIdx: 0,      // index into players[]
  drinkCounts: {},          // { [playerName]: number }
  roundNum: 1,              // 1-based, max 20
  totalRounds: 20,
  currentQuestion: null,    // { q: string, opts: string[4] }
  currentCategory: null,    // category ID string
  activePlayerAnswer: null, // selected option index (0-3)
  otherGuesses: {},         // { [playerName]: optionIndex }
  usedQuestions: {}         // { [categoryId]: number[] } — used indices
}
```

### Data Model

**Categories** (9 total): `love`, `spicy`, `ethics`, `personality`, `life`, `wild`, `nostalgia`, `opinions` (Hot Takes), `hypotheticals` (Would You?)

Each category has multiple questions. Each question has exactly 4 options.

```javascript
// Category shape
{ id: string, label: string, icon: string }

// Question shape
{ q: string, opts: [string, string, string, string] }
```

---

## Game Flow

1. **Setup (home screen)**: ≥2 players added, ≥1 category selected → start
2. **Active player's turn (game screen)**: sees question, picks answer secretly (others look away)
3. **Guessing phase (game screen)**: other players each pick A/B/C/D for what the active player chose
4. **Reveal (reveal screen)**: active player's real answer shown; wrong guessers get +1 drink
5. **Next round**: rotate active player, increment `roundNum`
6. **End (scores screen)**: after round 20, show drink tallies sorted descending

---

## CSS Conventions

- **All CSS is embedded** in `<style>` within `<head>` — no external stylesheet.
- **CSS custom properties** defined on `:root` for spacing, typography, radius, transitions.
- **Two themes**: `[data-theme="dark"]` (default) and `[data-theme="light"]`, toggled via `data-theme` on `<html>`.
- **Color tokens**: `--color-bg`, `--color-surface`, `--color-surface-2`, `--color-surface-offset`, `--color-border`, `--color-text`, `--color-text-muted`, `--color-text-faint`, `--color-primary`, `--color-accent`, `--color-teal`, `--color-yellow`, `--color-red`.
- **Fluid typography** via `clamp()`: `--text-xs` through `--text-2xl`.
- **Animations**: `float` (logo bob), `screenIn` (screen transition), `confettiFall`, `pulse-red`.
- **Reduced motion**: respected via `@media (prefers-reduced-motion: reduce)`.

---

## JavaScript Conventions

- All code runs in a single `<script>` tag at end of `<body>`.
- Theme toggle is wrapped in an IIFE; everything else is module-level (implicit global scope).
- **No classes** — pure functions + a single mutable `state` object.
- DOM is manipulated via `innerHTML` for larger render operations, `textContent` for simple updates.
- Event listeners are registered once on page load (for static elements) or re-attached after innerHTML replacement (for dynamic elements like answer buttons).
- Question cycling: used question indices tracked in `state.usedQuestions`; pool resets when all questions in a category are exhausted.
- **Confetti**: `spawnConfetti('red')` on wrong guesses, `spawnConfetti('gold')` on game end.

---

## How to Run

Open `drink-and-tell.html` in any modern browser. No server, no install, no build step required.

```bash
# macOS
open drink-and-tell.html

# Linux
xdg-open drink-and-tell.html

# Or just drag the file into a browser tab
```

The font (Cabinet Grotesk) loads from Fontshare CDN; system fonts are the fallback when offline.

---

## Development Guidelines

### Making Changes

- **Edit only `drink-and-tell.html`** — there is no other source file.
- Keep CSS in the `<style>` block, JS in the `<script>` block at end of `<body>`.
- Preserve the existing CSS custom property system when adding new colors or sizes — do not use raw hex/px values where a token already exists.
- When adding questions, maintain exactly **4 options per question** (`opts` array length = 4).
- When adding a new category, add it to both `CATEGORIES` (for UI rendering) and `QUESTIONS` (for question data).

### State Management Rules

- All mutable game state lives in the `state` object — do not create other top-level state variables.
- `startGame()` is the canonical reset point — it re-initializes all runtime state fields.
- `loadQuestion()` advances the game state for a new round; call it before `showScreen('game')`.

### UI Patterns

- New screens follow the `.screen` / `.screen.active` pattern — add the div to `<body>`, register it in the `screens` map, and show via `showScreen(name)`.
- Render functions (`renderQuestionCard`, `renderGuessingPhase`) set `cardArea.innerHTML` and then attach new event listeners — this is intentional, not a bug.
- Buttons that should be conditionally disabled use the `disabled` attribute; CSS handles the `.btn-primary:disabled` visual state.

### Adding Content

To add new questions to an existing category:
```javascript
// In the QUESTIONS object, append to the relevant array:
love: [
  // ... existing questions ...
  { q: "New question text?", opts: ["Option A", "Option B", "Option C", "Option D"] },
]
```

To add a new category:
```javascript
// 1. Add to CATEGORIES array
{ id: 'newcat', label: 'New Cat', icon: '🆕' }

// 2. Add to QUESTIONS object
newcat: [
  { q: "...", opts: ["A","B","C","D"] },
  // ... at least a few questions recommended
]
```

---

## Deployment

The app auto-deploys to GitHub Pages on every push to `main` via `.github/workflows/deploy.yml`. The workflow copies `drink-and-tell.html` to `_site/index.html` and uses the standard `actions/deploy-pages` action. No build step is involved.

---

## Known Limitations

- **No persistence** — all state is lost on page refresh.
- **No duplicate name prevention** — identical player names are silently rejected by the `includes()` check but with no user feedback.
- **Fixed 20 rounds** — `totalRounds` is hardcoded in `state`; there's no UI to change it.
- **No input validation feedback** — if a player name already exists or is empty, the add is silently ignored.
- **Single device** — designed for pass-and-play on one device, not networked multiplayer.
- **Font requires internet** — Cabinet Grotesk is loaded from CDN; the app is otherwise fully offline-capable.
