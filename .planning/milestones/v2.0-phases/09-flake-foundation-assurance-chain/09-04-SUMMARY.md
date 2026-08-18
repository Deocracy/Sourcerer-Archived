---
phase: 09-flake-foundation-assurance-chain
plan: 04
subsystem: infra
tags: [ci, github-actions, nix, nixostest, drift-gate, tauri, vitest, rustup]

# Dependency graph
requires:
  - phase: 09-02
    provides: "flake.nix devShell, rust-toolchain.toml, .nvmrc, rust-overlay pin"
  - phase: 09-03
    provides: "nix flake check including the seed nixosTest VM boot leg"
provides:
  - "Deocracy/Sourcerer public under the Deocracy org, this repo's history as master/default"
  - "scripts/drift-gate.mjs (D-10 cross-platform toolchain pin comparison)"
  - ".github/workflows/ci.yml — nix-checks (nix flake check -L + drift gate) and windows-tauri (rustup show, drift gate, vitest, verify:fonts, tauri build) jobs"
  - "One genuinely green end-to-end CI run (both jobs) on the pushed history"
  - "src/host/ai.ts: ai()/loadSession() honest-degrade on Channel construction failure, not just invoke() rejection"
affects: ["09-05", "09-06", "09-07"]

# Tech tracking
tech-stack:
  added: []
  patterns:
    - "Drift gate as a zero-dependency Node script (scripts/drift-gate.mjs) invoked identically inside nix develop and on windows-latest — one judge, two environments"
    - "CI negative-testing a pin-drift gate must freeze the resolved toolchain before mutating the pin file — spawning a fresh `nix develop` after mutating rust-toolchain.toml re-derives the toolchain FROM the mutated file (rust-overlay's fromRustupToolchainFile), so it can never diverge from itself; the real test enters the shell once, then mutates and re-invokes the script within that already-resolved session"

key-files:
  created: [scripts/drift-gate.mjs, .github/workflows/ci.yml]
  modified: [src/host/ai.ts]

key-decisions:
  - "approve-both selected for the Task 1 checkpoint: repo public, master pushed as default, placeholder main + stray claude/wiki-graph-llm-b11djf branches deleted"
  - "Drift-gate negative test runs inside one persistent `nix develop` shell (not a fresh `nix develop --command` per invocation) — the fresh-shell form re-resolves the mutated pin file live via rust-overlay and can never produce a real mismatch on the Nix side by construction"
  - "Fixed a real unhandled-rejection bug in src/host/ai.ts (ai()/loadSession()) rather than routing around it — the D-06 honest-degrade contract already promised callers never need a try/catch, but Channel construction sat outside the try/catch that backed that promise"

patterns-established:
  - "drift-gate — message prefix convention matches scripts/verify-fonts.mjs exactly (ESM, node: core imports only, labeled console.error + exit 1, plain console.log PASS)"

requirements-completed: [FOUND-02]

# Metrics
duration: ~65min (includes ~30min CI wall-clock across two pushed runs)
completed: 2026-08-04
---

# Phase 9 Plan 4: Assurance Chain — Publish + CI Pipeline Summary

**`Deocracy/Sourcerer` is public with this repo's history as `master`/default, and one real `.github/workflows/ci.yml` run is green end-to-end: `nix flake check -L` (including the seed nixosTest VM boot), a `windows-latest` Tauri build with `npx vitest run`/`verify:fonts`/`tauri build`, and `scripts/drift-gate.mjs` gating both platforms against `rust-toolchain.toml`/`.nvmrc`.**

## Performance

- **Duration:** ~65 min (drift-gate authoring + local verification, CI authoring, two pushed CI runs at ~7-8 min wall-clock each)
- **Started:** 2026-08-04T19:20Z (approx, first Read)
- **Completed:** 2026-08-04T20:38Z (second green CI run confirmed)
- **Tasks:** 3 (Task 1 checkpoint + Task 2 + Task 3)
- **Files modified:** 3 (2 created, 1 modified)

## Accomplishments

