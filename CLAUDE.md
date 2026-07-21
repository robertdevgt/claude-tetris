# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Vanilla Tetris implementation: HTML5 Canvas + CSS + JavaScript. No dependencies, no build step, no package.json, no test suite.

## Running

Open `index.html` directly, or serve statically (e.g. `python3 -m http.server 8000`, `npx serve .`). There is no build/lint/test command — changes to `game.js` are live on page reload.

## Architecture

Everything lives in `game.js` (~300 lines) as top-level state + functions (no classes/modules). Key pieces:

- **Board model**: `board` is a `ROWS × COLS` matrix; each cell is `0` (empty) or a piece-color index `1–7`.
- **Pieces**: `PIECES` are square matrices keyed by color index; rotation is done via `rotateCW` (transpose + reverse rows), not by storing pre-rotated states.
- **Collision** (`collide`): checks board bounds and existing fixed blocks for a given shape/offset.
- **Wall kicks** (`tryRotate`): after rotating, tries offsets `[0, -1, 1, -2, 2]` until a non-colliding position is found.
- **Game loop** (`loop`): driven by `requestAnimationFrame`; accumulates elapsed time in `dropAccum` and advances the piece one row once `dropInterval` is exceeded.
- **Line clearing** (`clearLines`): scans bottom-up, splices full rows out and unshifts empty rows in; re-checks the same index after a splice.
- **Scoring/leveling**: `LINE_SCORES = [0,100,300,500,800]` × current level; level increases every 10 lines; `dropInterval = max(100, 1000 - (level-1)*90)`.
- **Ghost piece** (`ghostY`): projects the current piece straight down to its landing row, drawn at `globalAlpha = 0.2`.
- **Rendering**: `draw()` redraws the full board each frame (grid, locked blocks, ghost, current piece) onto `#board`; `drawNext()` renders the preview piece onto a separate `#next-canvas`.

Flow: `init()` builds the board, seeds `next`, calls `spawn()`, and starts the loop. `spawn()` promotes `next` to `current` and generates a new `next`; if the new `current` immediately collides, `endGame()` fires and the overlay shows GAME OVER. Keyboard handling (`keydown`) drives movement/rotation/soft-drop/hard-drop/pause; `P` toggles pause independent of `gameOver` state.

## Tunable constants (top of `game.js`)

`COLS`, `ROWS`, `BLOCK`, `COLORS`, `LINE_SCORES`, `dropInterval`. If `COLS`/`ROWS`/`BLOCK` change, also update the `width`/`height` attributes of `<canvas id="board">` in `index.html` to match (`COLS × BLOCK` by `ROWS × BLOCK`).
