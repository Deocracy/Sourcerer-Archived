---
phase: 09-flake-foundation-assurance-chain
plan: 02
subsystem: infra
tags: [nix, flake, rust-overlay, rust-toolchain, devshell, tauri, direnv]

# Dependency graph
requires: ["09-01 (LICENSE, manifest license fields, Nix/direnv .gitignore block)"]
provides:
  - "rust-toolchain.toml (single Rust pin, channel 1.97.1, read by rustup + rust-overlay)"
  - ".nvmrc (single Node pin, 24.18.0, read by the flake + actions/setup-node)"
  - "flake.nix (devShells.x86_64-linux.default, lib, overlays.default, pinned nixpkgs)"
  - "flake.lock (nixpkgs + rust-overlay locked to explicit revisions)"
  - "nix/lib.nix (D-09 public surface: pinnedNixpkgs, pkgsFor, appModules)"
  - ".envrc (direnv auto-entry, watches both pin files)"
  - "README.md Prerequisites corrected to the flake path"
affects: ["09-03", "09-04", "09-05", "09-06", "09-07"]

# Tech tracking
tech-stack:
  added: ["rust-overlay (oxalica)", "nixpkgs nixos-26.05 pin"]
  patterns:
    - "Single-system flake (x86_64-linux only), no flake-utils for one target"
    - "rust-overlay fromRustupToolchainFile reads the same rust-toolchain.toml rustup reads natively"
    - "flake.nix description must be a literal string, not a computed/concatenated one — Nix's lightweight flake-metadata parse forces it before full evaluation"

key-files:
  created: [rust-toolchain.toml, .nvmrc, flake.nix, flake.lock, nix/lib.nix, .envrc]
  modified: [README.md]

key-decisions:
  - "flake.nix description field uses a single string literal, not string concatenation (+) — nix flake metadata's fast-path parser fails on a computed description with an opaque 'expected a string but got a thunk' error"
  - "shellHook strips mkShell's fabricated $out=\"$PWD/outputs/out\" rpath entry from NIX_LDFLAGS — this repo's checkout path contains a space (\"Vibe Coding\"), which breaks the cc-wrapper's word-splitting of that rpath flag and fails every single cargo build-script link step, unrelated to any project dependency"

patterns-established:
  - "Dev-shell version proof: shellHook prints resolved rustc/node versions on entry, making version drift self-evidencing without a separate check"

requirements-completed: []

# Metrics
duration: 20min
completed: 2026-08-04
---

# Phase 9 Plan 2: Version Pins and Repo-Root Flake Summary

**Authored the repo-root flake.nix pinning nixpkgs nixos-26.05 with a single devShell that builds the Tauri app end to end (proved via a live `cargo build`), closing the standing gap where the Rust/Tauri half didn't build on this NixOS dev host.**

## Performance

- **Duration:** ~20 min
- **Started:** 2026-08-04T18:27:30Z
- **Completed:** 2026-08-04T18:47:51Z
- **Tasks:** 3
- **Files modified:** 7 (6 created, 1 modified)

## Accomplishments