- Task 1 checkpoint resolved `approve-both`: `Deocracy/Sourcerer` flipped public, local `master` pushed and set as default branch, placeholder `main` (one-file `LICENSE` stub) and the stray `claude/wiki-graph-llm-b11djf` branch deleted
- `scripts/drift-gate.mjs`: zero-dependency ESM script comparing the environment's resolved `rustc`/`node` against `rust-toolchain.toml`'s `channel` and `.nvmrc`, rejecting floating channel names, resolving paths via `import.meta.url`. Verified: passes clean, fails on genuine drift (frozen-shell negative test), fails on a floating channel, restores the pin file cleanly
- `.github/workflows/ci.yml`: `nix-checks` job (`nix-installer-action@v22` → `nix flake check -L` → drift gate inside `nix develop`) and `windows-tauri` job (`setup-node@v7` reading `.nvmrc` → bare `rustup show` → `npm ci` → drift gate → `npx vitest run` → `npm run verify:fonts` → `npm run tauri build`), `permissions: contents: read`, no Rust-toolchain-setup action, no publish/attic step
- Two real pushed CI runs: first run's `windows-tauri` job failed on 15 unhandled promise rejections in `AssistantPanel.test.tsx` (pre-existing bug in `src/host/ai.ts`, newly exposed by running `npx vitest run` in CI for the first time — see Deviations); fixed and re-pushed; second run genuinely green on both jobs (`gh run view` conclusion `success` for `nix-checks` and `windows-tauri`)

## Task Commits

1. **Task 2: Cross-platform toolchain drift gate** - `a8eafef` (feat)
2. **Task 3: Publish repo + CI workflow** - `9af9b21` (feat) — includes the `approve-both` publish actions (executed in the main session after the Task 1 checkpoint approval) plus `.github/workflows/ci.yml`
3. **Deviation fix (Task 3, pre-green-run):** `36e778d` (fix) — `src/host/ai.ts` Channel-construction guard, required to reach a genuinely green CI run

_Task 1 (checkpoint:human-action) produced no commit — decision-only, executed as the first half of Task 3's action per plan structure._

## Files Created/Modified

- `scripts/drift-gate.mjs` - D-10 cross-platform toolchain pin comparison, invoked in both CI jobs
- `.github/workflows/ci.yml` - `nix-checks` + `windows-tauri` jobs, FOUND-02's three checks plus the drift gate
- `src/host/ai.ts` - `ai()`/`loadSession()` now catch `Channel` construction failures and route them through the existing error+done+resolve honest-degrade path, matching the `invoke()`-reject path already there

## Decisions Made

