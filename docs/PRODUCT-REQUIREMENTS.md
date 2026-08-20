# Sourcerer — Product Requirements (the Theia direction)

Status: CONFIRMED 2026-08-19. Product-level requirements for the Theia-based
Sourcerer. Everything in `docs/research/` is background research, not
commitments. GUI design is explicitly out of scope until after v4.0 — the
v4.0 GUI is default Theia, unchanged.

**Milestone mapping:** `.planning/` tracks milestones. **v3.0 Theia
Foundation** (already defined) is the prerequisite — repo reorg + the
theia-ide fork building and running on the NixOS host; it covers the
groundwork of R2. **v4.0 = Sourcerer 4.0**, the first shipped rebranded
product: R1 (= v3.0's deferred BRAND-01..04), R3, R4, R4a. Versions v1.0–v2.0
are the retired Tauri era.

## v4.0 requirements

- **R1 — Rebrand.** The product is Eclipse Theia IDE renamed to
  **Sourcerer**: application name, window title, icon/logo, welcome tab,
  about dialog, installer name. No other GUI change from Theia default.
  It replaces the earlier Tauri-based Sourcerer workbench as the product
  carrying that name ("Power Browser" was a placeholder name).
- **R2 — Own build.** We build and package it ourselves from Theia sources
  (own npm workspace pinned to a Theia release), producing an installable
  **Electron desktop app** — the same target as stock Theia IDE. Develop on
  Linux; Windows/macOS packaging follows. The specific Theia tag to pin is
  decided at build start (local sources are at v1.74.1).
- **R3 — Extension-based internals.** Everything we add is our own Theia
  extensions (`@sourcerer/*` packages) composed into the product. We do not fork
  or patch Theia core; upstream Theia releases can be adopted by re-pinning.
- **R4 — Incremental growth.** Parts are added as we go, each as a separate
  extension, without rework of what came before. The internals must be
  designed now to support the post-4.0 requirements below.
- **R4a — GUI customization bridge ships in v4.0.** One build-once Theia
  extension (`@sourcerer/customize`) exposing a runtime CSS layer
  (userChrome.css equivalent) and a dev-flagged privileged JS layer, so all
  post-4.0 GUI work (adding, changing, removing components) happens at
  runtime without a rebuild. It changes nothing visually by default —
  R1's "no GUI change" holds. Known limits accepted: webview interiors and
  codicon glyphs (see docs/research/gui-customizability-model.md). Heavier
  permanent features still land as compiled `@sourcerer/*` extensions.

## Post-4.0 requirements (internals must not preclude them)

- **R5 — Web browsing as first-class tabs** — web pages and editors as peers
  in one tab model: URL bar, back/forward, web-view tabs. Achievable as a
  compiled `@sourcerer/*` extension on the v4.0 base; does NOT include Firefox
  extension support (that is R7 — WebExtensions need Firefox's toolkit host,
  which no Theia extension can provide).
- **R6 — Fully customizable GUI** — every visible element addable,
  changeable, removable, ideally at runtime without a rebuild.
- **R7 — Real-browser capabilities** — DRM playback, WebExtensions,
  site isolation (the Firefox-fork research path).
- **R8 — AI integration** — Theia AI is already in the framework; our own
  agents/config later.
- **R9 — Extension distribution** — plugin registry (self-hosted Open VSX)
  when third-party distribution matters.

## Open items (not yet requirements)

- Opt-in telemetry (wanted; design undecided).
- Enterprise collaboration (@theia/collaboration / OCT) — relevance TBD.
- Docker-based deployment — understanding TBD.

## Explicitly out of scope for now

- Any GUI change beyond the rename (R1). GUI components are a post-4.0
  discussion. Sketches in `.planning/sketches/` are parked, not decisions.
- Choosing between internal-design options — internals that satisfy R3/R4
  and don't preclude R5–R9 can be designed pre-build, but that design is a
  separate document, not this one.
