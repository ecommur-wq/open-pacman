# AGENTS.md

Pac-Man clone in vanilla JS/HTML/CSS. Purpose: learning spec-driven development (see README, written in Spanish).

## How it runs

- No package.json, no build, no bundler, no tests, no lint. Do not add tooling unless asked.
- Run by opening `src/index.html` directly in a browser (no dev server required).
- Scripts are plain globals, not ES modules. Load order in `src/index.html` is required:
  `maze.js -> game.js -> render.js -> main.js`. Never convert to `import`/`export` or reorder.

## Architecture (only 4 JS files)

- `src/js/maze.js` — 28x31 maze as 31 strings of 28 chars, parsed to numbers.
  Exports as globals: `MAZE`, `TUNNEL_ROW`, `PACMAN_START`, `GHOST_STARTS`.
  Chars: `#` wall(1), `.` dot(2), ` ` empty(0), `-` pen door(3). Cell (x,y), origin top-left.
- `src/js/game.js` — state + rules. Depends on maze.js globals. Speeds are in cells/frame
  (`PACMAN_SPEED = 0.125` = 1/8 cell/frame) so movement aligns to the 8/10-frame grid —
  keep movement frame-aligned if you touch it.
- `src/js/render.js` — canvas drawing. Must read `game.grid` (not `MAZE`) so eaten dots
  disappear. `TILE = 20`; canvas is 560x620 = 28x31 tiles.
- `src/js/main.js` — loop (`requestAnimationFrame`), keyboard, overlay screens.

## Workflow: spec-driven

- Feature work goes through the repo skills (`.agents/skills/`): `spec` designs a spec into
  `specs/`, `spec-impl` implements an approved spec on a dedicated branch. Use them for
  non-trivial features instead of coding directly.
- Specs live in `specs/` (folder may not exist yet); filenames like `NN-slug.md`.
  A spec only counts as approved when its status line means "Approved" in any language.
- Specs are written in the language of the existing specs (project docs/comments/UI are
  Spanish). Match it.

## Gotchas

- `.agents/skills/` is installed from `klerith/fernando-skills` and tracked in
  `skills-lock.json` (hash-verified). Don't hand-edit those skill files.
- Comments and UI strings are Spanish; keep new ones consistent.