- **`approve-both`** for the Task 1 checkpoint (user decision, relayed by the coordinator, executed by the user directly in the main session after the harness's auto-mode classifier blocked the mutating `git remote add`/`gh repo edit --visibility` commands from this agent — see Issues Encountered).
- **Drift-gate negative test uses a single persistent `nix develop` shell**, not `nix develop --command` per invocation, for the plan's own negative-test script. A fresh `nix develop --command node scripts/drift-gate.mjs` after mutating `rust-toolchain.toml` re-derives the devShell's `rustToolchain` live from the now-mutated file (`pkgs.rust-bin.fromRustupToolchainFile ./rust-toolchain.toml`), so the "actual" and "expected" values move together by construction and can never diverge on the Nix side — the plan's literal verify script (spawning a fresh `nix develop` after the `sed`) cannot produce a real mismatch there. Verified this structurally, then re-ran the same assertions inside one already-resolved shell (mutate the file without re-entering `nix develop`) to exercise a genuine drift and confirm the gate actually catches it.
- **Fixed the unhandled-rejection bug at its root** (`src/host/ai.ts`) rather than papering over it in the test file. Both `ai()` and `loadSession()` construct `new Channel()` synchronously inside a `Promise` executor with no guard; a construction failure throws, and JS Promise-executor semantics convert that into a rejected promise the fire-and-forget caller (`AssistantPanel`'s per-mount `loadSession` effect) never catches — directly contradicting the functions' own doc comments ("callers never need a try/catch"). One `try/catch` per function, both routing through the existing error+done+resolve degrade path.

## Deviations from Plan

### Auto-fixed Issues

**1. [Rule 1 - Bug] CI-comment wording collided with the plan's own forbidden-substring grep**
- **Found during:** Task 3, local static verification of `.github/workflows/ci.yml`
- **Issue:** An explanatory comment ("No Rust setup action — dtolnay/rust-toolchain does not read rust-toolchain.toml...") contained the literal string the plan's `grep -qE "dtolnay/rust-toolchain"` check treats as a failure signal (same landmine 09-03 hit with `0.0.0.0`/`nixos-wsl` in comments).
- **Fix:** Reworded to "the common toolchain-setup actions do not read the repo's toolchain pin file" — same guidance, no literal forbidden substring.
- **Files modified:** `.github/workflows/ci.yml`
- **Verification:** Re-ran the plan's exact static verify block; all greps passed.
- **Committed in:** `9af9b21`

**2. [Rule 1 - Bug] Unhandled Channel-construction rejection in `src/host/ai.ts`**
- **Found during:** Task 3, first pushed CI run — `windows-tauri` job failed `npx vitest run` with 15 unhandled promise rejections in `AssistantPanel.test.tsx` despite all 23 named assertions passing
- **Issue:** `ai()` and `loadSession()` both construct `new Channel()` synchronously inside their `Promise` executor with no try/catch; on any construction failure the executor throws, which Promise semantics auto-convert to a rejected promise. `AssistantPanel`'s `useEffect` calls `void host.loadSession(...)` (fire-and-forget) on every mount for the default real session — any test that renders `<AssistantPanel />` before establishing `window.__TAURI_INTERNALS__` via `mockIPC()` hits this. This is a genuine production bug (contradicts both functions' own "callers never need a try/catch" doc-comment contract), not a test-only concern — it was simply never exercised by `npx vitest run` before, since this is the first time the suite has run in CI at all.
- **Fix:** Wrapped both `Promise` executor bodies in `try/catch`, routing a `Channel` construction failure through the same `onEvent({type:"error"}) → onEvent({type:"done"}) → resolve()` degrade path already used for `invoke()` rejection.
- **Files modified:** `src/host/ai.ts`
- **Verification:** `npx vitest run` locally — 31 files, 203 tests, exit 0, zero unhandled errors (previously 15). Re-pushed; second CI run green on both jobs.
- **Committed in:** `36e778d`

---

**Total deviations:** 2 auto-fixed (1 Rule 1 comment-wording collision, 1 Rule 1 real production bug blocking the plan's own "genuinely green run" success criterion). Both fixes are load-bearing for the plan's stated acceptance criteria; the second is scoped tightly to the two functions with the identical unguarded pattern, no broader refactor.

## Issues Encountered

- **Runtime permission-classifier block, not a GSD checkpoint.** After the Task 1 checkpoint's `approve-both` decision was relayed, this agent's own attempts to run `git remote add` and `gh repo edit --visibility public` were denied by Claude Code's auto-mode permission classifier (a harness-level gate distinct from GSD's checkpoint system). Per the harness's own guidance ("STOP and explain... let the user decide"), this agent halted and reported the block rather than attempting a workaround. The user then switched to manual permission mode and executed the publish sequence directly in the main session; this agent verified the resulting remote/branch/visibility state before continuing with Task 3's CI authoring.
- **`gh run watch`'s annotation fetch hit a transient GitHub API 503** partway through the first run; switched to polling `gh run view --json status` instead, no functional impact.
- **GitHub reported Dependabot alerts once the repo went public** — `gh api repos/Deocracy/Sourcerer/dependabot/alerts` currently lists 30 open alerts (13 high, 17 medium) across the dependency graph (`git push`'s own summary line showed a larger, differently-bucketed count, likely a broader graph snapshot). Per the coordinator's explicit instruction, not acted on this plan — flagged here for whichever later phase owns dependency hygiene.

## User Setup Required

None — no external service configuration required. The repo publish steps were user-executed (see Issues Encountered) but required no new credentials beyond the already-authenticated `gh` session.

## Next Phase Readiness

- FOUND-02 is satisfied: pushing to `Deocracy/Sourcerer` now runs all three required checks (`nix flake check` including the seed nixosTest, the `windows-latest` Tauri build, and the D-10 drift gate on both platforms), and one real run has passed both jobs.
- No publish/cache-push job exists yet — Plan 06 owns adding it behind these two green jobs, per this plan's explicit scope boundary.
- `packages.substrate-system` (09-03) is still unpushed to any cache — Plan 05/06's Attic wiring has a real, CI-proven-buildable target to push once the cache exists.
- Dependabot alerts (30 open, 13 high) are a new, real signal now that the repo is public — worth a deliberate look before/alongside Plan 05's cache work, not silently carried forward indefinitely.

---
*Phase: 09-flake-foundation-assurance-chain*
*Completed: 2026-08-04*

## Self-Check: PASSED

All three plan-listed/deviation files confirmed present on disk (scripts/drift-gate.mjs,
.github/workflows/ci.yml, src/host/ai.ts), plus this SUMMARY.md. All three commit hashes
(a8eafef, 9af9b21, 36e778d) confirmed in `git log --oneline --all`. Latest pushed CI run
(36e778d) confirmed `success` via `gh run view --json status,conclusion,jobs` for both
`nix-checks` and `windows-tauri`.
