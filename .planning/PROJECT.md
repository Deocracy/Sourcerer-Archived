# Sourcerer (Desktop Shell)

## What This Is

A Tauri 2 desktop application: a dockable multi-pane research workbench for scholars. The shell hosts **applets** (Library, Wiki, Graph, Chat, Notes, Writing Studio, Power Browser, Kanban, News, KeyPass, Databasise, Applet Builder…) plus a persistent right-hand **Dashboard Assistant** (AI chat panel). Part demo, part working application: every applet not yet built renders as a high-fidelity demo stub, and applets are implemented one at a time by registering into the applet framework, replacing their stub.

As of v1.0 the shell is fully interactive end-to-end: frameless window, dockable workspace, crash-safe persistence, applet framework with Notes as the real proving applet, metro Home dashboard, and a Dashboard Assistant backed by a **real** headless Pi sidecar behind `host.ai()`.

The **Databasise engine** (the existing Cozo+LightRAG wiki source-of-truth project at `D:\Vibe Coding\Databasise`, with REST + MCP surfaces) is a required core component of Sourcerer long-term — it will power the Wiki/Library/Graph applets. Its integration mode is deliberately deferred (see Key Decisions).

## Core Value

A pixel-perfect, fully interactive desktop shell where the applet framework demonstrably works end-to-end — one real applet (Notes) proves the loop (registry → host API → storage → AI seam), and every other applet is a believable stub ready to be replaced.

## Source of Truth (Design)

**`design-sync-setup-guide/design_handoff_bespoke_rails_shell/`** (adopted wholesale 2026-07-07, supersedes the original `design_handoff_sourcerer_tauri/` handoff):
- `Sourcerer Bespoke Rails.dc.html` — the working prototype built around the locked libraries (dockview-core, zustand, dnd-kit); rail/assistant resize + toggle behavior is the UX reference.
- `home-cards.js` etc. — Home dashboard card registry/renderers (ported verbatim in Phase 6).
- Tokens: 40px title bar, green `#86A38C` accent, IBM Plex, 0 border-radius inside, 10px radius on the outer window card only.

## Stack (shipped in v1.0)

- Tauri 2 (Rust backend), single frameless window (`transparent: true`, 10px card), custom title bar wired to window API
- React 18 + Vite + TypeScript; no component library; bespoke styling from design tokens
- dockview-core 2.0.0 (center dock) + dnd-kit (Home cards) + bespoke pointer events (rail/assistant resize)
- zustand 5.0.14 shell store; IBM Plex Sans/Mono/Serif bundled locally (OFL)
- Persistence: `tauri-plugin-store` (workspace.json + applets.json) mapping onto `host.storage`; graduate to SQLite when applets need real data
- `host.ai()` → Rust Tauri command → Node Pi sidecar (`@earendil-works/pi-coding-agent`), NDJSON streaming over a Channel; model zai-glm-4.7

## Current Milestone: v3.0 Theia Foundation

**Goal:** Replace the bespoke Tauri shell direction with a fork of Theia IDE as the Sourcerer frontend — a working, rebranded fork running locally as the environment Sourcerer is built up in incrementally.

**Target features:**
- Repo reorganization: freeze the superseded Tauri-shell and container-platform work into a `Sourcerer-v0/` folder (v2.0 Container Platform milestone frozen/superseded; phase archives kept)
- Theia IDE forked and building locally; dev workflow documented and repeatable
- Sourcerer branding/product customization applied (name, window title, theme direction)
- Fork runs as a working environment — the base future milestones extend

**Key context:** Same product goal (scholar research workbench: applets + Dashboard Assistant + Databasise engine), new foundation. Carry-over review is deliberately deferred — a later phase/milestone reviews what migrates from v1.0/v2.0 work (Pi assistant harness, Databasise integration plans, design tokens/theme, Nix flake/CI/Attic cache). This supersedes the VSCodium-foundation exploration verdict (`VSC Sourcerer Alternitive Exploration`, 2026-08-17).

## Requirements

### Validated

