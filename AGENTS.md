# AGENTS.md

Vanilla JS Asteroids clone. No dependencies, no bundler, no build step, no tests, no CI.

## Run

- `game.js` is loaded by a classic `<script>` tag in `index.html` (no ESM). Opening `index.html` over `file://` works.
- Local server if needed: `npx serve .` → `http://localhost:3000`.
- There is no `package.json`; do not invent npm scripts or add tooling without asking.

## Architecture

- Everything lives in one file, `game.js` (~420 lines). Sections are delimited by `// ── Name ──` comments — follow that convention for new blocks.
- Load order matters for anything new: another `<script src="...">` must be added in `index.html` before `game.js`; there is no `import`.
- Canvas is a fixed 800x600 defined in two places that must stay in sync: `W`/`H` in `game.js:5-6` and the `<canvas>` attributes in `index.html`. Center-based logic (`W/2`, `H/2`) and `wrap()` assume them.
- No devicePixelRatio scaling; the canvas is intentionally low-res/fixed-size.

## Game loop conventions

- `update(dt)` / `draw()` drive everything; every entity implements `update(dt)` and `draw()` plus a `dead` flag, and the global arrays are manually `.filter()`ed each frame.
- Motion values are per-second units multiplied by `dt` (e.g. `SPEED = 520`, `THRUST = 260`). Never use per-frame deltas — `dt` is clamped to `0.05` max and is `0` on the first frame.
- Space is toroidal: always move through `wrap(v, max)` (`game.js:27`), never plain clamping.
- One-shot input goes through `pressed(code)` (`game.js:20`), which is consume-once — calling it twice in a frame returns `false` the second time. Held-state input reads `keys[code]` directly.

## State machine

`state` is `'playing' | 'dead' | 'gameover'` (`game.js:241`); each branch in `update()` early-returns, so new features must handle the branch they belong to. `space` restarts only in the `gameover` branch. `ship.reset()` handles both new game and respawn (it also sets 3s invincibility + blink effect).

## Gotchas

- `POINTS`, `RADII`, `SPEEDS` (`game.js:61-63`) are indexed by size 1..3 but written 1→3, so values read in descending order (size 3 = 20 pts, size 1 = 100). Index 0 is unused padding.
- User-facing strings: all UI text (`drawOverlay`, `drawHUD`) and most comments are in **Spanish**; `index.html` sets `lang="es"`. Match Spanish for user-visible text and comments.
- `ctx.textAlign` leaks between draw helpers — set it explicitly before each `fillText`. Always pair canvas transforms with `ctx.save()`/`ctx.restore()`.
