# Phase 9: Flake Foundation & Assurance Chain - Pattern Map

**Mapped:** 2026-08-04
**Files analyzed:** 16 (create: 12, modify: 3, generated/not-authored: 1)
**Analogs found:** 6 / 16 (10 are greenfield — no in-repo analog exists or should be invented;
this is expected and stated explicitly per file, per the phase's own "nothing Nix-shaped
exists yet" framing)

## Scope Note

This phase is infrastructure/tooling (Nix flake + CI + license + version pins), not
application code. There are no controllers/components/services/models in the GSD sense.
"Role" below is reinterpreted for this phase's artifact kinds: **flake-module**, **CI-config**,
**pin-file**, **license-doc**, **runbook-doc**, **package-manifest** (existing file, field
edit only). Do not search for React/Rust-application analogs — none exist for a flake/CI
tree and none should be reported. The repo has zero `.nix` files, zero `.github/`, zero
`LICENSE`, zero `rust-toolchain.toml`, zero Node-version pin file, and no git remote today
[VERIFIED: `git remote -v` empty, `find` for `rust-toolchain*`/`.nvmrc` empty, `ls .github`
empty, `ls LICENSE*` empty — 2026-08-04]. Everything this phase ships is new except three
one-field edits to already-tracked files.

## File Classification

| New/Modified File | Role | Data Flow | Closest Analog | Match Quality |
|---|---|---|---|---|
| `flake.nix` | flake-module (config) | transform (declarative eval → devShells/packages/checks) | none | no analog — greenfield |
| `flake.lock` | generated artifact (not hand-authored) | — | none | n/a — produced by `nix flake update`, never edited by hand |
| `rust-toolchain.toml` | pin-file | transform (read by rustup + rust-overlay) | `src-tauri/Cargo.toml` (existing Rust pin surface, same tree) | role-match only — Cargo.toml pins library versions, not toolchain; no toolchain-file analog exists |
| `.nvmrc` (Node pin, Claude's discretion on exact filename) | pin-file | transform (read by flake + `actions/setup-node`) | `package.json` `engines`/version fields (absent today — see below) | no direct analog; `package.json` has no `engines` block to mirror |
| `nix/substrate/core.nix` | flake-module (NixOS module) | event-driven (systemd services, declarative) | none | no analog — greenfield |
| `nix/substrate/wsl-variant.nix` | flake-module (NixOS module) | transform (adapter composition) | `nix/substrate/core.nix` (sibling, same phase) | no cross-domain analog; internally consistent with core.nix's shape only |
| `nix/substrate/vm-variant.nix` | flake-module (NixOS module) | transform (adapter composition) | `nix/substrate/core.nix` (sibling, same phase) | no cross-domain analog |
| `nix/checks/seed-boot-test.nix` | test (nixosTest) | event-driven (VM boot → assert HTTP) | `vitest.config.ts` — weak, cross-domain only (both are "test config separate from source", nothing else transfers) | no real analog — different test framework/domain entirely |
| `nix/lib.nix` | flake-module (public surface, D-09) | transform (near-empty re-export) | none | no analog — greenfield, deliberately near-empty |
| `LICENSE` | license-doc | — | none | no analog — repo has zero license file today |
| `.github/workflows/*.yml` (CI, layout is Claude's discretion) | CI-config | request-response / batch (push/PR trigger → job matrix) | `package.json` scripts block (lines 6-13) — the CI job's actual invocation surface | role-match — CI has no in-repo CI analog, but its commands ARE the existing npm scripts |
| `package.json` (MODIFY — add `license` field) | package-manifest | CRUD (single field add) | itself | exact — edit in place, existing file |
| `src-tauri/Cargo.toml` (MODIFY — add `license` field) | package-manifest | CRUD (single field add) | itself | exact — edit in place, existing file |
| `docs/RUNBOOK.md` or similar (FOUND-03 channel-maintenance runbook; exact path is Claude's discretion) | runbook-doc | — | `README.md` (doc tone/heading conventions, same repo) | role-match — no runbook exists, but README.md sets the doc-writing register to match |
| `.gitignore` (MODIFY — append Nix/direnv entries) | pin-file (ignore rules) | CRUD (append) | itself | exact — existing file, same append-a-block convention already used for "Rust / Tauri build artifacts" |
| `.envrc` (direnv wiring, Claude's discretion per D-16) | pin-file | — | none | no analog — greenfield |

## No Analog Found

| File | Role | Data Flow | Reason |
|---|---|---|---|
| `flake.nix`, `nix/substrate/core.nix`, `nix/substrate/wsl-variant.nix`, `nix/substrate/vm-variant.nix`, `nix/checks/seed-boot-test.nix`, `nix/lib.nix`, `LICENSE`, `.envrc` | flake-module / license-doc / pin-file | various | Repo has zero `.nix` files and zero license file. Do not invent a fake analog — RESEARCH.md's `## Architecture Patterns` (Pattern 1-5) and `## Code Examples` sections are the correct reference source for these; they are already-verified-against-official-docs skeletons, not something to re-derive from this repo's application code. |
| `flake.lock` | generated | — | Never hand-authored; produced by `nix flake update` after `flake.nix`'s inputs block is written. No pattern to copy — just run the command. |
| `rust-toolchain.toml`, `.nvmrc` | pin-file | transform | No existing pin file of either kind in this repo (confirmed via `find`). Content shape comes from RESEARCH.md Pattern 3 (`rust-toolchain.toml`) and the `actions/setup-node` `node-version-file` convention, not from an in-repo analog. |

---

## Pattern Assignments

### `.github/workflows/*.yml` (CI-config, request-response/batch)

**Analog:** `package.json` scripts block — this is what the `windows-latest` CI job must
invoke; there is no existing CI file to copy structurally, but the exact npm command surface
already exists and must not be reinvented or paraphrased.

**Existing scripts to invoke verbatim** (`package.json` lines 6-13):
```json
"scripts": {
  "dev": "vite",
  "build": "tsc && vite build",
  "preview": "vite preview",
  "tauri": "tauri",
  "test": "vitest",
  "verify:fonts": "npm run build && node scripts/verify-fonts.mjs"
}
```
Concrete implications for the `windows-latest` job:
- `npm test` is bare vitest = **watch mode** (per README.md's own warning) — CI must run
  `npx vitest run`, not `npm test`, or the job hangs. This is a real landmine, not a
  hypothetical — the project's own README flags it.
- `npm run verify:fonts` already runs `npm run build` internally (chained), so if the CI job
  also runs `npm run build` separately for `tauri build`'s `beforeBuildCommand` (see below),
  don't redundantly re-invoke `verify:fonts`'s own build step — sequence it once.
- `npm run tauri build` is the actual Tauri packaging invocation (not in the scripts block
  today as its own alias — `"tauri": "tauri"` is the passthrough; CI calls `npm run tauri
  build` or `npx tauri build`).

**Build-hook surface the CI job exercises** (`src-tauri/tauri.conf.json` lines 6-11 —
**read-only reference, do NOT propose changes to this file**):
```json
"build": {
  "beforeDevCommand": "npm run dev",
  "devUrl": "http://localhost:1420",
  "beforeBuildCommand": "npm run build",
  "frontendDist": "../dist"
}
```
`npm run tauri build` on `windows-latest` will itself shell out to `npm run build`
(`beforeBuildCommand`) before bundling — the CI job does not need a separate explicit
`npm run build` step ahead of `tauri build` unless `verify:fonts` (which needs `dist/`) must
run first and be kept as a distinct, individually-reportable CI step. Recommended ordering:
`npm ci` → `npx vitest run` → `npm run verify:fonts` (builds `dist/` + checks it) →
`npm run tauri build` (rebuilds `dist/` again via its own `beforeBuildCommand` — acceptable
redundancy, keeps `verify:fonts` a clean red/green step independent of Tauri packaging).

**Node/Rust versions confirmed on the dev host** [VERIFIED via README.md + this session's env,
2026-08-04]: Node v24.18.0, npm 11.16.0; Rust has **no default toolchain configured** on this
Linux dev host today (`rustup default stable` not yet run) — this is exactly the "both worlds
floating" gap D-10 closes, confirming RESEARCH.md's framing is accurate, not stale.

---

### `rust-toolchain.toml` (pin-file, transform)

**No in-repo analog** — content comes from RESEARCH.md Pattern 3, not this repo. The one
concrete fact to pull FROM this repo: `src-tauri/Cargo.toml` dependencies block (lines 20-27)
pins `tauri = { version = "2", features = [] }` — the planner must resolve Tauri 2.x's actual
current MSRV against this exact dependency set before hardcoding a `channel` value (RESEARCH.md
Assumption A2 flags this explicitly — don't guess a Rust version number).

```toml
# src-tauri/Cargo.toml (existing, lines 1-27 — read-only reference)
[package]
name = "sourcerer"
version = "0.1.0"
description = "A Tauri App"
authors = ["you"]
edition = "2021"
...
[dependencies]
tauri = { version = "2", features = [] }
tauri-plugin-opener = "2"
serde = { version = "1", features = ["derive"] }
serde_json = "1"
tokio = { version = "1", features = ["sync", "time"] }
tauri-plugin-store = "2.4.3"
```

---

### `package.json` (package-manifest, CRUD — add `license` field)

**Analog:** itself. Current file has no top-level `license` key (full file below, 41 lines,
read in one pass — nothing omitted):
```json
{
  "name": "sourcerer",
  "private": true,
  "version": "0.1.0",
  "type": "module",
  ...
}
```
D-05 (PolyForm Noncommercial 1.0.0) requires adding `"license": "..."` — note PolyForm NC is
**not a standard SPDX identifier** recognized by npm's license-field validator the way `MIT`
or `AGPL-3.0` are; the conventional approach is `"license": "SEE LICENSE IN LICENSE"` (npm's
own documented convention for non-SPDX licenses) rather than inventing a nonstandard SPDX-like
string. Insert as a new key near `"private": true` (line 3), matching the existing flat
top-level key ordering.

---

### `src-tauri/Cargo.toml` (package-manifest, CRUD — add `license` field)

**Analog:** itself. Current `[package]` block (lines 1-6) has no `license` key:
```toml
[package]
name = "sourcerer"
version = "0.1.0"
description = "A Tauri App"
authors = ["you"]
edition = "2021"
```
Cargo's `license` field DOES expect a valid SPDX expression normally, but also accepts
`license-file = "../LICENSE"` (a path, relative to `Cargo.toml`) as the correct mechanism for
a non-SPDX license like PolyForm NC — use `license-file`, not a fabricated `license` string,
per Cargo's own documented escape hatch for exactly this case.

---

### `.gitignore` (pin-file, CRUD append)

**Analog:** itself — existing append-a-labeled-block convention, full file (35 lines) already
read in one pass:
```gitignore
# Rust / Tauri build artifacts
# Bare pattern: matches at any depth, so spike Rust builds are covered too.
target/
src-tauri/gen/schemas

...

# Large downloaded images (spike 010's nixos.wsl was protected only by
# happening to live under dist/)
*.wsl
*.vhdx
```
Every existing block is: a `#`-comment header naming the concern, one-line rationale comment
where non-obvious, then the bare patterns. New Nix/direnv entries (`result`, `result-*`,
`.direnv/`) should follow this exact block shape — one new labeled section, not scattered
loose lines. Note `*.wsl`/`*.vhdx` are already ignored (relevant if the substrate image build
output ever lands in the repo tree during local dev shell testing — it's already covered).

---

### `.github/workflows/*.yml` drift-gate + doc register (runbook-doc)

**Analog:** `README.md` (full file skimmed, lines 1-40 shown) — sets the doc tone: short
declarative sentences, a "Verified on this host: ..." provenance line pattern, explicit
"There is no X yet; Y is the intended fix" framing for known gaps.
```markdown
## Prerequisites

Verified on this host: **Node v24.18.0**, npm 11.16.0.

The Rust/Tauri half does **not** currently build on this Linux machine. It needs:
...
There is no `flake.nix` yet; a repo-root dev shell is a v2.0 P1 deliverable, which
is the intended fix for the above rather than imperative installs.
```
The FOUND-03 runbook (nixpkgs bump cadence, red-channel response, retention policy,
restore-from-scratch) should match this register: concrete, host-provenance-stamped, and
explicit about what doesn't exist yet rather than aspirational. This README passage is also
now **stale** once `flake.nix` exists — flag that the runbook/README update should correct
this specific "There is no flake.nix yet" line as part of Phase 9's own work (it becomes false
the moment `flake.nix` lands).

---

## Shared Patterns

### npm script invocation surface (CI + verify chain)
**Source:** `package.json` lines 6-13
**Apply to:** `.github/workflows/*.yml` (windows-latest job)
```json
"build": "tsc && vite build",
"test": "vitest",
"verify:fonts": "npm run build && node scripts/verify-fonts.mjs"
```
Use `npx vitest run` (not `npm test`, which is watch-mode) and sequence `verify:fonts` before
or instead of a redundant standalone `build` step — see full reasoning under the CI-config
pattern assignment above.

### Cross-platform Node script style (for any new `.mjs` helper this phase might add, e.g. a
drift-gate helper script if the planner chooses a Node script over inline shell)
**Source:** `scripts/verify-fonts.mjs` (full file, 53 lines, read in one pass)
```javascript
import { readdirSync, readFileSync, statSync } from "node:fs";
import { join } from "node:path";
...
console.error(`verify:fonts — FAIL: ...`);
process.exit(1);
...
console.log(`verify:fonts — PASS: ...`);
```
Conventions to reuse if the planner writes any new Node-based CI helper: ESM `node:`-prefixed
core imports, a labeled `console.error`/`process.exit(1)` failure convention (prefix every
message with the script's own name, e.g. `drift-gate — FAIL: ...`), plain `console.log` PASS
summary on success, no external dependencies. This is explicitly why the script is
cross-platform (works identically on Windows CI and the NixOS dev host) — the same reasoning
applies to any new pin-drift helper.

### `tauri.conf.json` build hooks (read-only reference — do not modify)
**Source:** `src-tauri/tauri.conf.json` lines 6-11
**Apply to:** understanding what `npm run tauri build` triggers in CI; do NOT edit this file
this phase (CSP and other `tauri.conf.json` changes are explicitly Phase 10's concern per
CONTEXT.md `<code_context>`).
```json
"build": {
  "beforeDevCommand": "npm run dev",
  "devUrl": "http://localhost:1420",
  "beforeBuildCommand": "npm run build",
  "frontendDist": "../dist"
}
```

### Labeled-block append convention (`.gitignore`-style)
**Source:** `.gitignore` (full file)
**Apply to:** `.gitignore` (this phase's own Nix/direnv additions) and, by extension, any
other append-only config file this phase touches
Every addition is a `#`-header block with an optional one-line rationale comment, never a bare
unlabeled pattern dropped at the end of the file.

---

## Metadata

**Analog search scope:** repo root (`package.json`, `package-lock.json`, `vitest.config.ts`,
`.gitignore`, `.gitattributes`, `README.md`), `scripts/` (`verify-fonts.mjs`),
`src-tauri/` (`Cargo.toml`, `Cargo.lock`, `tauri.conf.json`). Confirmed absent by direct
listing/`find`: any `.nix` file, `.github/`, `LICENSE*`, `rust-toolchain*`, `.nvmrc`, git
remote. Also referenced `.planning/phases/08-.../08-PATTERNS.md` to confirm the correct
"no analog, say so explicitly" convention this pattern map follows for a greenfield-heavy
phase (that phase's spike scope differs, but its honesty convention transfers).
**Files scanned:** `package.json` (full, 41 lines), `vitest.config.ts` (full, 16 lines),
`scripts/verify-fonts.mjs` (full, 53 lines), `src-tauri/tauri.conf.json` (full, 39 lines),
`src-tauri/Cargo.toml` (full, 27 lines), `.gitignore` (full, 35 lines), `.gitattributes`
(full, 11 lines), `README.md` (lines 1-40 of ~60), `package-lock.json` (lockfileVersion only,
grep), `src-tauri/Cargo.lock` (header + first entries, confirms `version = 4` lockfile format
for crane).
**Pattern extraction date:** 2026-08-04
