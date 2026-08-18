---
phase: 9
slug: flake-foundation-assurance-chain
status: planned
nyquist_compliant: false
wave_0_complete: false
created: 2026-08-04
---

# Phase 9 — Validation Strategy

> Per-phase validation contract for feedback sampling during execution.
> Derived from 09-RESEARCH.md "## Validation Architecture".

---

## Test Infrastructure

| Property | Value |
|----------|-------|
| **Framework** | N/A — infrastructure phase; validation IS the CI pipeline (`nix flake check` + GitHub Actions jobs) |
| **Config file** | `flake.nix` (checks output) + `.github/workflows/*.yml` — neither exists yet (Wave 0) |
| **Quick run command** | `nix flake check` (on the NixOS dev host) |
| **Full suite command** | Complete GitHub Actions workflow run (all jobs incl. `windows-latest` + drift gate) |
| **Estimated runtime** | flake eval seconds; full nixosTest minutes; full CI run tens of minutes (windows-latest Tauri build dominates) |

Existing app-code gates (`npm test` via vitest, `scripts/verify-fonts.mjs`) run inside the `windows-latest` job — they validate application code, not this phase's own deliverables.

---

## Sampling Rate

- **After every task commit:** Run `nix flake check` (fast eval leg)
- **After every plan wave:** Full CI workflow run (push branch / PR) — all jobs plus drift gate
- **Before `/gsd-verify-work`:** A genuinely green end-to-end CI run PLUS a real Attic push/pull round-trip proving FOUND-01's "no compiling on a second machine" claim
- **Max feedback latency:** minutes locally; one CI round-trip per wave

---

## Per-Task Verification Map

| Task ID | Plan | Wave | Requirement | Secure Behavior | Test Type | Automated Command | File Exists | Status |
|---------|------|------|-------------|-----------------|-----------|-------------------|-------------|--------|
| 01-T1 | 09-01 | 1 | FOUND-01 | License text present and verbatim | source assertion | `grep -q "PolyForm Noncommercial License 1.0.0" LICENSE` | ❌ W0 | ⬜ pending |
| 01-T2 | 09-01 | 1 | FOUND-01 | Manifests use non-SPDX escape hatch, not a fabricated id | source assertion | `node -e 'require("./package.json")'` + `cargo metadata --manifest-path src-tauri/Cargo.toml --no-deps` | ❌ W0 | ⬜ pending |
| 01-T3 | 09-01 | 1 | FOUND-01 | flake.lock stays tracked; Nix outputs ignored | source assertion | `grep -qx result .gitignore; ! grep -q flake.lock .gitignore` | ❌ W0 | ⬜ pending |
| 02-T1 | 09-02 | 1 | FOUND-01 | Pins are exact versions, never floating channels | source assertion | `grep -qx 'channel = "1.97.1"' rust-toolchain.toml` | ❌ W0 | ⬜ pending |
| 02-T2 | 09-02 | 1 | FOUND-01 | One committed lock pins every input to a revision | integration | `nix flake check --no-build` + `nix flake metadata --json` rev assertions | ❌ W0 | ⬜ pending |
| 02-T3 | 09-02 | 1 | FOUND-01 | Dev shell compiles the Tauri crate with no imperative installs | integration | `nix develop --command cargo build --manifest-path src-tauri/Cargo.toml` | ❌ W0 | ⬜ pending |
| 03-T1 | 09-03 | 2 | FOUND-01 | Seed service binds loopback only; no hardening template (D-14) | source assertion | `! grep -q 0.0.0.0 nix/substrate/core.nix` + hardening-directive grep | ❌ W0 | ⬜ pending |
| 03-T2 | 09-03 | 2 | FOUND-01 | Substrate closure is buildable and cacheable; WSL adapter isolated to one variant | integration | `nix build .#substrate-system` + `nix eval .#nixosConfigurations.substrate-vm.config.wsl` must fail | ❌ W0 | ⬜ pending |
| 03-T3 | 09-03 | 2 | FOUND-02 | VM boots to multi-user and a service answers on loopback with the expected body (D-12) | integration (nixosTest) | `nix flake check` / `nix build .#checks.x86_64-linux.seed-boot-test` | ❌ W0 | ⬜ pending |
| 04-T1 | 09-04 | 3 | FOUND-02 | Public-history disclosure is explicitly approved before it happens | checkpoint | human decision (blocking) | n/a | ⬜ pending |
| 04-T2 | 09-04 | 3 | FOUND-02 | Drift gate fails on a drifted pin (negative test required) | unit | `node scripts/drift-gate.mjs` clean-pass + mutated-pin fail | ❌ W0 | ⬜ pending |
| 04-T3 | 09-04 | 3 | FOUND-02 | All three checks run and pass on real runners; no toolchain action that ignores the pin file | CI smoke | `gh run list --limit 1 --json conclusion` = success | ❌ W0 | ⬜ pending |
| 05-T1 | 09-05 | 3 | FOUND-03 | Billable infra + DNS not provisioned without credentials and named targets | checkpoint | human action (blocking) | n/a | ⬜ pending |
| 05-T2 | 09-05 | 3 | FOUND-03 | Bucket private + encrypted; S3 reached via instance profile, no static keys; 8080 not exposed | config assertion | `aws s3api get-public-access-block` + security-group rule assertion | ❌ W0 | ⬜ pending |
| 05-T3 | 09-05 | 3 | FOUND-03 | Retention nonzero (Pitfall 5); CI token push-scoped, never admin; no secret in the Nix store | config + integration | `attic cache info sourcerer` retention nonzero; `curl -sfI https://<host>`; `gh secret list` | ❌ W0 | ⬜ pending |
| 06-T1 | 09-06 | 4 | FOUND-01 | Fresh clone knows the cache and its signing key with no client setup | integration | `curl -sf https://<host>/sourcerer/nix-cache-info` + `nix flake check --no-build` | ❌ W0 | ⬜ pending |
| 06-T2 | 09-06 | 4 | FOUND-02 | Push token reachable only from push-on-default; publish needs every check | CI smoke | `publish` job success + `<store-hash>.narinfo` fetchable from the cache | ❌ W0 | ⬜ pending |
| 06-T3 | 09-06 | 4 | FOUND-03 | Runbook covers bump cadence, red-channel, retention, restore, cost, caveats — and holds no secrets | doc assertion | section greps + `! grep -E "eyJ[A-Za-z0-9_-]{20,}\|AKIA"` | ❌ W0 | ⬜ pending |
| 07-T1 | 09-07 | 5 | FOUND-01 | Cold runner builds the closure with zero local compilation, on every push | CI gate | `nix build .#substrate-system --max-jobs 0 --accept-flake-config` | ❌ W0 | ⬜ pending |
| 07-T2 | 09-07 | 5 | FOUND-02 | Both red legs leave `publish` `skipped` and the cache unchanged | CI experiment | `gh run view --json jobs` conclusions recorded with run URLs | ❌ W0 | ⬜ pending |

