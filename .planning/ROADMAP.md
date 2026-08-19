# Roadmap: Sourcerer — v3.0 Theia Foundation

## Overview

v3.0 replaces the bespoke Tauri shell with a fork of Eclipse Theia IDE as the Sourcerer frontend. The journey is short and sequential: first clear the tree — freeze every superseded Tauri-shell artifact under `Sourcerer-v0/` with history intact and record the deferred carry-over review — then land the `eclipse-theia/theia-ide` product template as a Sourcerer-owned fork that builds from source in a Nix devShell and launches on the NixOS dev host. The milestone ends with a stock-looking but genuinely working Theia environment that future milestones extend. No branding, no theme, no applet work — those are deliberately deferred.

**Phase numbering reset:** v3.0 restarts at Phase 1 by user decision for this direction pivot. Prior milestones' phases are archived under `.planning/milestones/` (v1.0 Phases 1–7, v2.0 Phases 8–19 frozen mid-Phase-9).

## Phases

**Phase Numbering:**
- Integer phases (1, 2): Planned milestone work
- Decimal phases (1.1, 1.2): Urgent insertions (marked with INSERTED)

Decimal phases appear between their surrounding integers in numeric order.

- [ ] **Phase 1: Repo Reorganization** - Freeze superseded Tauri-shell work under `Sourcerer-v0/` with history preserved and record the deferred carry-over review
- [ ] **Phase 2: Theia Fork Build & Run** - Fork `theia-ide`, build it from source in a Nix devShell, and launch it as the working dev environment

## Phase Details

### Phase 1: Repo Reorganization
**Goal**: The repo tree is cleared for the Theia direction — every superseded Tauri-shell artifact frozen under `Sourcerer-v0/` with git history intact, and the deferred carry-over review recorded so it surfaces at the next milestone
**Depends on**: Nothing (first phase)
**Requirements**: REORG-01, REORG-02
**Success Criteria** (what must be TRUE):
  1. Superseded Tauri-shell artifacts (`src/`, `src-tauri/`, `sidecar/`, `design-sync-setup-guide/`, the Vite/TS/vitest configs, and the VSCodium exploration folder) live under `Sourcerer-v0/`, and `git log --follow` on a moved file still shows its pre-move history
  2. `.planning/` remains at the repo root and GSD commands (`/gsd-progress`, roadmap/state reads) resolve with no path fixes
  3. The repo root shows only what the Theia direction needs — no stale Tauri app source or build config at top level, and the tree is a clean landing site for a fork
  4. A pending todo exists under `.planning/todos/pending/` naming all four carry-over review topics (Pi assistant harness, Databasise integration plans, design tokens/theme, Nix flake/CI/Attic cache) and surfaces in `/gsd-progress`
**Plans**: TBD

**Note (settle at plan time, do not pre-decide here):** `flake.nix`/`flake.lock`/`nix/`, `.github/` CI, `docs/`, and `LICENSE` are *not* obviously Tauri-shell artifacts — the Nix/CI/Attic-cache stack is itself a listed carry-over-review topic (REORG-02) and Phase 2 needs a devShell at the root. Decide per-item during planning whether each moves, stays, or is superseded; record the call so the carry-over todo isn't silently pre-answered.

### Phase 2: Theia Fork Build & Run
**Goal**: A Sourcerer-owned fork of the `eclipse-theia/theia-ide` product template builds from source and launches on the NixOS dev host as the working environment future milestones extend
**Depends on**: Phase 1
**Requirements**: FORK-01, FORK-02, FORK-03
**Success Criteria** (what must be TRUE):
  1. The `eclipse-theia/theia-ide` product template (1.74 line) is present in the repo with an upstream remote configured, so `git fetch upstream` surfaces future Theia releases as mergeable rather than as a permanent divergence
  2. Entering the Nix devShell provides Node 22, Yarn Classic, the node-gyp toolchain, and nixpkgs Electron, and a clean `yarn` + build completes entirely from that shell with no host-global toolchain and no network-fetched Node/Electron
  3. The forked app launches on the dev host and is usable as an environment — stock Theia workbench with editor, file explorer, and terminal all responding; opening and closing the app is clean
  4. A source edit can be rebuilt and observed in a re-run through the browser-target dev loop, proving daily iteration works without a full Electron repackage
**Plans**: TBD

## Progress

**Execution Order:**
Phases execute in numeric order: 1 → 2

| Phase | Plans Complete | Status | Completed |
|-------|----------------|--------|-----------|
| 1. Repo Reorganization | 0/TBD | Not started | - |
| 2. Theia Fork Build & Run | 0/TBD | Not started | - |

## Explicitly Not In This Milestone

Recorded here so phase planning does not drift into them. Full list in REQUIREMENTS.md.

- Branding (BRAND-01..04): app name, icons, About/Welcome rebrand, default layout — next milestone
- Theme identity port (THEME-01): green `#86A38C` + IBM Plex — next milestone
- Documented dev workflow (DEV-01) — next milestone; Phase 2 proves the loop works, it does not have to write the doc
- Carry-over review (CARRY-01/02): Pi harness, Databasise, `host.*` → Theia DI mapping — deferred by decision, tracked by REORG-02's todo
- Forking the `eclipse-theia/theia` framework monorepo — anti-pattern; consume `@theia/*` as npm deps
- Any theme/chrome change, feature-stripping, public release/packaging/licensing resolution