- ✓ Frameless Tauri window with custom title bar (minimize/maximize/close, drag region, DPI-correct 1px borders, local fonts) — v1.0 Phase 1
- ✓ Left rail: expanded/compact/hidden modes, drag-resize grip, reorder/pin/drag-out-to-dock — v1.0 Phase 2 (+ live-relayout fix in Phase 6 gap closure)
- ✓ Workspace dock tree: tabs, 5-zone docking, splits with resizers, edge-split, multi-instance tabs — v1.0 Phase 2 (dockview-core)
- ✓ Named layouts + full workspace persistence restored on launch, crash-safe, corrupt-fallback — v1.0 Phase 3
- ✓ Applet framework: registry loader, `host` API (storage/ai/open/instanceId/theme), demo stubs for all unbuilt applets — v1.0 Phase 4
- ✓ Notes as the first real applet (persistent notes + AI summarize via the seam) — v1.0 Phase 5
- ✓ Dashboard Assistant panel: sessions, thread, composer, proposals UI (y/d/n), resize/snap — v1.0 Phase 6
- ✓ Home: metro card dashboard with draggable sections, FLIP animation, assistant ＋MAKE CARD mint — v1.0 Phase 6
- ✓ Assistant harness: headless Pi sidecar behind `host.ai()`, streamed replies, honest degrade, history survives restart — v1.0 Phase 7

### Active

(None — define with `/gsd-new-milestone`.)

### Out of Scope

- Databasise engine integration (live Wiki/Library/Graph data) — integration mode undecided; applets stay stubs
- All other applets beyond Notes — stubs only, built one at a time in later milestones
- Installer/packaging polish — later shipping-form milestone

## Context

Shipped v1.0 (2026-07-14): 7 phases, 35 plans, ~15.9k LOC (TS/TSX/Rust/CSS), 203 tests green, 9-day build (2026-07-06 → 2026-07-14).
Recurring lesson (struck in Phases 3, 4, 5, 6): green unit suites hide wiring bugs — the code-review gate catches criticals every phase; live human UAT catches what jsdom can't (pointer/drag feel).
Known debt at close (STATE.md Deferred Items): Phase 5 Notes has 3 never-run human UAT scenarios; v1.0 milestone audit skipped by user; rail-resize polish deferred; Phase 6 Info review findings open; `/gsd-secure-phase 3` outstanding.