- `rust-toolchain.toml` pins Rust to the exact channel `1.97.1` (satisfies Tauri 2.11.5's MSRV 1.77.2) with `rustfmt`/`clippy` components and a minimal profile — no floating channel name
- `.nvmrc` pins Node to `24.18.0`, matching both the dev host's installed Node and nixpkgs `nixos-26.05`'s `nodejs_24` package exactly (zero drift)
- `flake.nix` at the repo root: nixpkgs locked to `nixos-26.05`, `rust-overlay` (`follows = "nixpkgs"` for one true nixpkgs), `devShells.x86_64-linux.default` carrying the Rust toolchain (via `fromRustupToolchainFile`), `nodejs_24`, `cargo-tauri`, `attic-client`, `awscli2`, `nixos-rebuild`, `git`, and the Tauri Linux build/native-build inputs (`webkitgtk_4_1`, `libsoup_3`, `gtk3`, `librsvg`, `glib-networking`, `openssl`, `pkg-config`, `wrapGAppsHook4`)
- `nix/lib.nix` (D-09, 17 lines): `pinnedNixpkgs` re-export, `pkgsFor` entry point, and the deliberately empty `appModules` slot
- `overlays.default` — named, empty, commented as the Phase 15/16 fill point
- `flake.lock` committed, locking `nixpkgs` and `rust-overlay` to explicit revisions
- `.envrc` wires `use flake` plus `watch_file` on both pin files
- **Proved the shell builds the real app**, not just evaluates: inside `nix develop`, `npm ci`, `npm run build`, and `cargo build --manifest-path src-tauri/Cargo.toml` all succeed — the exact webkitgtk/libsoup linkage the README previously said was broken now links and finishes in ~56s
- `README.md`'s Prerequisites section rewritten to describe `nix develop` + `direnv allow` instead of the imperative gap it replaced; the "There is no flake.nix yet" and "rustup default stable" lines are gone; the `## Commands` section (including the `npx vitest run` watch-mode warning) is untouched

## Task Commits

1. **Task 1: Create the two native version pin files** - `446cee5` (feat)
2. **Task 2: Author flake.nix, nix/lib.nix, .envrc, and lock the inputs** - `279f6ab` (feat)
3. **Task 3: Prove the dev shell builds the Tauri app, and correct the README** - `3daab68` (fix, includes the shellHook rpath fix)

## Files Created/Modified

- `rust-toolchain.toml` - `[toolchain]` table, `channel = "1.97.1"`, `components = ["rustfmt", "clippy"]`, `profile = "minimal"`
- `.nvmrc` - exactly `24.18.0\n`
- `flake.nix` - inputs (`nixpkgs`, `rust-overlay`), `devShells.x86_64-linux.default`, `lib`, `overlays.default`
- `flake.lock` - generated by `nix flake lock`, locks both inputs to explicit revisions
- `nix/lib.nix` - D-09 public surface (17 lines, under the 40-line budget)
- `.envrc` - `use flake` + two `watch_file` lines
- `README.md` - Prerequisites section rewritten; `## Commands` and everything else unchanged

## Decisions Made

- **`description` must be a plain string literal, not a computed one.** The first `flake.nix` draft built the `description` field via string concatenation (`"a" + "b"`) for readability across two lines. `nix flake lock`/`nix flake metadata` failed with an opaque `error: expected a string but got a thunk at flake.nix:2:3` — Nix's flake-schema pre-check reads `description` via a lightweight parse pass before full evaluation and requires it to already be a forced string, not a thunk. Reproduced in isolation against a two-line minimal flake to confirm the cause before fixing. Fixed by using one single-line string literal.
- **Stripped a fabricated rpath entry from `NIX_LDFLAGS` in `shellHook`.** `pkgs.mkShell` sets `$out = "$PWD/outputs/out"` for interactive shells (there is no real derivation output to point at), and derives `-rpath $out/lib` into `NIX_LDFLAGS`. This repo's checkout path (`/home/chris/Vibe Coding/Sourcerer`) contains a space, so that rpath flag gets silently word-split by nixpkgs' cc-wrapper — `ld` then tries to open a bogus relative path (`Coding/Sourcerer/outputs/out/lib`) and **every single cargo build-script link step fails**, regardless of the crate (confirmed against `quote`, `serde_core`, `libc`, `thiserror`, `parking_lot_core`, and others — none of which need the fabricated rpath at all). This is a known-class nixpkgs `mkShell`-in-a-space-path landmine, not a project dependency problem. Fixed by having `shellHook` strip that one `-rpath $PWD/outputs/out/lib` entry from `NIX_LDFLAGS` before any build runs, using `$PWD` (not a hardcoded path) so the fix travels with the checkout location.

## Deviations from Plan

### Auto-fixed Issues

**1. [Rule 3 - Blocking issue] `mkShell`'s fabricated `$out` rpath breaks every cargo build-script link on a space-containing checkout path**
- **Found during:** Task 3 (proving `cargo build --manifest-path src-tauri/Cargo.toml` inside the dev shell)
- **Issue:** `NIX_LDFLAGS` contained `-rpath /home/chris/Vibe Coding/Sourcerer/outputs/out/lib` (mkShell's interactive-shell stand-in for a real derivation's own `$out`). The embedded space in this repo's checkout path caused nixpkgs' cc-wrapper to word-split that flag, producing a bogus relative link-search path and failing the link step of every crate's build script — a total, dependency-independent blocker for Task 3's whole verification.
- **Fix:** Added one line to `flake.nix`'s `shellHook` that strips `-rpath $PWD/outputs/out/lib` from `NIX_LDFLAGS` before printing the version banner, using `$PWD` (not a hardcoded path) so it self-corrects for any checkout location.
- **Files modified:** flake.nix
- **Verification:** Re-ran `nix flake check --no-build` (still green) and `cargo build --manifest-path src-tauri/Cargo.toml` inside `nix develop` after a clean `rm -rf src-tauri/target` — full workspace compiled successfully in ~56s, including `tauri v2.11.5`, `webkit2gtk v2.0.2`, `wry v0.55.1`.
- **Committed in:** `3daab68` (Task 3 commit)

**2. [Rule 1 - Bug] `flake.nix` description field used string concatenation, which Nix's flake-metadata parser cannot force**
- **Found during:** Task 2 (first `nix flake lock` attempt)
- **Issue:** `description = "..." + "...";` produced `error: expected a string but got a thunk at flake.nix:2:3` on every flake command (`nix flake lock`, `nix flake metadata`, `nix flake check`) — Nix reads `description` via a fast, pre-evaluation parse that requires a literal string, not a computed expression.
- **Fix:** Collapsed the two-line concatenated string into one single-line string literal with identical content.
- **Files modified:** flake.nix
- **Verification:** `nix flake lock` and `nix flake metadata` both succeeded afterward, showing the correct description text.
- **Committed in:** `279f6ab` (Task 2 commit)

---

**Total deviations:** 2 auto-fixed (1 Rule 1 - Nix flake-metadata parsing constraint on `description`; 1 Rule 3 - blocking nixpkgs `mkShell`-in-a-space-path landmine). Both fixes stay within `flake.nix`, satisfy every stated acceptance criterion, and are load-bearing for Task 3's actual proof (a real `cargo build` succeeding), not cosmetic.

## Issues Encountered

- The dev host's checkout path (`/home/chris/Vibe Coding/Sourcerer`) contains a space. This is now a proven, recurring landmine class for Nix `mkShell` interactive shells specifically (the `$out`/rpath issue above) — worth remembering for any future flake `shellHook` work in this repo, since the fix is defensive (`$PWD`-relative) but the underlying cause (space in path) isn't something the flake can eliminate.

## User Setup Required

None — no external service configuration required. `direnv` is optional (mentioned in the README as the auto-entry path); `nix develop` works standalone.

## Next Phase Readiness

- Plan 03 (substrate core module + WSL/VM variants + seed nixosTest) can add `checks.x86_64-linux.*` and new NixOS-module inputs to this `flake.nix` without restructuring — the `devShells`/`lib`/`overlays` outputs already established are additive.
- Plan 04's toolchain drift gate can read `rust-toolchain.toml` and `.nvmrc` directly, byte-for-byte, exactly as D-10 specifies.
- FOUND-01's full claim ("clone on a second machine, build the substrate image from cache, no compiling") is not yet fully provable — that needs Plan 05's deployed Attic cache and Plan 06's substituter wiring in the flake. This plan proves the dev-shell half (`nix develop` + a real Tauri build) but not the "no compiling, pulled from cache" half.

---
*Phase: 09-flake-foundation-assurance-chain*
*Completed: 2026-08-04*

## Self-Check: PASSED

All created/modified files confirmed present on disk (rust-toolchain.toml, .nvmrc, flake.nix, flake.lock, nix/lib.nix, .envrc, README.md, this SUMMARY.md). All three task commit hashes (446cee5, 279f6ab, 3daab68) confirmed in `git log --oneline --all`.
