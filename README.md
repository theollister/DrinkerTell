# Mereditch Presents: Drinky Tells

A single-file browser-based drinking party game. Players take turns answering taboo and personality questions in multiple-choice format — everyone else guesses the active player's answer, and wrong guessers earn a drink.

## How to Play

1. **Setup** — Add at least 2 players and pick one or more question categories.
2. **Answer secretly** — The active player sees a question and picks their answer (others look away).
3. **Everyone guesses** — Each other player guesses what the active player chose.
4. **Reveal** — The real answer is shown. Anyone who guessed wrong takes a drink.
5. **Repeat** — Players rotate through 20 rounds. At the end, a drink tally shows who needs to sober up.

## Running the Game

No install, no build step, no server required. Just open the file in a browser.

```bash
# macOS
open drink-and-tell.html

# Linux
xdg-open drink-and-tell.html
```

Or drag `drink-and-tell.html` into any browser tab.

> **Note:** The Cabinet Grotesk font loads from the Fontshare CDN. The game works offline but will fall back to system fonts without an internet connection.

## Categories

| Icon | Category    |
|------|-------------|
| ❤️   | Love        |
| 🌶️   | Spicy       |
| ⚖️   | Ethics      |
| 🧠   | Personality |
| 🌱   | Life        |
| 🃏   | Wild        |

## Features

- Dark and light theme toggle
- 6 question categories with 8 questions each (4 answer options per question)
- Question cycling — used questions are tracked and the pool resets when exhausted
- Confetti on wrong guesses and game end
- Designed for pass-and-play on a single device

## Tech Stack

Pure HTML, CSS, and vanilla JavaScript — no frameworks, no dependencies, no build tooling. The entire app lives in one file: `drink-and-tell.html`.
