# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the game

No build step, no dependencies. Open directly or serve statically:

```bash
# Windows
start index.html

# Local server (any of these)
python3 -m http.server 8000
npx serve .
```

## Architecture

Three files, all logic in `game.js` (~300 lines):

- `index.html` — two `<canvas>` elements: `#board` (300×600) for the playfield, `#next-canvas` (120×120) for piece preview; `#overlay` for pause/game-over states.
- `style.css` — dark/retro aesthetic, no classes dynamically added except `hidden` toggled on `#overlay`.
- `game.js` — entire game state and loop. No modules, no classes, plain functions and global `let` variables.

### game.js internals

**State**: `board` is a `ROWS×COLS` 2D array where `0` = empty, `1–7` = piece color index. `current` and `next` are piece objects `{ type, shape, x, y }`.

**Game loop**: `requestAnimationFrame(loop)` accumulates delta time in `dropAccum`; when it exceeds `dropInterval`, the piece drops one row or locks.

**Piece locking flow**: `lockPiece()` → `merge()` (writes piece into board) → `clearLines()` → `spawn()` (promotes `next` to `current`). If the new piece immediately collides in `spawn()`, `endGame()` is called.

**Rotation**: `rotateCW(shape)` does transpose + row-reverse. `tryRotate()` tries kicks at offsets `[0, -1, 1, -2, 2]` before giving up.

**Speed**: `dropInterval = Math.max(100, 1000 − (level − 1) × 90)` ms. Level increments every 10 lines.

### Tunable constants (top of game.js)

| Constant | Default | Note |
|---|---|---|
| `COLS` / `ROWS` | 10 / 20 | Also update canvas `width`/`height` in `index.html` |
| `BLOCK` | 30 | Pixel size per cell |
| `COLORS` | 7 colors | Index 0 = null (empty) |
| `LINE_SCORES` | `[0,100,300,500,800]` | Multiplied by current level |
