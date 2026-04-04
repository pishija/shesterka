# Project: Шестерка — Macedonian 6-Letter Wordle Clone

## Overview
A static single-file HTML/CSS/JS Wordle clone for Macedonian, using 6-letter words. No build step, no framework, no dependencies. Deployable to GitHub Pages as-is.

---

## Repository Structure

```
shesterka/
├── index.html          # entire game (HTML + CSS + JS inline)
├── words.js            # word list as JS arrays (answers + valid guesses)
├── extract-words.swift # one-time Swift script to generate words.js from Hunspell dict
└── README.md
```

All game logic lives in `index.html`. `words.js` is a separate file loaded via `<script>` so the word list can be updated independently without touching game logic.

---

## Word List Setup

- **Source:** [`gerazov/dictionary-mk`](https://github.com/gerazov/dictionary-mk) — Hunspell Macedonian spellcheck dictionary (`mk_MK.dic`)
- Write a one-time Swift script (`extract-words.swift`, not part of the final build) to:
  - Read `mk_MK.dic`
  - Strip Hunspell flags (anything after `/` on each line)
  - Filter to words that are exactly 6 Macedonian Cyrillic characters
  - Remove proper nouns (lines starting with uppercase)
  - Output as a JS file with two exported arrays
- Run locally with `swift extract-words.swift`
- Split output into two arrays in `words.js`:
  - `ANSWERS` — common, familiar words (hand-curated subset, ~500–1000 words)
  - `VALID_GUESSES` — full list of valid 6-letter words used for guess validation

---

## Game Logic

### Daily Word Selection

```js
const start = new Date('2025-01-01');
const today = new Date();
const dayIndex = Math.floor((today - start) / 86400000) % ANSWERS.length;
const answer = ANSWERS[dayIndex];
```

### Game State (localStorage)

Store under key `shesterka_state`:

```json
{
  "date": "2026-04-04",
  "guesses": ["ЗБОРОТ", "СТРАНА"],
  "gameStatus": "playing | won | lost"
}
```

On load: if stored date matches today, restore state. Otherwise reset.

### Guess Validation

- Must be exactly 6 characters
- Must exist in `VALID_GUESSES` (which includes all `ANSWERS`)
- Trigger shake animation if invalid

### Tile Coloring Logic

For each letter in the guess, assign one of three statuses:

- 🟩 `correct` — right letter, right position
- 🟨 `present` — right letter, wrong position
- ⬛ `absent` — letter not in word

**Important:** Handle duplicate letters correctly using a two-pass algorithm:
1. First pass: mark all `correct` letters, remove them from the pool
2. Second pass: mark remaining letters as `present` or `absent` based on what's left in the pool

---

## UI Components

### Board
- 6 rows × 6 columns grid
- Each tile: ~60×60px, border, large centered letter
- Flip animation on reveal (CSS `rotateX` transform)
- Shake animation on invalid guess

### Keyboard

Macedonian Cyrillic layout, 3 rows:

```
Row 1: Љ Њ Е Р Т Ѕ У И О П Ш Ѓ
Row 2: А С Д Ф Г Х Ј К Л Ч Ќ Џ
Row 3: Ж З Ц В Б Н М
Special keys: [⌫ BACKSPACE]  [ENTER]
```

- Keys update to reflect the best known status for each letter (correct > present > absent)
- Physical keyboard support: map Latin keypress events to their Cyrillic equivalents

### Modals

1. **Help modal** — shown automatically on first visit; explains the rules in Macedonian
2. **End-of-game modal** — win/loss message, reveals the answer if lost, share button
3. **Stats modal** *(optional, implement last)* — played, win %, current streak, max streak

### Header

- Title: **Шестерка**
- Icon buttons: Help (`?`) and Stats (`📊`)

---

## Sharing (Emoji Grid)

On win or loss, generate a shareable text block and copy to clipboard:

```
Шестерка 42 4/6

⬛🟨⬛⬛🟩⬛
🟨🟩⬛🟨⬛⬛
🟩🟩🟨🟩⬛🟨
🟩🟩🟩🟩🟩🟩
```

Use `navigator.clipboard.writeText()`. Show a brief "Копирано!" toast on success.

---

## Styling

- **Dark theme** by default
- CSS custom properties for all colors:

```css
:root {
  --color-correct: #538d4e;
  --color-present: #b59f3b;
  --color-absent:  #3a3a3c;
  --color-bg:      #121213;
  --color-text:    #ffffff;
  --color-tile-border: #565758;
  --color-key-bg:  #818384;
}
```

- Responsive layout — works on mobile; keyboard must fit on small screens without scrolling
- Font: `system-ui` or a safe stack that renders Cyrillic well (e.g. `Segoe UI, Helvetica Neue, Arial, sans-serif`)

---

## Animations

All via CSS `@keyframes`, triggered by adding/removing classes in JS:

| Animation | Trigger | Description |
|-----------|---------|-------------|
| `flip`    | On tile reveal | Each tile rotates on X axis; color applied at 50% point; staggered 0.1s per tile |
| `bounce`  | After winning row reveals | Tiles in winning row bounce vertically |
| `shake`   | Invalid guess | Row shakes horizontally |
| `pop`     | Letter typed | Tile briefly scales up |

---

## GitHub Pages Deployment

- Repo name: `shesterka` (or any name)
- In repo **Settings → Pages**, set source to `main` branch, root (`/`)
- `index.html` at the repo root is served directly — no build step needed
- Optional: add a `CNAME` file for a custom domain (e.g. `shesterka.mk`)

```
# CNAME (optional)
shesterka.mk
```

---

## Out of Scope (Keep Simple for v1)

- No hard mode
- No color-blind mode *(can add later as a CSS class toggle)*
- No backend, no API, no server
- No npm, no bundler, no framework

---

## Suggested Build Order

Follow this sequence — each step is independently testable:

1. **HTML structure + CSS** — scaffold the board, keyboard, and modals with static content; make it look right before any JS
2. **Placeholder word list** — add `words.js` with 20–30 hardcoded 6-letter Macedonian words for development
3. **Core game loop** — keyboard input (click + physical keyboard), tile filling, guess submission
4. **Tile coloring** — implement the two-pass coloring algorithm; update keyboard colors
5. **Animations** — add flip, shake, bounce, pop
6. **Persistence** — localStorage save/restore; daily word selection by date
7. **Modals** — help modal (first visit), end-of-game modal, stats modal
8. **Share button** — emoji grid generation, clipboard copy, toast notification
9. **Word extraction script** — write `extract-words.swift` to generate the real word list from the Hunspell `.dic` file; run locally with `swift extract-words.swift`
10. **Replace word list** — swap placeholder with real list; test edge cases (duplicates, special Macedonian letters like Ѓ, Ќ, Ѕ, Џ, Љ, Њ)
11. **README** — add usage instructions, word list attribution, and deployment steps

---

## Word List Attribution

The word list is derived from the OpenOffice Macedonian dictionary, originally developed by the Free/Libre Software Organisation of Macedonia (OSSM), adapted via [`gerazov/dictionary-mk`](https://github.com/gerazov/dictionary-mk). Include attribution in the README and a comment in `words.js`.
