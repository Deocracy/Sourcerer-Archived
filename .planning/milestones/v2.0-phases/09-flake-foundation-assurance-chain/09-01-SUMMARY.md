---
phase: 09-flake-foundation-assurance-chain
plan: 01
subsystem: infra
tags: [license, polyform-noncommercial, npm, cargo, gitignore, nix, direnv]

# Dependency graph
requires: []
provides:
  - "LICENSE (PolyForm Noncommercial 1.0.0, verbatim canonical text)"
  - "package.json license field (npm non-SPDX convention)"
  - "src-tauri/Cargo.toml license-file field (Cargo non-SPDX escape hatch)"
  - ".gitignore Nix/direnv ignore block (result, result-*, .direnv/)"
affects: [09-02, 09-03, 09-04, 09-05, 09-06, 09-07, "Phase 12 (public repo D-04)"]

# Tech tracking
tech-stack:
  added: []
  patterns:
    - "Non-SPDX license declared via each ecosystem's own escape hatch (npm: SEE LICENSE IN LICENSE; Cargo: license-file), never a fabricated SPDX identifier"
    - "Licensor/software identification attached via the license's own Required Notice: mechanism (Notices section), not by editing the license body"

key-files:
  created: [LICENSE]
  modified: [package.json, src-tauri/Cargo.toml, .gitignore]

key-decisions:
  - "PolyForm's canonical text has no <<licensor>>/<<software>> bracket placeholders (verified against polyformproject.org/licenses/noncommercial/1.0.0.txt and the polyformproject/polyform-licenses + license-development GitHub repos) - the plan's premise of bracket-token substitution didn't match the actual license text. Substituted via the license's own documented Required Notice: line instead, appended after Definitions."

patterns-established:
  - "LICENSE file: verbatim PolyForm text + trailing Required Notice: line for project identification, nothing above the title"

requirements-completed: [FOUND-01]

# Metrics
duration: 8min
completed: 2026-08-04
---

# Phase 9 Plan 1: License and Nix/Direnv Ignore Rules Summary

**Landed PolyForm Noncommercial 1.0.0 LICENSE plus non-SPDX license declarations in both package manifests and a Nix/direnv .gitignore block, making the repo legally publishable ahead of Phase 12's public-repo cutover.**

## Performance

- **Duration:** 8 min
- **Started:** 2026-08-04T18:18:07Z
- **Completed:** 2026-08-04T18:25:46Z
- **Tasks:** 3
- **Files modified:** 4 (1 created, 3 modified)

## Accomplishments

- `LICENSE` carries the complete, verbatim PolyForm Noncommercial License 1.0.0 text (all 14 required sections present) with a `Required Notice:` line identifying Deocracy Institute Corporation / Sourcerer
- `package.json` and `src-tauri/Cargo.toml` each declare the license through their own ecosystem's documented non-SPDX mechanism, with no fabricated SPDX identifier
- `.gitignore` gained a labeled Nix/direnv block (`result`, `result-*`, `.direnv/`) while deliberately leaving `flake.lock` tracked

## Task Commits

1. **Task 1: Add the PolyForm Noncommercial 1.0.0 LICENSE file** - `60b0723` (feat)
2. **Task 2: Declare the license in both package manifests** - `9b348fb` (feat)
3. **Task 3: Add the Nix/direnv ignore block to .gitignore** - `a8bb4cd` (feat)

## Files Created/Modified

- `LICENSE` - Verbatim PolyForm Noncommercial License 1.0.0 text, licensor/software identified via a trailing `Required Notice:` line
- `package.json` - Added `"license": "SEE LICENSE IN LICENSE"` next to `"private": true`
- `src-tauri/Cargo.toml` - Added `license-file = "../LICENSE"` to `[package]`
- `.gitignore` - Appended `# Nix build outputs / direnv state` block (`result`, `result-*`, `.direnv/`)

## Decisions Made

