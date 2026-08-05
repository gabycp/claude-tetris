# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

A classic Tetris implementation in vanilla JavaScript, HTML5 Canvas, and CSS — no dependencies, no build step, no `package.json`. The entire game logic lives in `game.js` (~300 lines).

## Running the game

There is no build/test/lint tooling. Open directly or serve statically:

```bash
open index.html              # macOS, just opens the file
python3 -m http.server 8000  # or any static server, then visit localhost:8000
```

## Architecture

Three files, no modules/bundler — `game.js` is loaded as a single classic script and relies on global state.

- **`index.html`** — DOM shell: `<canvas id="board">` (300×600, the play field) and `<canvas id="next-canvas">` (120×120, next-piece preview), plus HUD elements (`#score`, `#lines`, `#level`) and a shared `#overlay` used for both Pause and Game Over.
- **`style.css`** — dark/retro arcade styling only.
- **`game.js`** — all game state and logic, structured around a `requestAnimationFrame` loop:
  - **Board model**: `board` is a `ROWS × COLS` matrix; each cell is `0` (empty) or a color index `1–7` identifying which piece type locked there.
  - **Pieces**: `PIECES` are square matrices (index 0 unused/null). Rotation is done via `rotateCW` (transpose + reverse), not by storing pre-rotated states.
  - **Collision** (`collide`): checks board bounds and cell occupancy for a shape at a given offset.
  - **Wall kicks** (`tryRotate`): after rotating, tries offsets `[0, -1, 1, -2, 2]` columns until one doesn't collide.
  - **Game loop** (`loop`): accumulates elapsed time (`dropAccum`) each frame; once it exceeds `dropInterval`, the piece drops one row or locks via `lockPiece`.
  - **Locking a piece**: `lockPiece` → `merge` (bake shape into `board`) → `clearLines` → `spawn` (promote `next` to `current`, generate a new `next`; if the new piece immediately collides, `endGame` fires).
  - **Scoring**: `LINE_SCORES = [0, 100, 300, 500, 800]` for 1–4 lines, multiplied by `level`. Hard drop adds 2 points per row dropped; soft drop adds 1 point per row.
  - **Leveling/speed**: level increases every 10 lines; `dropInterval = max(100, 1000 - (level-1)*90)` ms.
  - **Ghost piece**: `ghostY()` projects the current piece straight down to its landing row; drawn with `globalAlpha = 0.2`.
  - All rendering (`draw`, `drawNext`, `drawGrid`, `drawBlock`) is manual Canvas 2D — no scene graph, redraw-everything-each-frame style.
  - Input is a single `keydown` listener switching on `e.code` (arrows, `KeyX` for rotate, `Space` for hard drop, `KeyP` for pause).

Tunable constants live at the top of `game.js`: `COLS`, `ROWS`, `BLOCK` (cell px size), `COLORS`, `LINE_SCORES`. If you change `COLS`/`ROWS`/`BLOCK`, also update the `<canvas id="board">` `width`/`height` attributes in `index.html` to match (`COLS × BLOCK` by `ROWS × BLOCK`).
