# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

Classic Tetris implemented in vanilla JavaScript with HTML5 Canvas and CSS. No dependencies, no build step, no package manager — just three files (`index.html`, `style.css`, `game.js`).

## Running the game

Open `index.html` directly in a browser, or serve it statically:

```bash
python3 -m http.server 8000
# or
npx serve .
```

There is no build, lint, or test tooling in this repo — it's plain HTML/CSS/JS consumed as-is by the browser.

## Architecture

All game logic lives in `game.js` (a single file, no modules). Key pieces:

- **Board model**: `board` is a `ROWS × COLS` matrix; each cell is `0` (empty) or a color index `1–7` identifying the locked piece.
- **Pieces**: defined in `PIECES` as square matrices. Rotation (`rotateCW`) is a transpose + row reversal, not a lookup table of rotation states.
- **Collision** (`collide`): checks board bounds and overlap with locked cells; used both for movement and for the ghost-piece projection.
- **Wall kicks** (`tryRotate`): after rotating, tries horizontal offsets `[0, -1, 1, -2, 2]` in order and takes the first that doesn't collide.
- **Game loop** (`loop`): driven by `requestAnimationFrame`, accumulates elapsed time in `dropAccum` and advances the piece one row when it exceeds `dropInterval`.
- **Line clearing** (`clearLines`): scans bottom-up, splices full rows out and unshifts empty rows at the top; re-checks the same row index after a splice.
- **Scoring/leveling**: `LINE_SCORES = [0, 100, 300, 500, 800]` multiplied by `level`; level increases every 10 lines; `dropInterval = max(100, 1000 - (level-1)*90)`.
- **Ghost piece**: `ghostY()` projects the current piece straight down until it would collide, drawn at `globalAlpha = 0.2`.

Global mutable state (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, timing vars) lives at module scope and is reset in `init()`, which is also called by the restart button.

Two canvases are used: `#board` (main play field) and `#next-canvas` (next-piece preview), each with its own 2D context and draw function (`draw()` / `drawNext()`).

## Tunable constants (top of `game.js`)

`COLS`, `ROWS`, `BLOCK`, `COLORS`, `LINE_SCORES`, `dropInterval`. If `COLS`/`ROWS`/`BLOCK` change, update the `width`/`height` attributes of `<canvas id="board">` in `index.html` to match (`COLS × BLOCK`, `ROWS × BLOCK`).
