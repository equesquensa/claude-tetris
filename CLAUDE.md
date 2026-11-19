# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Tetris in vanilla JavaScript + HTML5 Canvas. No dependencies, no package.json, no build step, no tests, no linter. The README and UI text are in Spanish; keep new user-facing strings consistent with that.

## Running

Open `index.html` directly in a browser, or serve the folder (e.g. `python -m http.server` or `npx serve`) and reload to see changes.

## Architecture

Three files: `index.html` (layout + element IDs), `style.css`, and `game.js` (all logic, loaded as a classic script at the end of `<body>`).

`game.js` is a single-file, global-state design:
- Mutable game state lives in top-level `let` variables (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, `animId`, ...), all reset in `init()`. DOM elements are looked up once at the top by ID, so renaming IDs in `index.html` breaks `game.js`.
- Board is a `ROWS x COLS` matrix of ints: `0` = empty, `1–7` = piece type, which indexes both `COLORS` and `PIECES` (index 0 is `null` in both). Pieces are `{type, shape, x, y}` with `shape` a copied matrix; rotation is matrix rotation (`rotateCW`) plus horizontal wall kicks `[0,-1,1,-2,2]` in `tryRotate`.
- Game loop: `requestAnimationFrame` -> `loop(ts)` accumulates `dropAccum` and gravity-drops when it exceeds `dropInterval`, then calls `draw()` every frame. Lock flow: `lockPiece()` -> `merge()` -> `clearLines()` (updates score/level/`dropInterval`) -> `spawn()` (which calls `endGame()` if the new piece collides).
- Pause/game over stop the loop via `cancelAnimationFrame(animId)`; resuming or `init()` restarts it. Both use the shared `#overlay` element.
- Scoring: `LINE_SCORES[cleared] * level`, +1 per soft-drop cell, +2 per hard-drop cell. Level = `floor(lines/10)+1`; `dropInterval = max(100, 1000 - (level-1)*90)`.
- Input: one `keydown` listener (arrows, X, Space, P). Up arrow and X both rotate (clockwise only).
- Rendering: `draw()` paints grid, locked board, a ghost piece (`ghostY()`, alpha 0.2), then the current piece; `drawNext()` renders the preview on `#next-canvas` and is called from `spawn()`. `drawBlock` is shared by both canvases.
- Theming: dark by default; `themeToggle` sets `data-theme="light"` on `<html>` and persists the choice in `localStorage` (`'theme'`, wrapped in try/catch). Canvas colors (`--grid`, `--highlight`) are read from CSS variables via `cssVar()` at draw time, so new canvas colors belong in `style.css` variables for both themes. While paused or game over the loop is stopped, so a theme change must call `draw()` and `drawNext()` itself.
- Gotchas: `endGame()` runs from inside `spawn()` (so inside `lockPiece()`), and gravity in `loop()` drops without scoring, unlike `softDrop()`. Piece selection is uniform random (no 7-bag).

## CI

`.github/workflows/claude.yml` (responds to `@claude` mentions on issues/PRs), `claude-code-review.yml` (PR review) and `claude-issue-triage.yml` (issue triage). Work often arrives via `claude/issue-*` branches merged by PR into `main`.
