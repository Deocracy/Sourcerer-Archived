# Power Browser 1.0 — Project Brief

Status: DRAFT — awaiting confirmation (2026-08-19). This is the founding
document for GSD (`/gsd-new-project` will consume it). All supporting research
is copied into `docs/research/`.

## What we are building

A **Firefox-ESR fork with a completely rebuilt GUI, hosting Theia IDE as a
first-class tab type in one unified tab strip**. Browser tabs and editor tabs
live in the same strip, move as one, and share session restore. The Theia
backend runs as a bundled Node.js sidecar process on localhost, supervised by
the browser.

This replaces the earlier Electron plan. The Firefox route was chosen because
it inherits Widevine DRM, full WebExtensions (uBlock, password managers),
Mozilla's security patch stream, Bergamot local translation, and the
open-source Firefox AI Runtime — none of which an Electron browser gets.

## The thesis

One application where web and IDE are peers, not an IDE with a preview pane or
a browser with a code plugin. Everything is a URL; every tab — web page or
editor — is a first-class citizen of one navigation model. The GUI is **fully
customizable**: every visible element addable, changeable, removable, ideally
at runtime in JS/HTML/CSS without a rebuild.

## Architecture (settled — see docs/research/power-browser-architecture.md)

1. **Power Browser GUI** — our privileged XHTML/CSS/JS (`power.xhtml` replaces
   browser.xhtml's visible UI). Unified Tab Strip renders from a single
   `TabModel: [{kind: web|ide, ...}]`.
2. **Firefox frontend machinery** — kept invisible: tabbrowser.js,
   SessionStore, WebExtensions host, Widevine, translation, AI Runtime.
3. **Gecko engine** — untouched.
4. **Theia backend** — bundled Node sidecar on `127.0.0.1:<port>`,
   token-authenticated, spawned/health-checked/restarted by
   `TheiaService.sys.mjs`.
5. **Theia Bridge** — two halves: `@power/browser-bridge` (Theia extension,
   stable public APIs only) ↔ `PowerBridge.sys.mjs` (JSWindowActor relay).
   Mirror, don't query: chrome keeps a push-updated snapshot of Theia tab
   state so IDE tabs repaint as fast as web tabs.
6. **`PowerBrowserAPI.sys.mjs`** — the single anti-corruption layer; the ONLY
   module touching Firefox internals (one file to fix per ESR break).

## Settled decisions (do not re-litigate without new evidence)

- **Firefox ESR fork, not Electron/Chromium** (research 2026-08-19).
- **Repo = Zen model**: `upstream/` (ESR tag) + `patches/` + `power/` (our
  tree) + CI that rebases each ESR point release and fails loudly.
- **Theia in browser target** inside a Firefox `<browser>` element — all
  Electron-specific advice (titleBarStyle etc.) is irrelevant.
- **Extension model** (D1–D7 in docs/research/extension-model-decisions.md):
  npm for Theia extensions, self-hosted Open VSX for plugins, WebExtensions
  keep working via toolkit. A plugin-facing browser API mirrors the
  WebExtensions `browser.*` shape, is gated on a permission model (D6), and is
  milestone 6+ (D7).
- **GUI customizability** (G1–G5 in docs/research/gui-customizability-model.md):
  build-once customization bridge — a userChrome.css-equivalent CSS layer plus
  a dev-flagged privileged JS layer. Two accepted walls: webview interiors and
  codicon glyphs.
- **Desktop only** (Windows/macOS/Linux). Mobile out of scope. Develop on
  Linux; Win+mac CI at milestone 2; signing deferred to milestone 5.
- **Browser + Theia sidecar ship/update in lockstep** through our own update
  channel.

## Build order → proposed GSD milestones

1. **Boot** — Firefox ESR builds under Power Browser branding, stock UI.
   Repo structure, patch pipeline, CI rebase discipline. (pure infra)
2. **Shell** — `power.xhtml`: web-only unified tab strip + toolbar/URL bar
   driven through gBrowser. Win+mac CI added here.
3. **Theia** — sidecar supervisor + one "IDE" tab loading Theia;
   bridge open/close/activate/list. **Settles two architectural decisions
   that are cheap now, miserable later: tab-state mirroring and
   focus/keyboard arbitration (Ctrl-W / Ctrl-Tab / Ctrl-P ownership).**
   Earliest daily-drivable build.
4. **Unify** — editor tabs as first-class tabs: open/close/reorder across
   kinds, optimistic-close reconcile for dirty editors.
5. **Re-host** — extension buttons/popups, permission prompts, downloads
   panel, find bar, context menus. Signing/packaging. Earliest shareable
   build.
6. **Polish** — cross-boundary drag into Lumino docks, mixed-tab session
   restore, startup skeletons, theme/zoom seam, AI sidebar (Firefox AI
   Runtime + provider-agnostic), plugin-facing browser API groundwork.

**Power Browser 1.0 = through milestone 5** (shareable: installable, signed,
extensions and permission prompts work, unified tabs). Milestone 6 items are
1.x unless promoted.

## Open questions (for GSD discuss phases, not now)

- Which ESR tag to base on (pick at milestone 1 start).
- Focus/keyboard arbitration policy detail (milestone 3 decision point).
- Custom telemetry — wanted (TEMP NOTES); design opt-in model, decide milestone.
- How Firefox's modularity is broken up (study task, milestone 1–2).
- Theia's Docker story and enterprise collaboration (@theia/collaboration/OCT)
  — relevance to Power Browser TBD (TEMP NOTES).
- Registry hosting (Open VSX instance) timing — needed by milestone 5?

## Source research (docs/research/)

| File | What it is |
|---|---|
| power-browser-architecture.md | The master plan: layers, components, build order, packaging |
| extension-model-decisions.md | D1–D7 extension/plugin/registry decisions, GUI risk ranking |
| gui-customizability-model.md | Verified GUI editability model G1–G5, cost tiers, the two walls |
| firefox-fork-browser-ide-research.md | Why Firefox fork beat Electron |
| tab-management-research/ | Tab paradigm history, academic studies, user preferences, everything-is-a-URL, Theia shell mechanics |

## Next steps

1. Confirm this brief (or edit it).
2. `git init` this folder; run `/gsd-new-project` here — this brief plus
   `docs/research/` is the context input (or bootstrap via `/gsd-ingest-docs`).
3. Milestone 1 (Boot) becomes the first GSD phase set.
