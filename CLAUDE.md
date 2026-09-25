# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

There is no build, lint, or test tooling — this is dependency-free vanilla JS/HTML/CSS (no `package.json`).

Run the game by opening `index.html` directly, or serve it locally:

```bash
python3 -m http.server 8000   # or: npx serve .   /   php -S localhost:8000
```

Then visit `http://localhost:8000`. To verify a change works, open the page in a browser and play.

## Architecture

Three files, all logic lives in `game.js` (~300 lines, single global scope, no modules):

- **`index.html`** — DOM shell: `<canvas id="board">` (300×600, the 10×20 grid at 30px/cell), `<canvas id="next-canvas">` (next-piece preview), HUD spans (`score`/`lines`/`level`), and a shared `#overlay` used for both PAUSE and GAME OVER states.
- **`style.css`** — dark/retro theme only; no layout logic depends on it.
- **`game.js`** — game state and loop, structured as:
  - **Board model**: `board` is a `ROWS × COLS` matrix; each cell is `0` (empty) or a piece-color index `1–7`.
  - **Pieces**: `PIECES` are square matrices; `rotateCW` transposes + reverses rows. `tryRotate` applies wall kicks by trying offsets `[0, -1, 1, -2, 2]` and using the first that doesn't `collide`.
  - **Collision**: `collide(shape, ox, oy)` checks board bounds and overlap with locked cells — the single gate used by movement, rotation, and spawn.
  - **Game loop**: `loop(ts)` runs via `requestAnimationFrame`, accumulates `dt` in `dropAccum`, and drops the piece one row once `dropAccum >= dropInterval`.
  - **Locking**: `lockPiece()` → `merge()` (writes piece into `board`) → `clearLines()` (bottom-up scan, splice + unshift empty row) → `spawn()` (promotes `next` to `current`, generates new `next`; if the new piece immediately collides, calls `endGame()`).
  - **Scoring/leveling**: `LINE_SCORES = [0,100,300,500,800]` × `level`; hard drop adds 2 pts/cell, soft drop 1 pt/row. `level` increases every 10 lines; `dropInterval = max(100, 1000 - (level-1)*90)`.
  - **Rendering**: `draw()` clears and redraws grid + locked board + ghost piece (via `ghostY()`, alpha 0.2) + current piece every frame; `drawNext()` renders the preview canvas separately.
  - **Input**: a single `keydown` listener drives movement/rotation/soft-drop/hard-drop; `P` toggles pause independent of `gameOver`/`paused` guards.

All tunable constants (`COLS`, `ROWS`, `BLOCK`, `COLORS`, `LINE_SCORES`, initial `dropInterval`) live at the top of `game.js`. If `COLS`/`ROWS`/`BLOCK` change, update the `#board` canvas `width`/`height` in `index.html` to match (`COLS × BLOCK`, `ROWS × BLOCK`).
