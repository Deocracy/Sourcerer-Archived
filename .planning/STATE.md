---
gsd_state_version: 1.0
milestone: v3.0
milestone_name: Theia Foundation
status: planning
last_updated: "2026-08-18"
last_activity: 2026-08-18
progress:
  total_phases: 2
  completed_phases: 0
  total_plans: 0
  completed_plans: 0
  percent: 0
---

# Project State

## Project Reference

See: .planning/PROJECT.md (updated 2026-08-18)

**Core value:** A working, rebranded-in-name Theia IDE fork running locally as the environment Sourcerer (scholar research workbench) is built up in incrementally.
**Current focus:** Phase 1 — Repo Reorganization

## Current Position

Phase: 1 of 2 (Repo Reorganization)
Plan: — of — (not yet planned)
Status: Ready to plan
Last activity: 2026-08-18 — v3.0 roadmap created (2 phases, 5/5 requirements mapped)

Progress: [░░░░░░░░░░] 0%

## Performance Metrics

**Velocity:**
- Total plans completed: 0 (this milestone; v1.0 shipped 35, v2.0 completed 18 before freeze)
- Average duration: — min
- Total execution time: 0 hours

**By Phase:**

| Phase | Plans | Total | Avg/Plan |
|-------|-------|-------|----------|
| - | - | - | - |

**Recent Trend:**
- Last 5 plans: — (new milestone)
- Trend: —

*Updated after each plan completion*

## Accumulated Context

### Decisions

Full log in PROJECT.md Key Decisions. Recent decisions affecting current work:

- 2026-08-18: **Direction pivot** — fork Theia IDE as the Sourcerer frontend; v2.0 Container Platform frozen mid-Phase-9, all bespoke Tauri-shell work superseded into `Sourcerer-v0/`. Supersedes the VSCodium exploration verdict (2026-08-17).
- 2026-08-18: Phase numbering **reset to 1** for this milestone (deliberate, for the pivot). v1.0 Phases 1–7 and v2.0 Phases 8–19 archived under `.planning/milestones/`.
- 2026-08-18: Fork the `theia-ide` **product template**, never the `eclipse-theia/theia` framework monorepo — consume `@theia/*` as versioned npm deps. Getting this wrong collapses the whole value of the pivot.
- 2026-08-18: FORK-02 build approach settled — **Nix devShell build-from-source** (Node 22, Yarn Classic, node-gyp toolchain, nixpkgs Electron). Browser-target dev runs are an acceptable daily-iteration path; the AppImage-wrap pattern in `/etc/nixos/configuration.nix` covers running released binaries only, kept for release artifacts later.
- 2026-08-18: **No theme/chrome change this milestone** — stock Theia UI and theme are acceptable. Branding and the `#86A38C`/IBM Plex identity port are next-milestone work.

### Roadmap Evolution

- v3.0 Theia Foundation roadmapped (2026-08-18): 2 phases, numbering reset to 1. Phase 1 Repo Reorganization (REORG-01/02) gates Phase 2 Theia Fork Build & Run (FORK-01/02/03) — the reorg must land before ~100k files of Theia enter the tree. All 5 v3.0 requirements mapped, no orphans. Research SUMMARY.md suggested 4 phases (Foundation / Branding / Theme / Carry-over); phases 2–4 of that suggestion are out of scope this milestone per the requirements' Deferred and Out-of-Scope sections, so the roadmap covers its Phase 1 only, split across the reorg gate.

### Pending Todos

None yet. **REORG-02 creates the first one** — the deferred carry-over review (Pi assistant harness, Databasise integration plans, design tokens/theme, Nix flake/CI/Attic cache).

### Blockers/Concerns

- [Phase 1]: `flake.nix`/`flake.lock`/`nix/`, `.github/` CI, `docs/`, and `LICENSE` at the repo root are not clearly "Tauri-shell artifacts" and the Nix/CI/Attic stack is itself a carry-over-review topic. Decide per-item at plan time whether each moves to `Sourcerer-v0/`, stays, or is superseded — Phase 2 needs a devShell at the root.
- [Phase 2, research flag]: NixOS + Electron native-module rebuild (`yarn rebuild:electron`) is the one un-tested unknown — no test build has been run on this dev host. Expect sandbox-helper / node-gyp friction; capture whatever workaround is found (DEV-01 formalizes the doc next milestone).
- [Phase 2]: Theia pins Node/Yarn `engines` per release and changes them across versions. Pin the devShell to *this* Theia's declared engines and re-verify at every deliberate Theia bump, not once.
- [Licensing, pre-public]: base layer is EPL-2.0/GPL-2.0-classpath, repo commits to AGPL (D-P1); "Eclipse Theia" name/logo reuse is a separate trademark question. Not blocking local-dev work; needs a dedicated review before anything public.
- [Repo hygiene]: `Deocracy/Sourcerer` is public with 30 open Dependabot alerts (13 high, 17 medium) surfaced 2026-08-04. Untouched; the Phase 1 reorg is a natural moment to reassess since much of the alerting dependency tree moves to `Sourcerer-v0/`.

## Deferred Items

Items acknowledged and carried forward. v1.0/v2.0 entries remain open but are frozen with their milestones — see `.planning/milestones/` for full context.

| Category | Item | Status | Deferred At |
|----------|------|--------|-------------|
| Carry-over | Pi harness / Databasise plans / design tokens / Nix flake+CI+Attic — what migrates to Theia | Deferred by decision; REORG-02 records it | v3.0 start 2026-08-18 |
| Branding | BRAND-01..04 (app name, icons, About/Welcome, default layout) | Deferred to next milestone | v3.0 start 2026-08-18 |
| Theme | THEME-01 identity port (`#86A38C` + IBM Plex) | Deferred to next milestone | v3.0 start 2026-08-18 |
| Dev workflow | DEV-01 documented repeatable build/run/rebuild doc | Deferred to next milestone | v3.0 start 2026-08-18 |
| Milestone | v2.0 Container Platform frozen mid-Phase-9, superseded by the pivot | Frozen | 2026-08-18 |
| uat_gap | v1.0 Phase 05 Notes: 3 pending human UAT scenarios | Open — frozen with v1.0 | v1.0 close 2026-07-14 |
| audit | v1.0 milestone audit never run (`/gsd-audit-milestone`) | Skipped by user decision | v1.0 close 2026-07-14 |
| review | v1.0 Phase 06 Info findings IN-01..IN-07 | Open, low severity — frozen | v1.0 close 2026-07-14 |
| security | `/gsd-secure-phase 3` outstanding | Open — frozen with v1.0 | v1.0 close 2026-07-14 |
| infra | Attic cache private for reads; `Deocracy/nixos-hosting` fix/attic-api-endpoint unmerged | Open — frozen with v2.0 | v2.0 freeze 2026-08-18 |

## Session Continuity

Last session: 2026-08-18
Stopped at: v3.0 ROADMAP.md created (2 phases), REQUIREMENTS.md traceability filled
Resume file: None

## Operator Next Steps

- Review `.planning/ROADMAP.md`, then `/gsd-plan-phase 1` to plan the repo reorganization
- Phase 1 planning must settle the root-file question (flake/CI/docs/LICENSE stay or move) before any `git mv` runs
