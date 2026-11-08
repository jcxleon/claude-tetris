# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Vanilla JS Tetris (HTML5 Canvas). No dependencies, no build step, no tests, no linter. README and UI strings are in Spanish; keep new user-facing text in Spanish.

## Running

Open `index.html` directly, or serve statically (`python3 -m http.server 8000`, `npx serve .`) and visit `http://localhost:8000`.

## Architecture

Three files: `index.html` (markup: board canvas 300x600, `#next-canvas` 120x120, HUD spans, `#overlay`), `style.css`, and `game.js` (all logic, loaded as a classic script with `'use strict'`).

`game.js` is a single global-state module:

- **State** lives in top-level `let` variables (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, `dropAccum`, `animId`...), reset in `init()`. Restart button just calls `init()`.
- **Board** is a `ROWS x COLS` (20x10) matrix of ints; `0` = empty, `1–7` index into both `COLORS` and `PIECES`. A piece is `{type, shape, x, y}`; `shape` is a matrix whose cell values are the color index.
- **Game loop**: `loop(ts)` via `requestAnimationFrame` accumulates `dropAccum` and gravity-drops when it exceeds `dropInterval`, then `draw()`s every frame. Pause/game over cancel the frame and rely on `animId`; `togglePause` restarts `loop` manually.
- **Piece lifecycle**: `lockPiece()` → `merge()` → `clearLines()` (updates score/level/speed) → `spawn()` (promotes `next`, game over if spawn collides). Soft/hard drop award score directly (1/2 points per cell) in addition to line-clear scoring (`LINE_SCORES * level`).
- **Rotation**: `rotateCW` + `tryRotate` with horizontal-only kicks `[0,-1,1,-2,2]`; no SRS, no rotation of the O piece special-casing.
- **Input**: single `keydown` listener (Arrows, `X` rotate, Space hard drop, `P` pause) using `e.code`.
- Level = `floor(lines/10)+1`; `dropInterval = max(100, 1000 - (level-1)*90)`.
