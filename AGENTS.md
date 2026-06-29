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

- There is nothing to install: the project has zero runtime and dev dependencies, no `node_modules`, and no build step. Node 18+ (the VM has Node 22) is all that is needed for the checks.
- Validate changes with `npm run check` (syntax + static/brand/link/license guards + engine tests + smoke) and `npm run typecheck`. `npm run typecheck` pulls TypeScript on demand via `npx` and therefore needs network access.
- Serve the app with `python3 -m http.server 8000 --bind 127.0.0.1`, then open `http://127.0.0.1:8000/`. Do NOT use `npm run serve`: that script calls bare `python`, which does not exist on the VM (only `python3`).
- Gameplay is a real-time, twitch-timed flyer. Automated GUI testers cannot reliably score by flapping because per-action input latency lets the bird flap-and-fall before the next click lands — this is a tooling limitation, not a bug. The flap/score/collision logic is covered by the headless engine tests (e.g. `flap sets the upward impulse`, `passing six pipes scores six points`, `FirstFlight unlocks at score 1`); rely on those for scoring verification, and use the browser only to confirm the app loads, renders, and responds.
