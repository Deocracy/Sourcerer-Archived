# Phase 9: Flake Foundation & Assurance Chain - Discussion Log

> **Audit trail only.** Do not use as input to planning, research, or execution agents.
> Decisions are captured in CONTEXT.md — this log preserves the alternatives considered.

**Date:** 2026-08-04
**Phase:** 9-Flake Foundation & Assurance Chain
**Areas discussed:** Binary cache hosting & cost, Repo topology & shared pins, CI shape & publish gate, Substrate skeleton depth, plus round 2 (nixpkgs pin, dev shell, accounts, boot check)

---

## Binary cache hosting & cost

| Option | Description | Selected |
|--------|-------------|----------|
| Attic on a VPS | Self-hosted, S3-backed, global dedup + GC | ✓ |
| Paid Cachix | Zero ops, vendor dependence | |
| Cachix now, Attic before Phase 11 | Two-step migration | |

**User's choice:** Attic, self-hosted.
**Notes:** Siting went through a detour — an "existing self-hosted box" option was mooted, then the user revealed GPU is being considered as a rented service (CUDA-as-a-service) so there is no owned AI box to co-site on; later resolved to **AWS EC2 + S3, because Deocracy's nonprofit status gets AWS credits/discounts**. Runbook question: chose runbook + automated cache GC (over runbook-only and runbook + nightly channel-green job).

---

## Repo topology & shared pins

| Option | Description | Selected |
|--------|-------------|----------|
| This repo's root | One flake + one lock; downstream repos pin it as an input | ✓ |
| Separate substrate repo | Cleaner boundary, two repos in lockstep | |
| Root now, split later | Extraction migration risk | |

**User's choice:** Repo-root flake.
**Notes:** Flake public surface: chose pinned-nixpkgs re-export + near-empty lib slot (over deferring entirely or designing the full app-input contract now). Shared pins: native files (rust-toolchain.toml, Node pin, existing lockfiles) + CI drift gate — taken as recommended after an explainer on the two-build-worlds drift problem; user did not object. Mid-area, the user corrected course twice, producing the **direction of record**: (1) substrate-first update chain with the substrate as version authority over the Tauri shell; (2) per-part app versioning with an agent-assisted power-user config layer (rejecting the sealed-appliance framing); (3) later, the multi-repo distribution model (core + component repos + community author repos, store pins vetted revisions).

---

## CI shape & publish gate

| Option | Description | Selected |
|--------|-------------|----------|
| All three checks | flake check + seed boot test + windows-latest Tauri + drift gate | ✓ |
| Skip the boot check for now | Build-only until Phase 13 | |
| All three + nightly channel job | Adds scheduled rot detection | |

**User's choice:** All three checks.
**Notes:** This area contained the license/visibility saga: initial answer was public+AGPL; then the user asked for architecture-copyleft (established legally impossible — idea/expression dichotomy; caseworker research run on AGPL §7 self-destruction of added restrictions and the Commons Clause "Sell" wording); user chose private-until-worked-out; final decision: **PolyForm Noncommercial 1.0.0, repo PUBLIC** under the existing Deocracy org, GitHub nonprofit/Pro plan. KVM confusion resolved along the way (KVM/QEMU are CI-runner plumbing, not shipped components); the question was re-worded after user feedback that option shorthand lacked subjects.

---

## Substrate skeleton depth

| Option | Description | Selected |
|--------|-------------|----------|
| Bootable + dev tooling baked in | NixOS-WSL base + nix-ld + vscode-server | ✓ |
| Barest bootable image | Minimum for CI to build/boot | |
| Bootable + dev tooling + provisional app-unit template | Would encode unmeasured hardening guesses | |

**User's choice:** Bootable + dev tooling.
**Notes:** Two-target question (WSL2 vs CI's QEMU VM): chose one shared core module with two thin variants, so CI boots the same contents users get.

---

## Round 2 (user opted to continue past the initial four areas)

| Question | Selected |
|----------|----------|
| nixpkgs pin | Stable NixOS 26.05 (over unstable, over following NixOS-WSL releases) |
| Dev shell contents | Substrate tools + full Tauri toolchain in one shell (over substrate-only, over two shells) |
| GitHub home | Existing Deocracy org — "Deocracy sourcerer is already up"; components get their own Deocracy repos; community components from authors' repos |
| Cache infra providers | AWS EC2 + S3 (nonprofit credits), GitHub Pro nonprofit plan — user's own answer, superseding the Hetzner recommendation |
| CI boot check asserts | Boots + placeholder loopback service answers (miniature of Phase 13's engine check) |

## Claude's Discretion

- "Blocks publish" semantics for Phase 9 = green checks gate the Attic cache push
- Attic sizing, bucket layout, push-token management (secret vs OIDC)
- Exact placeholder service for the seed boot check
- direnv wiring, flake output naming, CI workflow file structure

## Deferred Ideas

- Hosted embedding server has no owning phase (must resolve before Phase 14)
- CUDA-as-a-service / rented GPU revision to AI-PROVIDER-ARCHITECTURE.md's "own box" wording; FAISS-vs-Cozo reconciliation
- Nightly channel-green CI job → natural Phase 11 addition
- Raw local-config escape hatch → superseded by agent-authored apps + per-part pinning direction
- Counsel review of PolyForm NC fit (CLA, noncommercial boundary for university users) before Phase 12/16
