# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A classic Tetris game in vanilla JavaScript (ES6+), HTML5 Canvas, and CSS. **No dependencies, no build step, no package.json, no tests.** Three source files: `index.html`, `style.css`, `game.js`.

## Running

Open `index.html` directly (`open index.html`), or serve statically for a clean reload cycle:

```bash
python3 -m http.server 8000   # then open http://localhost:8000
```

There is nothing to build, lint, or test. To verify a change, reload the page and play.

## Architecture

All game logic lives in `game.js` (~300 lines, single global scope, `'use strict'`). It renders to two `<canvas>` elements defined in `index.html`: `#board` (300×600, the play field) and `#next-canvas` (the preview). Mutable game state is a set of module-level `let` variables (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, `dropAccum`, `animId`, ...); `init()` resets them all and is the single entry point (called on load and on restart-button click).

Key model conventions:

- **Board** is a `ROWS × COLS` array of ints. `0` = empty; `1–7` = a filled cell whose value indexes into `COLORS` and `PIECES` (same index identifies both the color and the tetromino type).
- **Pieces** are square int matrices in `PIECES` (index 1–7). Rotation is a pure 90°-CW transform (`rotateCW`), never mutated in place until accepted.
- **Collision** (`collide(shape, x, y)`) is the one gatekeeper for all movement — walls, floor, and settled blocks. Every move/rotate/drop tests it before committing.
- **Rotation with wall kicks** (`tryRotate`) tries horizontal offsets `[0, -1, 1, -2, 2]` and applies the first that doesn't collide.

Piece lifecycle: `spawn()` promotes `next` → `current`, generates a new `next`, and calls `endGame()` if the fresh piece already collides. `lockPiece()` = `merge()` (write piece into board) → `clearLines()` → `spawn()`.

Game loop: `loop(ts)` runs on `requestAnimationFrame`, accumulating elapsed time in `dropAccum`; when it exceeds `dropInterval` the piece steps down one row (or locks). `draw()` runs every frame: grid → settled board → ghost piece (`ghostY()` projected landing, alpha 0.2) → current piece. Pause/resume works by canceling and restarting the rAF loop (`togglePause`), resetting `lastTime` so accumulated time doesn't jump.

Scoring/leveling live in `clearLines`: `LINE_SCORES[cleared] * level`; level = `floor(lines/10)+1`; `dropInterval = max(100, 1000 - (level-1)*90)` ms. Hard drop adds 2 pts/cell, soft drop 1 pt/row.

Input is a single `keydown` listener at the bottom of `game.js`. `P` toggles pause always; all other keys are ignored while paused or game-over.

## Gotchas

- The `#board` canvas `width`/`height` in `index.html` are hardcoded to `COLS*BLOCK` × `ROWS*BLOCK` (300×600). If you change `COLS`, `ROWS`, or `BLOCK` in `game.js`, update the canvas attributes in `index.html` to match, or rendering breaks.
- DOM element lookups (`document.getElementById(...)`) are top-level in `game.js`, so element `id`s in `index.html` and the lookups must stay in sync.
- `README.md` is in Spanish and documents the same behavior in more detail.