- Prior related work: the Databasise engine project (formerly named Sourcerer) at `D:\Vibe Coding\Databasise` — Python/Cozo/LightRAG, 27-tool MCP + REST, v3.0 shipped. This app is its human GUI in the long-term "one server, two clients" vision.
- User owns runtime config (providers/models/keys); don't interrogate runtime choices.

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Tauri 2 + React 18 + Vite + TS, no component library | Handoff recommendation; bespoke pixel-perfect design | ✓ Good — shipped v1.0 pixel-faithful |
| Notes is the only real applet in v1 | Proves the whole applet-framework loop with minimal scope | ✓ Good — full loop proven incl. live AI summarize |
| Databasise integration mode (sidecar vs external server vs later) | **Deliberately undecided by user** — decide at its own milestone | — Deferred |
| `host.ai()` backend | Originally "stub in v1" — Phase 7 shipped a real headless Pi sidecar instead | ✓ Good — real streamed replies, honest degrade, history persistence |
| Port prototype dock/rail algorithms near-1:1 | Proven sound in the interactive prototype | ⚠ Revised — center dock adopted dockview-core, Home cards dnd-kit; only rail/assistant stayed bespoke (2026-07-06/07 decision) |
| Adopt `bespoke_rails_shell` handoff wholesale | Working prototype around the locked libraries | ✓ Good — Phase 2 reworked Phase 1 chrome to match |
| Cut the 20px floating-stage inset | Halo/grip/click-eating on a real desktop window | ✓ Good — card fills window, 10px radius at true edge |
| Debug exe must build with `--features tauri/custom-protocol` | Plain `cargo build` loads devUrl → "refused to connect" | ✓ Good — documented launch landmine |
| D-P1: AGPL license | Strong copyleft before any public repo / Store submission | — Decided 2026-08-03 |
| D-P2: privileged/docker-api-class store submissions auto-rejected, no appeal in v1 | Minimize triage burden and attack surface | — Decided 2026-08-03 |
| D-P3: Axis-3 blessed (system-flake + profile-tier split) | As recorded in CONTAINER-PLATFORM.md | — Decided 2026-08-03 |
| D-P4: Determinate Nix pin | Installer reliability, flakes default | — Decided 2026-08-03 |
| D-P5: hosted/cloud committed as its own milestone after Platform v1 | Auth-surface interface lands in P2 (stubbed); business track confirmed | — Decided 2026-08-03 |
| Two-host milestone: NixOS dev host + Windows box for WSL legs/UAT | Dev moved to NixOS; WSL substrate premise stands | — Decided 2026-08-03 |
| Phase numbering continues from 8 | GSD default; globally unique across milestones | — Decided 2026-08-03 |
| **Nix-native rule: if it's not in nixpkgs, we package it** | Provenance (auditable source chain vs opaque publisher blob) + store page-sharing at commercial scale (per-tenant microVMs share pages via virtio-fs/DAX only across identical store paths; distinct image userlands never share). NOT permissions (identical either way since the engine drop), NOT disk (amortizes per-image), NOT speed (no difference). | — Decided 2026-08-04. Community catalog is Nix-native by construction; OCI is never a submission format. Sole exception: time-boxed first-party escape hatch (Phase 13 uv2nix), tracked as debt. |
| Phase 8 rescoped: engine-less OCI unit → Nix-native substrate service | `collabora-online` turned out to be in nixpkgs with a full module and cached at 13.8 MB (vs ~1.5–2 GB image), obsoleting the original kill-question | — Decided 2026-08-04. New kill-question: does a self-sandboxing service (coolwsd) survive the §7 baseline under WSL2's kernel? |
| Two-tier AI: hosted default (open-source embeddings, self-hosted own box; cloud generation) + local opt-in (Ollama as one-click catalog app in the substrate) | Zero-setup out of the box; local AI is a product point (privacy/offline), not an optimization. Same pinned embedding model on both tiers so hosted→local switch never re-embeds | — Decided 2026-08-04. Full decision set: `.planning/research/AI-PROVIDER-ARCHITECTURE.md` |
| **Direction pivot: fork Theia IDE as the Sourcerer frontend** — v2.0 Container Platform frozen mid-Phase-9, all bespoke Tauri-shell work superseded and moved to `Sourcerer-v0/` | Building/maintaining a bespoke shell duplicates what a mature extensible workbench platform provides; Theia gives docking, extensions, theming, and an AI framework out of the box. Supersedes the VSCodium exploration verdict (2026-08-17) | — Decided 2026-08-18. Carry-over review (Pi harness, Databasise plans, tokens, Nix flake/CI/cache) deferred to a later phase |

## Constraints

- **Pixel-perfect fidelity** to the bespoke_rails_shell tokens/metrics (40px title bar, green #86A38C accent, IBM Plex type scale, 0 border-radius inside, 1px borders)
- No Google Fonts at runtime — bundle fonts locally
- Applets never bypass the `host` API; `host.ai()` is the only AI seam
- Demo stubs must persist for every unbuilt applet (the "part demo" half is a feature, not debt)

## Evolution

This document evolves at phase transitions and milestone boundaries.

**After each phase transition** (via `/gsd-transition`):
1. Requirements invalidated? → Move to Out of Scope with reason
2. Requirements validated? → Move to Validated with phase reference
3. New requirements emerged? → Add to Active
4. Decisions to log? → Add to Key Decisions
5. "What This Is" still accurate? → Update if drifted

**After each milestone** (via `/gsd-complete-milestone`):
1. Full review of all sections
2. Core Value check — still the right priority?
3. Audit Out of Scope — reasons still valid?
4. Update Context with current state

---
*Last updated: 2026-08-18 — milestone v3.0 Theia Foundation started (v2.0 Container Platform frozen/superseded)*