*Status: ⬜ pending · ✅ green · ❌ red · ⚠️ flaky*
*(Task IDs filled by the planner 2026-08-04. `File Exists` stays ❌ until the wave that creates the artifact runs.)*

---

## Wave 0 Requirements

- [ ] `flake.nix` — does not exist yet (repo has nothing Nix-shaped)
- [ ] `.github/workflows/*.yml` — does not exist yet
- [ ] `rust-toolchain.toml` + Node pin file — do not exist yet (both worlds currently float)
- [ ] The seed nixosTest (boot + placeholder loopback HTTP service) — does not exist yet
- [ ] A deployed, reachable Attic server (AWS EC2 + S3) — does not exist yet; FOUND-01's "no compiling" proof is necessarily a late-wave step

*Framework install: none — Nix's own tooling (`nix flake check`, nixosTest) is the framework.*

---

## Manual-Only Verifications

| Behavior | Requirement | Why Manual | Test Instructions |
|----------|-------------|------------|-------------------|
| Clean second-machine clone → dev shell → cache-only substrate build | FOUND-01 | Needs a machine (or pristine checkout/user) without warm /nix/store paths | Fresh clone outside the dev tree, `nix develop`, build substrate image, inspect log: everything substituted from the Attic URL, nothing compiled |
| Runbook completeness review | FOUND-03 | Doc quality is not scriptable | Read the channel-maintenance runbook: bump cadence, red-channel response, retention policy, restore-from-scratch all present and actionable |
| Red-blocks-publish end-to-end | FOUND-02 | Requires a deliberate failure pushed to CI | Open a PR with a broken flake check (or failing test); confirm the publish/cache-push job does not run |
