# Phase 9: Flake Foundation & Assurance Chain - Research

**Researched:** 2026-08-04
**Domain:** Nix flakes, NixOS-WSL image builds, GitHub Actions CI (nixosTest + conventional Windows job), self-hosted Attic binary cache on AWS, license file mechanics
**Confidence:** MEDIUM-HIGH (Nix mechanics HIGH via official docs/GitHub; Attic maturity and exact AWS sizing MEDIUM — WebSearch-verified, not Context7-verified; no Context7 MCP available this session, CLI fallback `ctx7` also not present — all claims below are WebSearch/WebFetch cross-verified against official docs where possible)

<user_constraints>
## User Constraints (from CONTEXT.md)

### Locked Decisions

**Binary cache (FOUND-03)**
- D-01: Attic, self-hosted on AWS. Attic server on a small EC2 instance; storage on S3 (Attic's native backend — no compatibility caveat). Chosen over paid Cachix (vendor dependence) and Hetzner (Deocracy's nonprofit status gets AWS credits/discounts). Cache stays a plain substituter URL — swappable by construction.
- D-02: Runbook + GC automation. FOUND-03's runbook is written (nixpkgs bump cadence, red-channel response, retention policy, restore-from-scratch) AND Attic's GC/retention is configured so the cache does not grow unbounded by default.
- D-03: Publish = cache push. Until Phase 11 builds the real channel, "blocks publish" means: the closure is pushed to Attic only after every CI check is green. The cache push is the publish step.