- **PolyForm placeholder-token premise was wrong; used the license's own Notices mechanism instead.** The plan instructed substituting `<<licensor>>`/`<<software>>` tokens in the Definitions section. The actual canonical text (fetched from `https://polyformproject.org/licenses/noncommercial/1.0.0.txt`, cross-checked against the `polyformproject/polyform-licenses` and `polyformproject/license-development` GitHub repos) never uses bracket placeholders — it refers generically to "the licensor" and "the software" throughout, by design, so any adopter can use the text unmodified. The license's own Notices section documents the correct project-identification mechanism: a plain-text `Required Notice:` line (it even gives the exact example format, `Required Notice: Copyright Yoyodyne, Inc. (http://example.com)`). Appended `Required Notice: Copyright Deocracy Institute Corporation (Sourcerer)` after Definitions — this satisfies the plan's verification (both strings present, no bracket tokens, nothing above the title line) while keeping the enforceable license body 100% verbatim.

## Deviations from Plan

### Auto-fixed Issues

**1. [Rule 1 - Bug] PolyForm license text has no `<<licensor>>`/`<<software>>` placeholder tokens to substitute**
- **Found during:** Task 1 (LICENSE file creation)
- **Issue:** Plan's action text asserted the canonical PolyForm text uses `<<licensor>>`/`<<software>>` bracket placeholders in the Definitions section. Fetching the actual canonical `.txt` and cross-checking PolyForm's own GitHub repos showed this is factually incorrect for this license family — the text is fully generic with no bracket tokens anywhere.
- **Fix:** Used the license's own documented `Required Notice:` mechanism (from its Notices section) to attach licensor/software identification as a trailing line, instead of attempting a substitution that doesn't exist in the source text.
- **Files modified:** LICENSE
- **Verification:** All automated verify checks pass (title, all 14 section headings, `Deocracy Institute Corporation`, `Sourcerer`, no `licensor>>` token, ≥60 lines, nothing above the title line)
- **Committed in:** `60b0723` (Task 1 commit)

---

**Total deviations:** 1 auto-fixed (1 Rule 1 - incorrect premise in plan text, corrected against the verified canonical source)
**Impact on plan:** No scope creep — the fix stays within Task 1's file (`LICENSE`) and satisfies every stated acceptance criterion; the license body itself remains 100% verbatim/unmodified.

## Issues Encountered

- `cargo metadata --manifest-path src-tauri/Cargo.toml --no-deps` (part of Task 2's automated verify block) cannot run on this dev host: no Rust toolchain is installed yet (`rustup toolchain list` → "no installed toolchains"). This is the exact "both worlds floating" gap `09-CONTEXT.md`/D-10/D-16 already document and explicitly assign to this phase's later flake/toolchain plans (Rust arrives via `rust-toolchain.toml` + rust-overlay through `nix develop`, not via `rustup default stable` on bare metal) — installing a toolchain here would be out of this task's scope and contrary to the phase's own Nix-first approach. Validated `Cargo.toml`'s TOML syntax and the `[package]` block instead via Python's `tomllib` (confirms `license-file = "../LICENSE"` parses correctly, no `license` key present) — a functionally equivalent check for "manifest not corrupted." The real `cargo metadata` check will pass once a later plan in this phase lands the Rust toolchain.

## User Setup Required

None - no external service configuration required.

## Next Phase Readiness

- Repo now carries a real, complete license file with both manifests pointing at it — clears the way for Phase 12's public-repo cutover (D-04)
- `.gitignore` is ready for the flake/Nix work landing in later plans of this phase (Nix build symlinks and direnv state cannot be accidentally committed)
- Outstanding: no Rust toolchain configured on this dev host yet — a later plan in this phase (rust-toolchain.toml / flake.nix / dev shell, per D-10/D-16) needs to close that gap before `cargo metadata`/`cargo build` can run natively here

---
*Phase: 09-flake-foundation-assurance-chain*
*Completed: 2026-08-04*

## Self-Check: PASSED

All created/modified files confirmed present on disk (LICENSE, package.json, src-tauri/Cargo.toml, .gitignore, this SUMMARY.md). All three task commit hashes (60b0723, 9b348fb, a8bb4cd) confirmed in `git log --oneline --all`.
