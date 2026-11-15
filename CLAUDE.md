# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Vanilla JavaScript Tetris. Three files, no dependencies, no build step, no package.json, no tests:

- `index.html` — DOM structure, the two `<canvas>` elements (`#board` 300×600, `#next-canvas` 120×120), the pause/game-over overlay, loads `game.js`.
- `style.css` — dark/retro arcade visual theme.
- `game.js` (~300 lines) — all game logic.

## Running

No install or build. Either open `index.html` directly in a browser, or serve it statically (e.g. `python3 -m http.server 8000`, `npx serve .`) and visit `http://localhost:8000`. There is no lint, test, or build command in this repo.

## Architecture (`game.js`)

Everything lives in module-level state and top-level functions — no classes, no modules.

- **Board model**: `board` is a `ROWS × COLS` matrix; each cell is `0` (empty) or a color index `1–7` identifying which piece locked there.
- **Pieces**: `PIECES` are square matrices (index 0 unused, 1–7 = I,O,T,S,Z,J,L). `randomPiece()` deep-copies a shape into `{ type, shape, x, y }`.
- **Rotation**: `rotateCW(shape)` transposes + reverses rows. `tryRotate()` applies it and attempts wall kicks via the offsets `[0, -1, 1, -2, 2]`, keeping the first offset that doesn't collide.
- **Collision**: `collide(shape, ox, oy)` is the single source of truth for both bounds-checking and board-overlap checking; it's reused by movement, rotation, ghost-piece projection, and spawn-blocked detection.
- **Game loop**: `loop(ts)` runs via `requestAnimationFrame`, accumulates elapsed time in `dropAccum`, and advances the piece one row (or locks it) once `dropAccum >= dropInterval`.
- **Locking a piece**: `lockPiece()` → `merge()` (bakes the piece into `board`) → `clearLines()` → `spawn()` (promotes `next` to `current`, generates a new `next`; if the new piece immediately collides, calls `endGame()`).
- **Line clearing**: `clearLines()` scans bottom-up, splices out full rows and unshifts empty ones at the top, and on any clear recomputes `score`, `level` (`floor(lines/10)+1`), and `dropInterval` (`max(100, 1000 - (level-1)*90)`).
- **Scoring**: `LINE_SCORES = [0,100,300,500,800]` indexed by lines-cleared-at-once, multiplied by `level`; hard drop adds 2pts/row dropped, soft drop adds 1pt/row.
- **Ghost piece**: `ghostY()` projects `current` straight down until it would collide; `draw()` renders it at `globalAlpha = 0.2`.
- **Rendering**: `draw()` clears and redraws the whole board canvas every frame (grid → locked blocks → ghost → current piece); `drawNext()` renders the preview canvas independently.
- **Input**: a single `keydown` listener drives movement/rotation/soft-drop/hard-drop/pause; `P` toggles pause even while paused or game-over, everything else is ignored in those states.

Tunable constants are all at the top of `game.js`: `COLS`, `ROWS`, `BLOCK`, `COLORS`, `LINE_SCORES`, `dropInterval`. If `COLS`/`ROWS`/`BLOCK` change, update the `#board` canvas `width`/`height` in `index.html` to match (`COLS×BLOCK` by `ROWS×BLOCK`).
