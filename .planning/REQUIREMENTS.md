# Requirements: Sourcerer — Milestone v3.0 Theia Foundation

**Defined:** 2026-08-18
**Core Value:** A working, rebranded-in-name Theia IDE fork running locally as the environment Sourcerer (scholar research workbench) is built up in incrementally.

## v3.0 Requirements

Requirements for this milestone. Each maps to roadmap phases.

### Reorganization

- [ ] **REORG-01**: Superseded Tauri-shell work (app source, `src-tauri`, sidecar, design handoffs, and other files not needed by the Theia direction) is moved into a `Sourcerer-v0/` folder via `git mv`, preserving history; `.planning/` stays at the repo root
- [ ] **REORG-02**: A pending todo records the deferred carry-over review (Pi assistant harness, Databasise integration plans, design tokens/theme, Nix flake/CI/Attic cache) so it surfaces at the next milestone

### Fork

- [ ] **FORK-01**: The `eclipse-theia/theia-ide` product template (current 1.74 line, Electron target) is forked and present in the repo, tracking upstream so future merges stay possible
- [ ] **FORK-02**: The fork builds from source on the NixOS dev host via a Nix devShell (Node 22, Yarn Classic, node-gyp toolchain, nixpkgs Electron). Browser-target dev runs are an acceptable daily-iteration path; the existing AppImage-wrap pattern in `/etc/nixos/configuration.nix` covers only *running* released binaries, not building, and is kept for release artifacts later
- [ ] **FORK-03**: The forked app launches and runs on the dev host as a working environment — stock Theia UI and stock theme are acceptable; this is the base future milestones extend

## Deferred Requirements (next milestone — the "add after MVP" list)

Tracked but not in the v3.0 roadmap. Branding items are the todo list requested for a later plan.

### Branding (Product Extension pattern — one `sourcerer-product` Theia extension)

- **BRAND-01**: `applicationName: "Sourcerer"` set — window title and app metadata carry the product name
- **BRAND-02**: App icons replaced on all platforms (`.ico`/`.icns`/`.png` sets under `applications/electron/resources/`)
- **BRAND-03**: About dialog and Getting Started/Welcome page rebranded via DI `rebind()` — no stock Eclipse/Theia branding visible on first launch
- **BRAND-04**: Coherent default first-launch layout (which panels open cold) — need not resemble the old bespoke shell

### Dev workflow

- **DEV-01**: Documented, repeatable build/run/rebuild workflow (the Theia equivalent of the v1.0 `tauri dev` landmine doc)

### Theme

- **THEME-01**: Identity port — green `#86A38C` accent + IBM Plex via Theia's theme system (identity only, not pixel fidelity)

### Carry-over review

- **CARRY-01**: Deliberate review of what migrates from v1.0/v2.0: Pi assistant harness → Theia AI framework, Databasise integration plans, design tokens, Nix flake/CI/Attic cache
- **CARRY-02**: `host.storage`/`host.ai()`/`host.open()` → Theia DI/preference/AI-framework mapping decided; applets as native Theia extensions

## Out of Scope

Explicitly excluded. Documented to prevent scope creep.

| Feature | Reason |
|---------|--------|
| Forking the `eclipse-theia/theia` platform monorepo | Community-documented anti-pattern; consume `@theia/*` as npm deps via the theia-ide template |
| Any theme/chrome change this milestone | User decision 2026-08-18: stock Theia theme, no touching theme for now |
| Pixel-perfect port of the bespoke_rails_shell tokens (40px bar, 0-radius) | Fights Theia's Lumino shell geometry; identity port (THEME-01) deferred instead |
| Theia AI wiring / Pi assistant port | Deferred to carry-over review; different enough architecture that a rushed port risks doing it wrong twice |
| Feature-stripping (debugger, Git, terminal removal) | Hide-don't-delete; strip deliberately only after the build is proven reproducible |
| Public release / packaging / licensing resolution | Local-dev milestone; EPL-2.0 vs AGPL (D-P1) check required before anything public |

## Traceability

Which phases cover which requirements. Updated during roadmap creation.

| Requirement | Phase | Status |
|-------------|-------|--------|
| REORG-01 | — | Pending |
| REORG-02 | — | Pending |
| FORK-01 | — | Pending |
| FORK-02 | — | Pending |
| FORK-03 | — | Pending |

**Coverage:**
- v3.0 requirements: 5 total
- Mapped to phases: 0 (roadmap pending)
- Unmapped: 5 ⚠️

---
*Requirements defined: 2026-08-18*
*Last updated: 2026-08-18 after initial definition*