**Repo, license, visibility**
- D-04: Public repo under the existing Deocracy GitHub org (repo currently has NO git remote — Phase 9 wires it). Public = free Actions minutes + KVM on standard runners. Full git history including `.planning/` becomes public.
- D-05: License = PolyForm Noncommercial 1.0.0 (supersedes D-P1's plain AGPL). Prohibits commercial *use* entirely; has NO copyleft. Deocracy retains all commercial rights. A CLA is required from the first outside contributor.
- D-06: GitHub plan: Deocracy org on GitHub's nonprofit/Pro plan.

**Flake topology & public surface**
- D-07: flake.nix at this repo's root — one flake, one lock. Downstream repos consume it as a pinned flake input.
- D-08: nixpkgs pin = stable NixOS 26.05 (nixos-26.05 branch). Calm 6-month runbook rhythm. NixOS-WSL's 2605.x image line (spike-010-validated) is built from it.
- D-09: Public surface = pinned-nixpkgs re-export + a named, deliberately near-empty lib/overlay output. Makes TOOLS-02 and STORE-01 mechanically true later.

**Shared version pins (success criterion 4)**
- D-10: Native files + CI drift gate. No invented format. `rust-toolchain.toml` (read by rustup on Windows AND by rust-overlay in the flake) + a Node version pin the flake and `actions/setup-node` both read; Nix consumes the existing `package-lock.json` (importNpmLock) and `Cargo.lock` (crane) directly for libraries. Plus one CI job that FAILS when the `windows-latest` job's resolved toolchain diverges from the flake's. Neither pin file exists today.

**CI (FOUND-02)**
- D-11: All three checks, red blocks publish: (1) `nix flake check`; (2) seed nixosTest; (3) `windows-latest` conventional Tauri build. Plus D-10's drift gate.
- D-12: The boot check asserts boots + a placeholder service answers — multi-user target reached AND a trivial HTTP service inside the image answers on a loopback port.
- D-13: KVM/QEMU are CI-runner plumbing only — nothing you ship contains them. Public repo means KVM is available on standard runners.

**Substrate image skeleton**
- D-14: Bootable + dev tooling baked in. NixOS-WSL base module + nix-ld + vscode-server support. Nothing user-facing. No app-unit hardening template this phase.
- D-15: One shared core module, two thin variants. Contents defined once; flake exposes the real WSL2 image target AND a plain-VM variant the CI boot check uses.

**Dev shell**
- D-16: One shell, both worlds. `nix develop` provides substrate-build tools (nix build wrappers, attic client) AND the full Tauri toolchain — Rust via rust-toolchain.toml + rust-overlay, Node, tauri CLI, webkitgtk system libs.

### Claude's Discretion
- Attic instance sizing, S3 bucket layout, token/key management for cache push (GH Actions secret vs OIDC).
- Exact seed placeholder service (any trivial HTTP responder).
- direnv/.envrc wiring for the dev shell.
- Flake output naming/layout beyond the decisions above.
- CI workflow file structure (one workflow vs several).

### Deferred Ideas (OUT OF SCOPE)
- Hosted embedding server has no owning phase — not this phase's concern.
- CUDA-as-a-service / rented GPU for the hosted AI tier.
- Nightly channel-green CI job (rebuild against bumped nixpkgs) — natural Phase 11 addition, not selected for Phase 9.
- User-facing local system config (editable configuration.nix / blessed overlay) — NOT the customization path.
- Counsel review of PolyForm NC fit — advisable before Phase 12/16, not a Phase 9 gate.
</user_constraints>

<phase_requirements>
## Phase Requirements

| ID | Description | Research Support |
|----|-------------|------------------|
| FOUND-01 | Developer can clone the repo on a second machine, enter the dev shell, and build the substrate image from the binary cache without compiling | `## Architecture Patterns` (flake layout, devShell), `## Code Examples` (Attic push/pull, substituter config), `## Package Legitimacy Audit` (flake inputs) |
| FOUND-02 | A red CI run (flake check, seed nixosTest, `windows-latest` Tauri job) blocks any publish | `## Architecture Patterns` (CI workflow shape), `## Code Examples` (nixosTest wiring, windows-latest job, drift gate), `## Validation Architecture` |
| FOUND-03 | Binary-cache hosting is decided, costed, and documented with a channel-maintenance runbook (cache is a production service from P2b on) | `## Don't Hand-Roll` (Attic vs hand-rolled cache), `## Standard Stack` (Attic + S3 + EC2), `## Common Pitfalls` (GC, retention, egress cost) |
</phase_requirements>

## Summary

Phase 9 assembles known Nix primitives into one repo-root flake and one CI pipeline; it does not invent new mechanics. The four moving pieces — flake topology, NixOS-WSL image target + plain-VM nixosTest variant, GitHub Actions with KVM-enabled `nix flake check`, and a self-hosted Attic cache on AWS EC2+S3 — are each individually well-documented and widely deployed, but this specific combination (WSL image + QEMU test variant sharing one module, Windows-conventional Tauri build living beside a Nix substrate build, in one flake) has few complete worked examples online. Plan tasks as assembly-with-verification, not novel design: each piece has a canonical reference below, and the plan should wire them together and prove the wiring with `nix flake check` + a first green CI run rather than re-deriving mechanics from scratch.

The one confirmed hard constraint: `nix-installer-action` (the standard way to get KVM-enabled Nix on GitHub Actions) stops supporting upstream Nix installs after **2026-01-01**, which has passed — as of this research date the action installs Determinate Nix by default (`determinate: true` is now the default; upstream is no longer selectable). This directly answers D-P4/D-08's "pin which Nix ships" question for CI: CI's Nix is Determinate Nix by construction, not upstream, unless a different installer action is deliberately chosen. This does not block `nix flake check` — Determinate Nix is a hardened build of upstream Nix and is a drop-in substitute for `nix flake check`/`nixosTest` purposes — but it should be stated explicitly in the flake/CI docs so nobody is surprised later.

**Primary recommendation:** One flake at repo root with `nixpkgs` pinned to `nixos-26.05`, `rust-overlay` (oxalica) for the Rust toolchain read from `rust-toolchain.toml`, `nixos-wsl` (nix-community) as a flake input feeding one shared NixOS module used by both the `.wsl` image output and a plain-VM `nixosTest` in `checks.x86_64-linux`; CI via `DeterminateSystems/nix-installer-action` (KVM auto-enabled on public repos) running `nix flake check`, a separate `windows-latest` job using `actions/setup-node` + rustup's native `rust-toolchain.toml` auto-detection for the Tauri build, and a drift-gate job that diffs the Windows job's resolved `rustc`/`node` versions against the same two pin files the flake reads. Cache: `services.atticd` (merged into nixpkgs) on a small EC2 instance backed by S3, GC configured server-side (interval + per-cache retention-period, no cron needed), pushed to from CI via a scoped token in a GH Actions secret.

## Architectural Responsibility Map

| Capability | Primary Tier | Secondary Tier | Rationale |
|------------|-------------|----------------|-----------|
| Flake definition (devShells/packages/nixosConfigurations) | Build tooling (repo root) | — | Single source of truth for both build worlds per D-07 |
| Rust/Node toolchain pins | Native pin files (`rust-toolchain.toml`, Node version file) | Flake (reads same files) + CI (reads same files) | D-10 — no invented format, both worlds read the same native files |
| Substrate image contents | NixOS module (shared) | Two thin variants: NixOS-WSL adapter, plain-VM adapter | D-15 — one definition, CI tests the same contents users get |
| CI orchestration | GitHub Actions (`.github/workflows/`) | — | GitHub-hosted, public repo, KVM available on standard runners |
| Binary cache serving | Attic server (EC2, `services.atticd`) | S3 (object storage backend) | D-01 — self-hosted, swappable substituter URL by construction |
| Cache push (publish gate) | CI job (post-green) | Attic client (`attic push`) | D-03 — cache push IS the publish step until Phase 11 |
| License declaration | Repo root `LICENSE` file | `package.json`/`Cargo.toml` `license` field (SPDX id) | D-05 — static, no runtime component |

## Standard Stack

### Core

| Component | Version / Pin | Purpose | Why Standard |
|---------|---------|---------|--------------|
| nixpkgs | `nixos-26.05` branch [CITED: nixos.org release channels, per D-08 locked decision] | Base package set, NixOS modules | Locked decision D-08; stable 6-month channel |
| Determinate Nix (via `nix-installer-action`) | Whatever `DeterminateSystems/nix-installer-action@main` currently pins [ASSUMED — version not pinned to a specific tag in this research; pin to a released tag, not `@main`, at implementation time] | Nix binary + flakes on GitHub Actions runners, with KVM auto-enabled | [VERIFIED via WebFetch of the action's own README, 2026-08-04]: as of this research date the action installs Determinate Nix by default; **upstream Nix installation via this action stopped being available 2026-01-01**, which has already passed as of this research date. `determinate: false` no longer yields upstream Nix through this action. |
| `nixos-wsl` (nix-community) | `github:nix-community/NixOS-WSL` — pin to a `nixos-26.05`-compatible ref/release (the 2605.x line; spike 010 validated `2605.7.2`) [CITED: github.com/nix-community/NixOS-WSL] | NixOS-WSL base module + `.wsl` image builder (`system.build.tarballBuilder`) | Only maintained NixOS-on-WSL2 distribution mechanism; already validated end-to-end in spike 010 |
| `rust-overlay` (oxalica) | `github:oxalica/rust-overlay` [CITED: github.com/oxalica/rust-overlay] | Provides `rust-bin.fromRustupToolchainFile` so the flake's Rust toolchain is read from the same `rust-toolchain.toml` rustup reads on Windows | Standard, actively maintained way to make a flake and rustup agree on one toolchain file — directly satisfies D-10 |
| Attic (`zhaofengli/attic`) | `services.atticd` — now packaged in nixpkgs proper (`services.atticd.package` option confirmed present) [CITED: mynixos.com option listing, cross-referenced against docs.attic.rs] | Self-hosted, S3-backed, multi-tenant Nix binary cache with built-in GC | Locked decision D-01. Note: the `attic` crate on crates.io is labeled "WIP" [CITED: crates.io/crates/attic] — treat as a maturity flag, not a blocker; it is nonetheless the standard/only actively-maintained self-hosted S3-backed Nix cache server and is what the CONTAINER-PLATFORM research already committed to. |

### Supporting

| Library | Version | Purpose | When to Use |
|---------|---------|---------|-------------|
| `flake-utils` (numtide) or hand-rolled `forAllSystems` | any current tag | Reduces `x86_64-linux`-only boilerplate across `devShells`/`packages`/`checks` | Optional — this flake only targets `x86_64-linux` (dev host + CI + Windows via non-Nix path), so a hand-rolled single-system flake is also reasonable and arguably simpler (ponytail: don't add flake-utils for one system) |
| `attic-client` (CLI, part of the `attic` package) | tracks server version | `attic login`, `attic push`, `attic use` — both in CI and on dev machines | Add to the shared devShell (D-16) and as a CI step before/after build |
| `dtolnay/rust-toolchain` (GH Action) OR plain `rustup` (already on `windows-latest` runners) | pinned tag if used | Installing/selecting the Rust toolchain on the `windows-latest` job | [CITED, mixed confidence]: GitHub's `windows-latest` runner image ships `rustup` preinstalled; running any `cargo`/`rustup` command in a directory containing `rust-toolchain.toml` causes rustup to auto-install and use that exact toolchain with **zero extra action needed** — this is the simplest drift-gate-friendly path. `dtolnay/rust-toolchain` does NOT natively read `rust-toolchain.toml` (confirmed via its own issue tracker) — do not rely on it to honor the pin file; either skip it and let bare rustup pick up the file, or use an action explicitly documented to read toolchain files. |
| `actions/setup-node` | `@v6` (current major at research time) [CITED: actions/setup-node README] | Installs Node on `windows-latest`, reading the same Node pin the flake reads | `node-version-file: '.nvmrc'` input reads `.nvmrc`, `.node-version`, `.tool-versions`, or `package.json` `engines` — pick ONE format and have the flake read the identical file |

### Alternatives Considered

| Instead of | Could Use | Tradeoff |
|------------|-----------|----------|
| Attic self-hosted on AWS | Cachix (paid tier) | Rejected by D-01 — vendor dependence on a production-critical path; Garnix's July 2026 shutdown is the standing lesson. Cachix free tier (5 GB) is explicitly known-insufficient for an ML-stack closure (per CONTAINER-PLATFORM-PLAN.md P1 notes) even before this substrate touches ML packages in a later phase. |
| Attic self-hosted on AWS EC2+S3 | Attic self-hosted on Hetzner | Rejected by D-01 — Deocracy's nonprofit AWS credits/discounts beat Hetzner's raw price. |
| `dtolnay/rust-toolchain` | `actions-rust-lang/setup-rust-toolchain` | The latter explicitly documents `rust-toolchain.toml` auto-detection and adds a problem-matcher; worth using if bare-rustup pickup proves fragile in practice — but bare rustup (no action at all) is the simplest correct default and should be tried first (ponytail: fewer moving parts). |
| Server-side GC config | A cron job hitting `attic gc` | Unnecessary — `atticd` runs its own internal GC loop on a configured interval (default 12h) with a per-cache `retention-period` setting (default 3 months; `0` disables time-based GC) [CITED: docs.attic.rs/user-guide]. Answers research question 7 directly: GC is atticd server config, not external cron. |

**Installation (flake inputs, not npm/pip):**
```nix
# flake.nix inputs block (illustrative — see Architecture Patterns for full skeleton)
inputs = {
  nixpkgs.url = "github:NixOS/nixpkgs/nixos-26.05";
  nixos-wsl.url = "github:nix-community/NixOS-WSL";
  nixos-wsl.inputs.nixpkgs.follows = "nixpkgs";
  rust-overlay.url = "github:oxalica/rust-overlay";
  rust-overlay.inputs.nixpkgs.follows = "nixpkgs";
};
```

**Version verification performed:** `nix --version` on the dev host resolves to upstream Nix 2.34.8 with flakes already enabled globally (`experimental-features = fetch-tree flakes nix-command` in the host's system-wide `nix.conf`) [VERIFIED: `nix show-config` on this machine, 2026-08-04]. This is the NixOS host's own Nix, separate from whatever CI's `nix-installer-action` provisions — no drift concern between them since neither reads the other's config; both simply need to correctly evaluate the same flake.

## Package Legitimacy Audit

This phase's only external dependencies are **Nix flake inputs** (GitHub-hosted, not an npm/PyPI/cargo registry surface) plus the pre-existing npm/cargo lockfiles this phase reads but does not modify. `slopcheck` targets package registries (npm/pip/cargo `install`) and does not apply to flake `github:org/repo` references — there is no equivalent automated tool for flake-input provenance. Verification below is manual: GitHub org reputation + commit/star history, cross-checked against the CONTAINER-PLATFORM research that already selected each one.

| Input | Source | Age / Activity | Stars/Downloads proxy | Disposition |
|-------|--------|-----------------|------------------------|-------------|
| `nixpkgs` | `github:NixOS/nixpkgs` | Official NixOS org, 10+ years | N/A — the canonical package set | Approved |
| `nixos-wsl` | `github:nix-community/NixOS-WSL` | `nix-community` org (the standard "vetted but not core NixOS" umbrella org), active, already used in spike 010 with a sha256-verified release | N/A (flake input, not registry pkg) | Approved — already validated in spike 010 |
| `rust-overlay` | `github:oxalica/rust-overlay` | Single well-known maintainer (oxalica), multi-year history, extremely widely used in the Nix/Rust ecosystem | N/A | Approved |
| `attic` (server + client) | `github:zhaofengli/attic`, packaged into nixpkgs as `services.atticd` | Crate labeled "WIP" on crates.io but the NixOS module is merged into nixpkgs proper and it's the CONTAINER-PLATFORM research's explicit D-01 pick | N/A | Approved — flag the "WIP" crate label in the runbook as a known maturity caveat, not a blocker |

**Packages removed due to slopcheck [SLOP] verdict:** none — slopcheck does not apply to flake inputs; no npm/pip/cargo packages are added by this phase.
**Packages flagged as suspicious [SUS]:** none.

*All flake-input claims above are `[CITED]` (official repos / nixpkgs option listings) rather than `[VERIFIED]` in the strict npm-registry sense, since no registry-verification tool exists for flake inputs. The planner should still pin exact commit/tag refs (not floating branches) for `nixos-wsl` and `rust-overlay` in `flake.lock`, which is what `flake.lock` does automatically on first `nix flake update`.*

## Architecture Patterns

### System Architecture Diagram

```
                         ┌─────────────────────────────┐
                         │   repo-root flake.nix        │
                         │   (nixpkgs=26.05, one lock)  │
                         └───────────────┬───────────────┘
                                         │
              ┌──────────────────────────┼──────────────────────────┐
              │                          │                          │
     devShells.default          nixosConfigurations         packages / checks
   (Rust+Node+Tauri deps    ┌──────┴───────┐          ┌───────┴────────┐
    + attic client +        │               │          │                │
    substrate build tools)  │  substrate    │          │ checks.<sys>.  │
              │              │  core module  │          │  seedBootTest  │
              │              │  (services,   │          │  (nixosTest,   │
     `nix develop`           │  users, dev   │          │  plain-VM      │
     on NixOS dev host       │  tooling —    │          │  variant)      │
                              │  defined ONCE)│          └───────┬────────┘
                              └──────┬────────┘                  │
                        ┌────────────┴────────────┐              │
                        │                          │              │
              WSL adapter variant          plain-VM adapter variant
              (nixos-wsl module +          (used ONLY by the
               nix-ld + vscode-server)      nixosTest above)
                        │
              nixosConfigurations.<name>
                .config.system.build.tarball
                        │
                        ▼
                 substrate.wsl image
              (what a real user's
               `wsl --import` consumes,
               Phase 10's concern)

CI (GitHub Actions, public repo, standard runners):
  push/PR ──▶ Job 1: nix-installer-action (KVM auto) ──▶ `nix flake check`
                                                          (evaluates flake,
                                                           builds seedBootTest
                                                           nixosTest in QEMU,
                                                           asserts curl on
                                                           loopback HTTP svc)
              Job 2: windows-latest ──▶ npm ci ──▶ rustup auto-picks
                      rust-toolchain.toml ──▶ npm run tauri build
              Job 3 (drift gate): reads rust-toolchain.toml + Node pin file
                      directly, compares against Job 2's resolved
                      `rustc --version` / `node --version` output ──▶ fail
                      on mismatch
              ALL GREEN ──▶ Job 4: attic push (CI's built closure ──▶
                      Attic server on EC2, backed by S3) = the publish step (D-03)
```

### Recommended Project Structure

```
/ (repo root)
├── flake.nix                  # devShells + packages + nixosConfigurations + checks
├── flake.lock                 # ONE lock for the whole repo
├── rust-toolchain.toml        # read by rustup (Windows) AND rust-overlay (flake)
├── .nvmrc                     # (or chosen Node pin file — read by flake AND actions/setup-node)
├── nix/
│   ├── substrate/
│   │   ├── core.nix           # shared module: services, users, dev tooling (D-15)
│   │   ├── wsl-variant.nix    # imports nixos-wsl module + core.nix
│   │   └── vm-variant.nix     # plain-VM module + core.nix, used only by the nixosTest
│   ├── checks/
│   │   └── seed-boot-test.nix # nixosTest: boots vm-variant, curls the placeholder service
│   └── lib.nix                # D-09's near-empty public lib/overlay surface
├── LICENSE                    # PolyForm Noncommercial 1.0.0 full text
├── .github/
│   └── workflows/
│       └── ci.yml              # (or split into ci.yml + windows.yml — Claude's discretion)
├── package.json                # existing — license field added (D-05)
├── src-tauri/Cargo.toml        # existing — license field added (D-05)
└── ...                          # existing src/, src-tauri/, sidecar/ untouched
```

### Pattern 1: Shared core module, two thin variants (D-15)

**What:** Define the substrate's services/users/dev-tooling in one `nix/substrate/core.nix` NixOS module. Two tiny wrapper modules each `imports = [ core.nix ]` plus exactly the adapter layer that differs: one adds `nixos-wsl.nixosModules.default` + WSL-specific settings, the other adds nothing WSL-specific and is what `nixosTest` boots under plain QEMU.

**When to use:** Any time CI needs to prove the same content real users get, without CI being able to actually run WSL (`nixosTest` boots QEMU-NixOS, not WSL — the honest, already-flagged gap this phase accepts).

**What breaks if the WSL module is imported into a plain nixosTest VM:** NixOS-WSL's module assumes a WSL2 kernel/interop layer (e.g., `wsl.interop`, `/init` as the WSL entrypoint instead of a normal bootloader, WSL-specific networking). A `nixosTest` VM boots via QEMU with a normal systemd-boot-style init path — importing the full `nixos-wsl.nixosModules.default` there will either fail evaluation (options referencing WSL-only mechanisms) or silently no-op in ways that don't test what matters. Keep the WSL adapter and the VM adapter as separate, non-overlapping import lists that both pull from the same `core.nix`.

**Example:**
```nix
# nix/substrate/core.nix
{ pkgs, ... }: {
  users.users.dev = { isNormalUser = true; extraGroups = [ "wheel" ]; };
  programs.nix-ld.enable = true;
  services.openssh.enable = true; # example dev-tooling surface
  # D-12 seed placeholder service lives here too, so both variants get it:
  systemd.services.seed-placeholder = {
    wantedBy = [ "multi-user.target" ];
    serviceConfig.ExecStart = "${pkgs.python3}/bin/python3 -m http.server 8080 --bind 127.0.0.1";
  };
}

# nix/substrate/wsl-variant.nix
{ nixos-wsl, ... }: {
  imports = [ ./core.nix nixos-wsl.nixosModules.default ];
  wsl.enable = true;
  wsl.defaultUser = "dev";
}

# nix/substrate/vm-variant.nix
{ ... }: {
  imports = [ ./core.nix ];
  # no WSL adapter — this is what nixosTest boots under QEMU
}
```
Source pattern: synthesized from NixOS-WSL's own flake.nix module-composition style [CITED: github.com/nix-community/NixOS-WSL/blob/main/flake.nix] plus standard NixOS module-sharing conventions (nix.dev, NixOS manual). No single official doc shows exactly this WSL/VM split — this is the phase's own novelty, called out honestly rather than presented as a copied recipe.

### Pattern 2: nixosTest in `checks` + asserted HTTP service (D-12)

**What:** A `pkgs.nixosTest` (or `pkgs.testers.runNixOSTest`) derivation wired into `checks.x86_64-linux.seedBootTest`, so `nix flake check` builds and runs it automatically.

**Example:**
```nix
# nix/checks/seed-boot-test.nix
{ pkgs, vmVariantConfig }:
pkgs.nixosTest {
  name = "seed-boot-test";
  nodes.machine = vmVariantConfig; # nix/substrate/vm-variant.nix's config
  testScript = ''
    machine.wait_for_unit("multi-user.target")
    machine.wait_for_unit("seed-placeholder.service")
    machine.wait_for_open_port(8080)
    machine.succeed("curl -sf http://127.0.0.1:8080/ ")
  '';
}
```
Source: `pkgs.nixosTest` / Python testing-driver pattern is documented NixOS manual mechanics [CITED: nixos.org NixOS manual, "Writing NixOS Tests"]; the `checks.<system>.<name>` wiring so `nix flake check` picks it up automatically is standard flake-output convention [CITED, cross-verified via WebSearch against multiple community examples, e.g. blakesmith.me/2024/03/02/running-nixos-tests-with-flakes.html].

### Pattern 3: rust-toolchain.toml as the single Rust pin (D-10)

**What:** One `rust-toolchain.toml` at repo root, read three ways: (1) rustup natively on `windows-latest` (zero config — any `cargo`/`rustup` invocation in that directory auto-installs/uses the pinned toolchain); (2) `rust-overlay`'s `fromRustupToolchainFile` in the flake; (3) the drift-gate job, which parses the file directly (no Nix eval needed for the comparison side).

**Example:**
```toml
# rust-toolchain.toml
[toolchain]
channel = "1.82.0"   # match whatever Tauri 2.x's MSRV / src-tauri/Cargo.toml currently requires — verify at plan time, do not guess a number here
components = ["rustfmt", "clippy"]
```
```nix
# in flake.nix
rustToolchain = pkgs.rust-bin.fromRustupToolchainFile ./rust-toolchain.toml;
```
Source: [CITED: github.com/oxalica/rust-overlay README + reference.md] for the Nix side; [CITED: rust-lang.github.io/rustup/overrides.html] for the file format and rustup's native auto-pickup behavior.

### Pattern 4: CI drift gate (D-10)

**What:** A CI step (can live in the `windows-latest` job itself, as a final step) that captures `rustc --version --verbose` and `node --version`, then compares the parsed values against `rust-toolchain.toml`'s `channel` and the Node pin file's content. Fail the job (and thus the whole run, satisfying D-11/D-12) on any mismatch.

**Example (illustrative, PowerShell since `windows-latest`):**
```yaml
      - name: Toolchain drift gate
        shell: bash
        run: |
          set -euo pipefail
          expected_rust=$(grep -oP '(?<=channel = ")[^"]+' rust-toolchain.toml)
          actual_rust=$(rustc --version | awk '{print $2}')
          expected_node=$(cat .nvmrc | tr -d '\n')
          actual_node=$(node --version | tr -d 'v')
          [[ "$actual_rust" == "$expected_rust"* ]] || { echo "Rust drift: expected $expected_rust, got $actual_rust"; exit 1; }
          [[ "$actual_node" == "$expected_node"* ]] || { echo "Node drift: expected $expected_node, got $actual_node"; exit 1; }
```
No single official source shows this exact recipe — it is bespoke assembly of two well-documented facts (rustup honors `rust-toolchain.toml` automatically; `actions/setup-node` reads a version file) rather than a copied pattern. Flag as `[ASSUMED]` shape, verify shell semantics at plan/implementation time.

### Pattern 5: Attic push as the publish gate (D-03)

**What:** After all three CI checks (flake check, seed nixosTest, windows-latest build) are green, a final job logs into Attic with a token from a GH Actions secret and pushes the built closure(s).

**Example:**
```yaml
      - name: Push to Attic cache
        if: success()
        run: |
          attic login sourcerer https://cache.sourcerer.example.org "${{ secrets.ATTIC_PUSH_TOKEN }}"
          attic push sourcerer-cache ./result   # or the relevant store paths
```
Client-side commands `attic login` / `attic push` / `attic use` [CITED: docs.attic.rs/tutorial.html]. Token minting is server-side via `atticd-atticadm make-token`, scoped to specific caches/permissions [CITED: docs.attic.rs].

### Anti-Patterns to Avoid

- **Importing the full `nixos-wsl` module into the `nixosTest` VM variant:** breaks evaluation or silently tests the wrong thing — see Pattern 1.
- **Cron-driven GC instead of `atticd`'s built-in GC config:** unnecessary extra moving part; `atticd` already runs GC on a configurable interval with per-cache retention — see Common Pitfalls.
- **Using `dtolnay/rust-toolchain` and assuming it reads `rust-toolchain.toml`:** it does not, by the action's own maintainer's design choice [CITED: github.com/actions-rs/toolchain issue #208 discussion, actions-rust-lang fork README]. Either skip any action and let bare rustup pick up the file, or explicitly choose an action documented to support toolchain files.
- **Floating flake inputs (`nixpkgs.url = "github:NixOS/nixpkgs/nixos-26.05"` without ever running `nix flake update` deliberately and committing the lock):** defeats the entire "one lock, reproducible months later" thesis (D-07, FOUND-01). The lock file is the actual pin; the branch name in the URL is just where updates come from.

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| S3-backed Nix binary cache server | A custom `nix-serve` + S3 sync script | `services.atticd` | Attic already does content-addressed dedup, chunking, JWT auth, per-cache GC/retention, and has a first-class NixOS module — a hand-rolled `nix-serve` variant would need to reimplement all of that with none of the community maintenance |
| GC / retention scheduling | A cron job calling `nix-collect-garbage` against the cache store | `atticd`'s internal `[garbage-collection]` config block (interval + `default-retention-period`) | It's a first-class server feature, not an external concern — D-02's "GC automation" is satisfied by config, not a script |
| Rust toolchain version resolution across Windows + Nix | A custom version-string parser/installer | `rustup` (native on Windows, already reads `rust-toolchain.toml`) + `rust-overlay`'s `fromRustupToolchainFile` (reads the identical file on the Nix side) | Both tools already implement "read this TOML, install/select that toolchain" — hand-rolling would duplicate two already-correct implementations and risk subtly disagreeing with either |
| NixOS-in-a-QEMU-VM test harness | Bespoke QEMU invocation + serial console scraping | `pkgs.nixosTest` / `pkgs.testers.runNixOSTest` | This is NixOS's own, extensively used integration-test framework — reinventing it means reinventing VM boot orchestration, the Python test-driver protocol, and `nix flake check` integration |

**Key insight:** every piece of this phase already has a first-class, actively maintained Nix-ecosystem tool. The actual work is wiring, not invention — which matches the phase's own "Descope trigger: none — assembly only" framing in ROADMAP.md.

## Common Pitfalls

### Pitfall 1: nixosTest boots QEMU-NixOS, not WSL — treat as an accepted gap, not a bug to fix here
**What goes wrong:** Someone treats a green seed nixosTest as proof the real `.wsl` image works under WSL2.
**Why it happens:** The shared-core-module pattern (D-15) makes the two variants look nearly identical, inviting the assumption they're equivalent.
**How to avoid:** Document explicitly (in the runbook and in the flake's own comments) that the nixosTest variant proves the *service contents*, not the WSL adapter layer. A thin real-WSL smoke test is explicitly Phase 10's concern per D-12/context, not this phase's.
**Warning signs:** Any plan task phrased as "prove the WSL image boots" using only the nixosTest — that's the VM variant, not the WSL variant.

### Pitfall 2: `nix-installer-action` now means Determinate Nix, not upstream — silent since 2026-01-01
**What goes wrong:** Docs, prior training data, or older blog posts assume `nix-installer-action` installs vanilla upstream Nix by default.
**Why it happens:** That was true before 2026-01-01; the action's default flipped and the upstream-Nix option was removed entirely, not just made non-default. [CITED: DeterminateSystems/nix-installer-action README, fetched 2026-08-04]
**How to avoid:** State explicitly in the flake/CI docs and the channel-maintenance runbook that CI's Nix is Determinate Nix. If a plan step wants upstream Nix specifically, it must choose a different installer action (e.g. `cachix/install-nix-action`) and accept losing the free default KVM enablement, or independently verify KVM availability.
**Warning signs:** A CI failure that references Determinate-specific behavior (e.g., lazy trees, its own flag surface) that wasn't anticipated.

### Pitfall 3: Attic's crate is labeled "WIP" — don't let that block D-01, but don't hide it either
**What goes wrong:** Either (a) treating Attic as fully "done" software with no operational caution, or (b) treating the WIP label as a reason to second-guess the already-locked D-01 decision.
**Why it happens:** crates.io shows the raw crate metadata without the ecosystem context that Attic is nonetheless the standard, actively-used self-hosted Nix cache (used by NixOS's own CI in places, documented in nix.dev tutorials, and already selected in the CONTAINER-PLATFORM research).
**How to avoid:** Note the maturity caveat once in FOUND-03's runbook (D-02) as an operational risk to watch — e.g., "pin the exact `attic-server` NixOS module version deployed; do not blind-upgrade on every nixpkgs bump without checking Attic's own changelog" — without re-litigating D-01 at plan time.
**Warning signs:** A plan or discussion trying to re-open the Attic-vs-Cachix-vs-hand-rolled question.

### Pitfall 4: `dtolnay/rust-toolchain` silently ignoring `rust-toolchain.toml`
**What goes wrong:** A plan adds `dtolnay/rust-toolchain@stable` to the `windows-latest` job expecting it to honor the repo's `rust-toolchain.toml`, and it instead installs whatever `stable` resolves to at the time, which can silently diverge from the pin (defeating the whole point of D-10's drift gate).
**Why it happens:** It's the most commonly cited Rust-toolchain-setup action in search results and tutorials, and many of those tutorials predate or ignore this specific gap.
**How to avoid:** Either omit any toolchain-setup action entirely and rely on bare rustup's native file pickup (simplest, recommended), or use an action explicitly documented to read toolchain files (`actions-rust-lang/setup-rust-toolchain`, or the community `dsherret/rust-toolchain-file` fork).
**Warning signs:** The drift gate (Pattern 4) is the actual safety net here — if this pitfall is hit, the drift gate should catch it, which is exactly why D-10 mandated the gate in the first place.

### Pitfall 5: GC misconfiguration silently disables itself
**What goes wrong:** `atticd`'s `default-retention-period` defaults to `0`, which **disables** time-based GC — a naive "I configured GC" belief without explicitly setting a nonzero retention period leaves the cache growing unbounded, directly contradicting D-02.
**Why it happens:** `0` reads as "immediate/aggressive GC" to an unfamiliar reader; it actually means "off."
**How to avoid:** Explicitly set `default-retention-period` (e.g. `"3 months"`, matching Attic's own documented default recommendation) in the `services.atticd` NixOS module config, and verify with `attic cache info <name>` that the configured cache shows a nonzero retention period after deployment.
**Warning signs:** Cache size growing without bound across nixpkgs bumps; `attic cache info` showing retention `0`/unset.

## Code Examples

### Attic client login + push (CI)
```bash
# Source: docs.attic.rs/tutorial.html
attic login sourcerer https://cache.example.org "$ATTIC_TOKEN"
attic use sourcerer-cache   # writes substituter + trusted-public-key into ~/.config/nix/nix.conf
attic push sourcerer-cache ./result
```

### Client substituter config (consuming machine, e.g. FOUND-01's "second machine")
```nix
# flake.nix nixConfig block — makes a fresh clone pull from the cache automatically
# on the FIRST `nix flake check`/`nix build`, before `attic use` has ever run
{
  nixConfig = {
    extra-substituters = [ "https://cache.example.org/sourcerer-cache" ];
    extra-trusted-public-keys = [ "sourcerer-cache:REPLACE-WITH-REAL-KEY=" ];
  };
  # ...rest of flake
}
```
Source: standard flake `nixConfig` mechanism [CITED: nix.dev / NixOS Nix manual — `nixConfig` in flakes]. This directly satisfies FOUND-01's "clone on a second machine... builds... without compiling" — the substituter is declared IN the flake itself, so a fresh clone gets it without any prior `attic use` setup step, as long as the user accepts the flake's `nixConfig` (Nix prompts once, or `--accept-flake-config`/global trusted-users config).

### rust-overlay reading rust-toolchain.toml
```nix
# Source: github.com/oxalica/rust-overlay reference.md
{
  inputs.rust-overlay.url = "github:oxalica/rust-overlay";
  outputs = { self, nixpkgs, rust-overlay, ... }:
    let
      pkgs = import nixpkgs {
        system = "x86_64-linux";
        overlays = [ rust-overlay.overlays.default ];
      };
      rustToolchain = pkgs.rust-bin.fromRustupToolchainFile ./rust-toolchain.toml;
    in {
      devShells.x86_64-linux.default = pkgs.mkShell {
        buildInputs = [ rustToolchain pkgs.nodejs pkgs.webkitgtk_4_1 pkgs.pkg-config
                         pkgs.librsvg pkgs.gtk3 pkgs.wrapGAppsHook4 ];
      };
    };
}
```
Source: synthesized from [CITED: wiki.nixos.org/wiki/Tauri] (buildInputs list: `librsvg` + `webkitgtk_4_1`; nativeBuildInputs: `pkg-config`, `wrapGAppsHook4`, `cargo`, `cargo-tauri`) plus oxalica's reference docs for the toolchain-file read.

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|---------------|--------|
| `nix-installer-action` installing upstream Nix by default, `determinate: true` opt-in | Determinate Nix by default; upstream install path removed | 2026-01-01 | CI's Nix flavor is Determinate Nix unless a different action is chosen — document this, don't assume upstream |
| `webkitgtk` (4.0/soup2) for Tauri on Linux | `webkitgtk_4_1` (soup3) | Tauri 2.0 (2024) | The devShell must reference `webkitgtk_4_1`, not the older attribute, or builds fail with missing `javascriptcoregtk` |
| NixOS-WSL `2511.x` release line | `2605.x` release line (tracks nixpkgs 26.05) | Concurrent with nixpkgs 26.05's 2026-05-30 release | Already validated in spike 010 at `2605.7.2`; this phase's flake input pin should track the same line |
| Attic crate a young/experimental project | Attic's NixOS module merged into nixpkgs proper (`services.atticd`) | Sometime before this research date (exact date not independently verified — [ASSUMED] based on MyNixOS listing it under the `nixpkgs` namespace) | Simplifies deployment: no separate `attic.nixosModules.atticd` flake import needed if the nixpkgs-bundled module is current enough; verify version parity at plan time |

**Deprecated/outdated:**
- `stock system.autoUpgrade` for update mechanics — already flagged as rejected in the upstream CONTAINER-PLATFORM research (Phase 11's concern, not this phase's, but the flake should not accidentally wire it in as a default).

## Assumptions Log

| # | Claim | Section | Risk if Wrong |
|---|-------|---------|---------------|
| A1 | `attic`'s NixOS module is merged into nixpkgs proper as of the `nixos-26.05` branch specifically (not just some more-recent unstable snapshot) | Standard Stack, State of the Art | If wrong, the flake needs `attic.url = "github:zhaofengli/attic"` as an explicit extra input importing `attic.nixosModules.atticd`, a small but real flake-topology change. Verify with a direct `nix eval` against the pinned nixpkgs at plan/implementation time before committing to "no extra input needed." |
| A2 | The exact `rust-toolchain.toml` `channel` value (a specific Rust version) needed for Tauri 2.x's current MSRV | Pattern 3 code example | Deliberately left as a placeholder (`1.82.0`) rather than asserted as fact — Tauri's MSRV moves between releases; the plan must check `src-tauri/Cargo.toml`'s `tauri = "2"` resolved version and Tauri's own docs at implementation time, not trust a number from this research pass. |
| A3 | `windows-latest`'s preinstalled `rustup` will, with zero extra GitHub Action, auto-install the toolchain pinned in `rust-toolchain.toml` on first `cargo`/`rustup` invocation | Pattern 3, Pattern 4, Pitfall 4 | This is asserted from rustup's documented override-file mechanics (a general rustup behavior, not Windows-runner-specific) plus community sourced claims that GitHub's runner images ship rustup — if the exact `windows-latest` image snapshot at implementation time lacks rustup or has it misconfigured, an explicit rustup-install step becomes necessary. Cheap to verify: the first CI run either works or doesn't, and the drift gate (Pattern 4) is the safety net either way. |
| A4 | GitHub's Nonprofit program specifically extends to unlimited/expanded Actions minutes on a public repo the same way the general "public repos get unlimited free minutes on GitHub-hosted runners" policy does | User Constraints (D-06), Common Pitfalls | Low risk regardless — D-04 already establishes "public repo" as the source of free unlimited Actions minutes independent of the nonprofit plan; the nonprofit plan (D-06) is about the org's paid-seat tier, not Actions minutes specifically. If the nonprofit specifics differ from the general public-repo policy, it doesn't change this phase's CI design, only possibly a cost line in the runbook. |

**If this table is empty:** N/A — table is populated; see above.

## Open Questions

1. **Exact AWS EC2 instance size + S3 bucket lifecycle policy for the Attic deployment**
   - What we know: D-01 locks "small EC2 instance" + S3 storage; the CONTAINER-PLATFORM-PLAN.md's P1 body budgeted "~$5-10/mo Attic-on-VPS" as a rough comparison point (that figure was for a generic VPS, not AWS-specific pricing, and predates the nonprofit-credits framing).
   - What's unclear: Actual instance type (t3.micro/t3.small are the obvious starting candidates for a low-traffic single-tenant cache server) and whether S3 lifecycle rules should independently expire objects, or whether Attic's own chunk-store GC is sufficient (Attic's GC operates at the Nix-cache-metadata level; whether it actually deletes the underlying S3 objects or just unlinks references needs a doc/behavior check before writing the runbook's cost model).
   - Recommendation: This is "Claude's discretion" per CONTEXT.md — size conservatively (t3.small or t3.micro) at plan time, and add an explicit runbook checklist item to verify Attic's GC actually frees S3 storage (not just DB rows) before publishing a cost figure as fact.

2. **Whether the flake needs `crane`/`importNpmLock` at all in THIS phase**
   - What we know: D-10 states "Nix consumes the existing `package-lock.json` (importNpmLock) and `Cargo.lock` (crane) directly for libraries," and CONTAINER-PLATFORM.md §4 names both as the standard tools.
   - What's unclear: This phase's Windows Tauri bundle is explicitly NOT Nix-built (scoped exception, conventional pinning) — so it's not obvious the flake needs to actually invoke `crane`/`importNpmLock` to produce a buildable package output in Phase 9 specifically, versus just having the *pin files* available for the drift gate. If no `packages.<system>.default` Tauri-app derivation is planned this phase, `crane`/`importNpmLock` may have nothing to build yet.
   - Recommendation: Treat D-10's wording as "the mechanism this repo will use whenever Nix does build these lockfile-driven pieces" (a standing convention), not necessarily "this phase must produce a working crane/importNpmLock derivation." The planner should decide, at task-breakdown time, whether Phase 9 needs an actual buildable `packages.default` beyond the substrate image, or whether wiring the *pin files* (rust-toolchain.toml, Node pin) is sufficient to satisfy D-10's letter this phase.

3. **Which Node version pin file format (`.nvmrc` vs `package.json` `engines` vs `.node-version`)**
   - What we know: `actions/setup-node`'s `node-version-file` input accepts any of these; the flake side needs to read whichever is chosen (trivial either way — `builtins.readFile`/`fromJSON`).
   - What's unclear: No CONTEXT.md decision picks one; it's listed under "Claude's Discretion" implicitly (D-10 says "a Node version pin," not which file).
   - Recommendation: `.nvmrc` is the simplest (single line, no JSON parsing on the Nix side) and is explicitly supported by `node-version-file`. Recommend `.nvmrc` unless the planner has a reason to prefer `package.json engines` (e.g., wanting a single file for both npm-ecosystem tooling and the pin).

## Environment Availability

| Dependency | Required By | Available | Version | Fallback |
|------------|------------|-----------|---------|----------|
| `nix` (CLI, flakes enabled) | Authoring/testing the flake locally | ✓ | 2.34.8 (upstream, not Determinate) [VERIFIED: `nix --version` + `nix show-config` on this host, 2026-08-04] | — |
| `git` | Wiring the new GitHub remote (D-04) | ✓ | 2.54.0 | — |
| `gh` (GitHub CLI) | Creating/wiring the public repo under the Deocracy org, managing secrets | ✓ | 2.97.0 | — |
| `attic` (client CLI) | Dev-shell cache push/pull, verifying the deployed cache | ✗ (not installed on this host outside a flake devShell) | — | Provided by the flake's `devShells.default` (D-16) once written; also installable ad hoc via `nix profile install nixpkgs#attic-client` for pre-flake verification |
| AWS account / EC2 + S3 access | Deploying the Attic server (D-01) | Not verified in this session — outside this tool's reach (no AWS CLI probe run; assume the user/org already has or will provision AWS access under Deocracy's nonprofit program) | — | None — this is a hard external dependency; the plan should include an explicit "AWS account + IAM credentials provisioned" precondition/checkpoint before the Attic deployment tasks |
| GitHub Deocracy org access (to wire the remote, D-04) | Repo creation/push | Not verified in this session (no `gh auth status` run against the Deocracy org specifically) | — | Plan should include a `checkpoint:human-verify` or early task confirming `gh auth status` shows access to the Deocracy org before attempting `gh repo create`/`git remote add` |

**Missing dependencies with no fallback:**
- AWS account/credentials for the Attic EC2+S3 deployment — must be confirmed available before Attic-deployment tasks are executed, not assumed.

**Missing dependencies with fallback:**
- `attic` CLI — resolved automatically once the flake's devShell exists; not a blocker to flake authoring itself.

## Validation Architecture

### Test Framework

This is an infrastructure/tooling phase, not application code — there is no `pytest`/`vitest`-style unit-test framework applicable to the flake/CI artifacts themselves. The existing app repo does have `vitest` configured (`npm test`, per `vitest.config.ts`) for application code, and `scripts/verify-fonts.mjs` as an existing verification gate — both should be invoked as part of the `windows-latest` job (per CONTEXT.md code_context notes) alongside the actual Tauri build, but they validate application code, not this phase's own deliverables.

This phase's "tests" ARE the CI checks themselves — each FOUND requirement maps directly to an automated, scriptable command, not a test file.

| Property | Value |
|----------|-------|
| Framework | N/A — infrastructure phase; validation IS the CI pipeline |
| Config file | `flake.nix` (checks output) + `.github/workflows/*.yml` |
| Quick run command (local, dev host) | `nix flake check` |
| Full suite command (CI) | The complete GitHub Actions workflow run (all jobs) |

### Phase Requirements → Test Map

| Req ID | Behavior | Test Type | Automated Command | File Exists? |
|--------|----------|-----------|-------------------|-------------|
| FOUND-01 | Clean clone → dev shell → substrate image builds from cache, no compiling | integration (manual clean-machine or CI-simulated) | `nix develop -c true && nix build .#nixosConfigurations.<substrate>.config.system.build.tarball --option substitute true` — inspect build log for zero local-build derivations (all fetched from the Attic substituter) | ❌ Wave 0 — needs the flake + a populated Attic cache to test against meaningfully |
| FOUND-02 | Red CI run (any of the 3 checks, or the drift gate) blocks publish | CI / smoke | `nix flake check` (flake+nixosTest leg); the `windows-latest` job (via `gh workflow run` or a real push); a deliberately-broken PR to prove red-blocks-merge | ❌ Wave 0 — needs `.github/workflows/*.yml` authored first |
| FOUND-03 | Cache hosting decided/costed/documented; GC configured | doc + config assertion | `attic cache info <cache-name>` (retention nonzero) + presence of the runbook doc file, reviewed manually (not automatable in the pytest sense) | ❌ Wave 0 — needs Attic deployed + the runbook file written |

### Sampling Rate

- **Per task commit:** `nix flake check` (fast leg — flake evaluation; full nixosTest build is the slow leg but still local before pushing)
- **Per wave merge:** Full CI workflow run (push to a branch / open a PR) — all three jobs plus the drift gate
- **Phase gate:** A genuinely green end-to-end CI run (all jobs) on the actual `.github/workflows/*.yml`, PLUS a real Attic cache push/pull round-trip proving FOUND-01's "no compiling on a second machine" claim, before `/gsd-verify-work`

### Wave 0 Gaps

- [ ] `flake.nix` itself — does not exist yet (repo is currently "nothing Nix-shaped exists yet" per CONTEXT.md code_context)
- [ ] `.github/workflows/*.yml` — does not exist yet
- [ ] `rust-toolchain.toml` / Node pin file — do not exist yet
- [ ] `nix/checks/seed-boot-test.nix` (the nixosTest) — does not exist yet
- [ ] A deployed, reachable Attic server to test push/pull against — does not exist yet; FOUND-01's "no compiling" claim can only be genuinely verified after this is stood up, so it is necessarily a late-wave verification step, not an early one

*(Framework install: none needed beyond the flake itself — Nix's own tooling (`nix flake check`, `nixosTest`) IS the framework for this phase.)*

## Security Domain

`workflow.security_enforcement` is `false` in `.planning/config.json` [VERIFIED: read directly, 2026-08-04] — per the output_format instructions, this section is omitted only if explicitly `false`, which it is. Given `security_enforcement: false`, this section is included in abbreviated form to flag the one load-bearing security-adjacent decision this phase makes (secrets handling for the cache push token), without running a full ASVS sweep.

| Concern | Applies | Standard Control |
|---------|---------|-------------------|
| CI secret handling (Attic push token) | yes | GitHub Actions encrypted secrets (`secrets.ATTIC_PUSH_TOKEN`), scoped Attic token (not the root token) minted via `atticd-atticadm make-token` with only push permission on the specific cache — never commit the token, never use the server's root JWT secret in CI |
| Public repo exposing `.planning/` history | yes (already accepted, D-04) | No control needed — this is an accepted, twice-flagged cost per D-04, not a defect to mitigate |
| Substrate placeholder service (D-12) exposure | yes | Bound to `127.0.0.1` only (loopback), matching the seed test's own assertion — never bind `0.0.0.0` even for a throwaway placeholder, to avoid setting a bad precedent copied into later phases |

## Sources

### Primary (HIGH confidence)
- github.com/DeterminateSystems/nix-installer-action (README, fetched 2026-08-04) — Determinate-Nix-by-default, upstream-Nix-removed-2026-01-01, KVM auto-enablement, example workflow YAML
- docs.attic.rs/tutorial.html (fetched 2026-08-04) — `attic login`/`push`/`use` commands, token/CI auth model
- docs.attic.rs/admin-guide/deployment/nixos.html (fetched 2026-08-04) — `services.atticd` module, JWT secret generation, reverse-proxy note
- wiki.nixos.org/wiki/Tauri — official NixOS wiki Tauri devShell buildInputs/nativeBuildInputs list (`webkitgtk_4_1`, `librsvg`, `wrapGAppsHook4`)
- github.com/oxalica/rust-overlay (README + docs/reference.md) — `fromRustupToolchainFile` usage
- rust-lang.github.io/rustup/overrides.html — `rust-toolchain.toml` format and rustup's native file-pickup behavior
- github.com/actions/setup-node (docs/advanced-usage.md) — `node-version-file` input, supported file types

### Secondary (MEDIUM confidence)
- github.com/nix-community/NixOS-WSL (flake.nix, building.html docs) — `.wsl`/tarball build output mechanics, flake input wiring
- crates.io/crates/attic — "WIP" maturity label on the crate (cross-referenced against the nixpkgs-merged module status, which is more current than the crate page)
- mynixos.com option listings for `services.atticd.package` — used to infer nixpkgs-proper packaging status (not independently confirmed via a direct `nix eval` against the pinned `nixos-26.05` branch — see Assumption A1)
- blakesmith.me/2024/03/02/running-nixos-tests-with-flakes.html — community-verified `checks.<system>.<name>` + `nix flake check` wiring pattern for nixosTest
- github.com/actions-rs/toolchain issue #208 — confirms `dtolnay/rust-toolchain` does not read `rust-toolchain.toml` by design

### Tertiary (LOW confidence)
- WebSearch-only cost/pricing figures for GitHub Actions minutes and general AWS EC2 sizing guidance — not independently verified against AWS's own pricing pages or GitHub's official nonprofit-program page this session; flagged in Open Questions rather than stated as fact in the Standard Stack table
- Exact date Attic's module was merged into nixpkgs proper — not independently pinned to a commit/date

## Metadata

**Confidence breakdown:**
- Standard stack: MEDIUM-HIGH — every tool choice is locked by CONTEXT.md decisions or backed by an official doc; version pins for `nixos-wsl`/`rust-overlay` need a `flake.lock`-time resolution rather than a hand-picked tag from this research
- Architecture: MEDIUM — the shared-core-module WSL/VM split and the drift-gate mechanics are original synthesis (no single copied recipe exists), built from individually well-documented pieces; flagged honestly rather than presented as a found pattern
- Pitfalls: HIGH — each pitfall traces to a specific, checkable fact (nix-installer-action's Jan 2026 cutover, dtolnay/rust-toolchain's documented non-support of toolchain files, atticd's retention-period-zero-means-off default)

**Research date:** 2026-08-04
**Valid until:** ~30 days for the Nix-ecosystem mechanics (stable, slow-moving); ~7 days for the `nix-installer-action`/Determinate-Nix default behavior specifically, since that's an actively-evolving vendor product surface — re-verify at plan/implementation time if this research is more than a couple weeks stale.
