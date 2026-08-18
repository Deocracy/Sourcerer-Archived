# Project Research Summary

**Project:** Sourcerer (Tauri 2 desktop shell + applet framework)
**Domain:** Dockable multi-pane desktop workbench (VS Code/Rider/Obsidian-class shell) with plugin applets
**Researched:** 2026-07-06
**Confidence:** HIGH overall (versions npm/docs-verified; pitfalls cross-checked against Tauri issue tracker)

## Executive Summary

Sourcerer is a Tauri 2 frameless desktop shell for researchers with a dockable workspace and modular applet framework — similar in complexity to VS Code/Rider/Obsidian but with a novel twist: a deliberately minimal plugin contract proven in an HTML prototype, and full pixel-perfect demo stubs for unbuilt applets alongside one real applet (Notes) in v1.

**Recommended stack:** Tauri 2.11 + React 18 (NOT 19) + Vite 5/6 (NOT latest 8 unverified) + TypeScript, bespoke CSS Modules + design tokens, Zustand for the shell store, `tauri-plugin-store` for persistence. Deliberately avoid component libraries, generic drag-drop libraries, Redux, and dynamic plugin loading.

**Primary risks:**
1. **Porting drag algorithms correctly** — React 18 StrictMode cleanup + pointer capture + passive listeners must work or drag will feel broken.
2. **Persisted-state robustness** — workspace save/restore with schema versioning + corruption handling must be built from day one or adding applets mid-development will break saved layouts.

Both risks are well-understood with clear mitigation strategies (see PITFALLS.md).

## Key Findings

### Stack (STACK.md)

- Tauri 2.11.x (CLI 2.11.4, API 2.11.1), **React pinned 18.2.0** (npm `latest` = React 19.2.7 — wrong for this project), Vite per `npm create tauri-app@latest` scaffold (NOT hand-picked 8.x with unverified Rolldown toolchain)
- Zustand 5.0.14 for the single shell store; `tauri-plugin-store` (JS 2.4.3) bridged via persist middleware; defer `tauri-plugin-sql` until an applet needs relational data
- **No DnD library** — port the prototype's native Pointer Events hit-testing (`hitTest`, `performDock`, `startDockDrag`) directly
- CSS Modules + `tokens.css` custom properties; no Tailwind, no runtime CSS-in-JS
- Fonts bundled locally (Fontsource IBM Plex Sans/Mono/Serif); no Google Fonts at runtime
- `ai_complete` as async Tauri command; Channels (not events) if streaming is needed later

### Features (FEATURES.md)

- **Table stakes:** 5-zone drag/dock, tabs, workspace persistence, named layouts, 3-mode left rail, focus management, resize bounds, window controls, Home view
- **Differentiators:** high-fidelity demo stubs (the "part demo" sales pitch — no competitor precedent), minimal applet framework, persistent AI assistant (stubbed), proposal review UX, metro Home dashboard
- **Anti-features / defer:** multi-window (don't hard-assume one window architecturally, but don't build), command palette, real AI backend, Databasise integration, other applets, full a11y
- **Gaps the handoff doesn't spec:** focus/keyboard-routing model, corrupt/stale persisted-state fallback, numeric resize clamp bounds — make these explicit requirements

### Architecture (ARCHITECTURE.md)

- Single Zustand shell store: `dockTree`, `activePaneId`, rail state, assistant state, `savedLayouts`; applet-private state stays behind `host.storage`
- **Static registry** (compile-time imports), not `import.meta.glob` dynamic loading — runtime plugins only matter for a future Applet Builder milestone
- Host API as per-mount factory closure: `storage` (namespaced `sourcerer:<key>:<k>`), `ai` (stub), `open`, `instanceId`, `theme`
- Persistence: debounced whole-store writes (~250ms) with `schemaVersion` + migration chain + load-time validation + graceful fallback; explicit flush on `close-requested` (tauri-plugin-store corruption issue #3085)
- Build order: scaffold → store/persistence skeleton → dock tree/rail → registry/host/stubs → Notes → assistant/Home

### Pitfalls (PITFALLS.md) — Top 5 Critical

1. **Drag region swallows clicks** — `data-tauri-drag-region` only on the spacer; manually test every title-bar button (Phase 1)
2. **React 18 StrictMode double-invokes drag listeners** — proper cleanup, keep StrictMode ON (Phase 2)
3. **Missing pointer capture breaks fast drags** — `setPointerCapture()` or window-level binding matching the prototype (Phase 2)
4. **Store writes not crash-safe** — app-level debounce + validation + `.bak` + flush-on-close (persistence phase)
5. **Schema drift breaks restore** — `schemaVersion` + migrations + validation from day one (persistence phase)

Also: WebView2 DPI gap silently breaks pixel-perfect 1px borders at 125%/150% Windows scaling — verify PerMonitorV2 awareness in Phase 1.

## Suggested Phase Structure

1. **Shell Foundation** — frameless window, custom title bar, window controls, DPI verification, local fonts, scaffold pinning (React 18/Vite). Hard prerequisite for everything.
2. **Workspace Core** — dock tree + rail port (highest-risk phase; read `support.js` listener/pointer pattern before spec), persistence skeleton with schema versioning, focus management, resize clamps.
3. **Applet Framework** — static registry, host API, demo stubs for all applets; finalize the applet module signature before any real applet exists.
4. **Notes Applet** — first real applet; proves the whole loop (registry → host → storage → ai seam).
5. **Shell Features** — Dashboard Assistant panel (stubbed AI), Home metro dashboard, named layouts polish.

Phases 1–4 hard-sequential; 5's two halves reorderable; later applets = independent per-applet phases.

## Gaps to Address During Phase Planning

- Focus-routing model (which pane receives keydown; ⌘↵ / y-d-n with multiple panes open) — sync with prototype before Phase 2 spec
- Persisted-state fallback (corrupted JSON, missing applet keys → default workspace + placeholder, never crash)
- Numeric resize clamp values for rail/assistant/splits — extract from prototype
- Verify prototype `prune()` handles last-tab-close correctly by testing the .dc.html directly
- Whether Databasise's `cargo run` vs `cargo tauri dev` landmine reproduces in a fresh scaffold — verify at scaffold time

## Research Flags for Phase Planning

- **Phase 2 needs deep planning research:** prototype pointer-event pattern + StrictMode-safe port strategy; schema-versioning structure
- **Phases 1/3/4 are standard patterns** — skip heavy research
- **Phase 5:** test assistant resize grip for passive-listener scroll jank

## Sources

See per-dimension files: STACK.md (npm registry + v2.tauri.app verified), FEATURES.md (VS Code docs cross-checked), ARCHITECTURE.md (Tauri docs + GitHub #3085), PITFALLS.md (Tauri GitHub issue tracker).
