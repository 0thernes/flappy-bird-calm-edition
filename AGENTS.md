# Project Context For AI Assistants

This file is for Codex, Cursor, Copilot, Continue, and similar tools.

Read this first before making changes in the repo.

## Read Order

1. `README.md`
2. `docs/architecture_master_blueprint.md`
3. `CONTRIBUTING.md`
4. `tests/smoke-test.mjs`
5. `.memory/security.md` before touching storage, network, or DOM injection

## What This Project Is

A solo-built, browser-first GlidieBirdie variant tuned for comfort and clarity.

The runtime is intentionally simple:

- one HTML shell
- one CSS file
- one game engine file
- one small service worker
- no build step
- no runtime dependencies
- no backend

## Hard rules

- Do not add a build step.
- Do not add runtime dependencies.
- Do not add remote assets, hosted fonts, analytics, or CDN imports.
- Do not add a backend or network data flow for gameplay state.
- Do not split `game.js` into modules.
- Do not split `style.css` into multiple files.
- Keep the service worker tiny and app-shell-focused.

## Change And Merge Flow (required — enforced by branch protection)

`main` is protected; direct pushes are rejected. Every change — human or agent — follows the same path:

1. Create a branch.
2. Commit there.
3. Open a pull request.
4. CI (`Smoke check`) must pass: static-checks, engine tests, typecheck, smoke.
5. The PR merges automatically once green — no human approval is required. Patch and minor Dependabot PRs auto-merge the same way; **major** Dependabot bumps are held for human review.

**Guardrail changes are the exception.** A PR that touches the project's own safety machinery — anything under `.github/`, `tests/`, `AGENTS.md`, `CLAUDE.md`, `CODEOWNERS`, or `package.json` — will NOT auto-merge. The `Guardrail check` blocks it until a human reviews the change and applies the `guardrail-approved` label. Agents must never apply that label themselves; leave guardrail changes for the human.

Do not attempt to push to `main` directly; it will be rejected. Keep PRs small and single-purpose so the gate stays fast and merge collisions stay rare.

## Runtime Rules

- Put tunable constants in `CONFIG`.
- Respect `state.dt` or `state.dtSec` for time-based behavior.
- Route persistence through the guarded storage helpers only.
- Keep mobile controls functional when gameplay changes.
- Preserve keyboard access, live announcements, and reduced-motion handling.
- Prefer extending the current structure over inventing a parallel one.

## Documentation Rules

If you change the public shape of the project, update the matching docs in the same change set.

- `README.md` for player-facing features and workflow
- `docs/architecture_master_blueprint.md` for runtime structure
- `CONTRIBUTING.md` for contributor workflow
- `SUPPORT.md` for troubleshooting changes

## Important Files

| File | Purpose |
| --- | --- |
| `game.js` | Entire runtime engine |
| `index.html` | Semantic shell, canvas host, controls, tutorial, PWA metadata |
| `style.css` | Layout, components, themes, accessibility styling |
| `service-worker.js` | Minimal offline shell caching |
| `manifest.webmanifest` | Install metadata |
| `tests/smoke-test.mjs` | Lightweight repo safety checks |
| `.memory/` | Canonical project notes and constraints |

## When In Doubt

- Prefer a smaller change.
- Preserve the current runtime shape.
- Check the architecture guide before refactoring.
- Run `npm run check` before closing the task.

## Cursor Cloud specific instructions

This is a zero-dependency, no-build, browser-first project. There is nothing to `npm install`
(no `dependencies`/`devDependencies`, no lockfile — do not add one). The standard commands in
`package.json` and `README.md` work as documented; notes below are only the non-obvious caveats.

- **Run the game:** serve the folder and open it, e.g. `python3 -m http.server 8000 --bind 127.0.0.1`,
  then visit `http://127.0.0.1:8000/index.html`. Note: the `npm run serve` script calls `python`,
  but this VM only has `python3` — invoke `python3` directly (or `python3 -m http.server 8000`).
- **Checks/tests:** `npm run check` (syntax + static + brand/link/license guards + 44 engine tests +
  smoke) runs offline with no install. `npm run typecheck` fetches TypeScript on demand via
  `npx -y -p typescript@5`, so it needs network the first time.
- **Driving gameplay from automation/headless:** `game.js` loads as a classic `<script defer>`, so its
  top-level globals are reachable from the page scope — `state`, `bird`, `pipes`, `CONFIG`,
  `doFlap()`, and `startGame()`. The engine only flaps on a `pointerdown` on the `#game` canvas or a
  `keydown`; a Space keypress is intentionally ignored while focus is on a button (e.g. right after
  clicking the tutorial's dismiss button), so prefer real pointer clicks on the canvas or call
  `doFlap()` directly. Gap center for the next pipe is `pipe.topHeight + state.gap / 2`.
- **Layout gotcha:** the play canvas sits below the page header; in a short viewport scroll the canvas
  into view before interacting with it.
