# AGENTS.md

## Project

Vanilla JS Canvas clone of Pac-Man. No framework, no build system, no tests, no lint, no `package.json`. The app is static files under `src/`; to run it, open `src/index.html` in a browser.

## Architecture (important)

- `index.html` loads scripts as plain `<script>` tags **in order**: `js/maze.js` → `js/game.js` → `js/render.js` → `js/main.js`. No ES modules.
- Files communicate exclusively through `window` globals: `maze.js` exposes `MAZE`, `TUNNEL_ROW`, `PACMAN_START`, `GHOST_STARTS`; `game.js` exposes `createGame`, `update`, `DIRS`; `render.js` exposes `draw`; `main.js` drives the loop. `game.js` reads maze globals, `render.js` reads `DIRS`.
- When adding a new JS file, add a matching `<script>` tag to `index.html` and expose symbols on `window`.
- The maze is an ASCII grid in `js/maze.js`, parsed into numbers: `1` wall, `2` dot, `3` door, `0` walkable. Grid is 28×31; canvas is 560×620 (TILE=20 in `render.js`) — changing maze dims requires updating the canvas size in `index.html`.
- `MAZE` is pristine and shared; `createGame()` copies it (`grid = MAZE.map(row => row.slice())`) and mutates only the copy. Never mutate `MAZE`.

## Workflow conventions

- **Spec-driven development is the required workflow** for new features (see README). Skills are installed at `.agents/skills/`: use the `spec` skill to design a feature (writes to `specs/`) and `spec-impl` to implement an approved spec. No specs exist yet — the first one starts the `specs/` folder.
- **Language: the project is Spanish.** Code comments, UI copy, README, and specs are written in Spanish. Reply in the same language as the user's prompt.

## Code style (differs from defaults)

Follow the existing style everywhere:
- Spaces inside parens and brackets: `( value )`, `[ value ]`, not `(value)`.
- Spaces around operators: `x + 1`, `=== 1`. No spaces after commas.
- Single quotes, semicolons, 2-space indent.
- Short comment headers at the top of each file explaining its role and interdependencies (e.g. `// game.js ... Depende de globals de maze.js`).