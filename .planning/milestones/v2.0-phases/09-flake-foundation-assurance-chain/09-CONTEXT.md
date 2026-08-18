# Phase 9: Flake Foundation & Assurance Chain - Context

**Gathered:** 2026-08-04
**Status:** Ready for planning

<domain>
## Phase Boundary

One repo-root flake and one CI pipeline that both build worlds read — the Nix world (substrate
image, dev shell, later the engine and every catalog app) and the conventional Windows Tauri
build — plus a binary cache, such that nothing reaches a user machine that CI did not prove.

**In scope:** repo-root `flake.nix` (devShells + packages + nixosConfigurations skeleton, one
lock); the public GitHub repo under the Deocracy org with the PolyForm Noncommercial license;
CI (flake check + seed boot-test + `windows-latest` Tauri job + toolchain drift gate); Attic
binary cache on AWS wired push/pull; shared version pins; channel-maintenance runbook + cache
GC automation.

**Out of scope:** provisioning the substrate onto user machines (Phase 10 owns the `.wsl`
image install flow, seam, kill switch); the update channel and revert button (Phase 11); the
app-unit hardening template (Phase 15, informed by deferred Phase 8 — deliberately NOT
authored this phase); the store registry mechanics (Phase 16); the hosted embedding server
(no phase owns it yet — see Deferred).

</domain>

<decisions>
## Implementation Decisions

### Binary cache (FOUND-03)

- **D-01: Attic, self-hosted on AWS.** Attic server on a small EC2 instance; storage on S3
  (Attic's native backend — no compatibility caveat). Chosen over paid Cachix (vendor
  dependence for a production-critical path; the Garnix shutdown is the standing lesson) and
  over Hetzner (Deocracy's **nonprofit status** gets AWS credits/discounts, beating the
  Hetzner math). The cache stays a plain substituter URL — swappable by construction.
  **Sizing note (user, 2026-08-04): start small** — smallest viable instance to begin
  (upgrade path in the runbook); the cache only becomes production-critical at Phase 11.
- **D-02: Runbook + GC automation.** FOUND-03's runbook is written (nixpkgs bump cadence,
  red-channel response, retention policy, restore-from-scratch) AND Attic's GC/retention is
  configured so the cache does not grow unbounded by default. Phase 11 inherits a
  self-maintaining cache.
- **D-03: Publish = cache push.** Until Phase 11 builds the real channel, "blocks publish"
  means: the closure is pushed to Attic only after every CI check is green. The cache push is
  the publish step. (Claude's discretion, recorded.)

### Repo, license, visibility

- **D-04: Public repo under the existing Deocracy GitHub org** ("Deocracy sourcerer" is
  already up; this local repo currently has NO git remote — Phase 9 wires it). Public =
  free Actions minutes + KVM on standard runners. Accepted cost, flagged twice: the full git
  history including `.planning/` becomes public.
- **D-05: License = PolyForm Noncommercial 1.0.0** (user decision 2026-08-04 — **supersedes
  D-P1's plain AGPL** and an interim AGPL+Commons-Clause candidate). Existing lawyer-drafted
  template; publishable without a counsel gate. Two deltas from earlier intent, flagged and
  accepted: it prohibits commercial *use* entirely (not just selling), and it has **no
  copyleft** (clones need not share source). Deocracy retains all commercial rights as
  licensor — which the hosted-cloud milestone requires. A **CLA is required from the first
  outside contributor** or contributed code poisons the right to run the paid hosted tier.
- **D-06: GitHub plan:** Deocracy org on GitHub's nonprofit/Pro plan.

### Flake topology & public surface

- **D-07: flake.nix at this repo's root** — one flake, one lock. Downstream repos (Phase 13
  Databasise, Phase 16 store, all app repos) consume it as a **pinned flake input**.
- **D-08: nixpkgs pin = stable NixOS 26.05** (nixos-26.05 branch). Calm 6-month runbook
  rhythm: minor bumps within 26.05, one planned migration when 26.11 lands. NixOS-WSL's
  2605.x image line (spike-010-validated) is built from it. Note: Phase 8's Collabora facts
  were verified against *unstable* — re-verify `services.collabora-online` availability on
  26.05 when Phase 15 arrives.
- **D-09: Public surface = pinned-nixpkgs re-export + a named, deliberately near-empty
  lib/overlay output.** The slot app expressions will evaluate against exists from day one;
  Phase 15's compiler and Phase 16's submission format fill it. Makes TOOLS-02
  ("floating-ref installs impossible by construction") and STORE-01 ("expressions against
  the pinned flake") mechanically true later.

### Shared version pins (success criterion 4)

- **D-10: Native files + CI drift gate.** No invented format. `rust-toolchain.toml` (read by
  rustup on Windows AND by rust-overlay in the flake) + a Node version pin the flake and
  `actions/setup-node` both read; Nix consumes the existing `package-lock.json`
  (importNpmLock) and `Cargo.lock` (crane) directly for libraries. Plus one CI job that
  FAILS when the `windows-latest` job's resolved toolchain diverges from the flake's — the
  Windows Tauri bundle is the one artifact Nix can never verify, so that is where the check
  earns its keep. Neither pin file exists today — both worlds are currently floating.

### CI (FOUND-02)

- **D-11: All three checks, red blocks publish:** (1) `nix flake check` — Nix code valid,
  substrate image builds; (2) seed nixosTest — CI boots the built substrate image in a
  throwaway VM; (3) `windows-latest` conventional Tauri build. Plus D-10's drift gate.
- **D-12: The boot check asserts boots + a placeholder service answers** — multi-user target
  reached AND a trivial HTTP service inside the image answers on a loopback port. The seed
  check is a working miniature of Phase 13's real check (engine answers on loopback): the
  plumbing is proven in the exact shape it gets reused.
- **D-13 [informational]:** KVM/QEMU are CI-runner plumbing only — nothing you ship contains
  them. Public repo means KVM is available on standard runners; no verify item remains.

### Substrate image skeleton

- **D-14: Bootable + dev tooling baked in.** NixOS-WSL base module + nix-ld +
  vscode-server support so VS Code Remote-WSL works against the substrate from day one
  (matches the P1 plan's wording). Nothing user-facing. No app-unit hardening template this
  phase — with Phase 8 deferred there is no measured exemption set; authoring one now would
  encode guesses (roadmap note stands: provisional until Phase 15 re-decides).
- **D-15: One shared core module, two thin variants.** The substrate's contents (services,
  users, dev tooling) are defined once; the flake exposes the real WSL2 image target AND a
  plain-VM variant that the CI boot check uses. CI genuinely tests the same contents users
  get, differing only in the WSL adapter layer.

### Dev shell

- **D-16: One shell, both worlds.** `nix develop` provides substrate-build tools (nix build
  wrappers, attic client) AND the full Tauri toolchain — Rust via rust-toolchain.toml +
  rust-overlay, Node, tauri CLI, webkitgtk system libs — so the NixOS dev host develops the
  shell app entirely through the flake. FOUND-01's clone-on-a-second-machine test proves the
  whole story.

### Claude's Discretion

- Attic instance sizing, S3 bucket layout, token/key management for cache push (GH Actions
  secret vs OIDC).
- Exact seed placeholder service (any trivial HTTP responder).
- direnv/.envrc wiring for the dev shell.
- Flake output naming/layout beyond the decisions above.
- CI workflow file structure (one workflow vs several).

</decisions>

<direction_of_record>
## Direction of Record (reaches past this phase — recorded here, mechanics owned downstream)

**Update chain — substrate first.** The NixOS substrate is the FIRST link: CI proves closure
→ cache → substrate updates → the substrate drives the Tauri shell update. The substrate
closure pins the compatible shell version and is the version authority; Tauri performs its
own Windows-side install (Store auto-update / tauri-plugin-updater) but follows the
substrate. Host-OS detection is Tauri's job (warden side of the seam). **Mechanics owed
downstream:** Phase 10 — the transport seam interface carries a version/compat field from
day one; Phase 11 — the ordering logic (substrate updates, then shell follows).

**Per-part versioning + power-user config.** Apps update per-part with individual version
control, not one monolithic bump: base system = one CI-proven closure; each app carries its
own version on top. Power users (expected to be many) get a real config file pinning
per-part versions, with the Pi coding agent as the maintenance path for custom builds. A
custom overlay is **outside the CI guarantee** — "nothing reaches a user CI didn't prove"
scopes to the base closure + catalog apps; custom pins are validated locally at rebuild
time, with generation rollback + agent repair as the safety net. Phase 11's promise wording
must scope itself accordingly. Phase 9's share: the flake is structured so apps are
per-version addressable later and a user-config import point is not precluded — the lib
surface (D-09) is where both land.

**Multi-repo distribution.** Core repo (this one) + first-party non-core components each in
their own Deocracy-org repo + community components from authors' own repositories. The app
store is a **registry of repos + pinned revisions**: author pushes → Phase 16 pipeline
re-runs on that exact revision (validate → build → boot-test → score → cosign-sign) → only
then does the store advance its pin → user auto-update pulls the store pin, **never an
author repo's HEAD** (an author push must never reach user machines unvetted). Every app
repo is a flake consuming the core flake as a pinned input; first-party and community ride
the identical mechanism. Registry mechanics belong to Phases 15/16.

</direction_of_record>

<canonical_refs>
## Canonical References

**Downstream agents MUST read these before planning or implementing.**

### Milestone architecture
- `.planning/research/CONTAINER-PLATFORM.md` — architecture source of truth. §4 (Nix stack:
  build tooling, assurance chain, update model — this phase implements its P1 slice), §2
  (deployment modes / warden abstraction — why nothing may assume localhost). §5 partly
  superseded by the Nix-native decision.
- `.planning/research/CONTAINER-PLATFORM-PLAN.md` — Phase P1 body (deliverables list this
  phase executes). Stale OCI language in P0/P4/P5 bodies — ROADMAP.md + REQUIREMENTS.md win
  where they disagree.
- `.planning/research/AI-PROVIDER-ARCHITECTURE.md` — weights-are-data rule (never in the Nix
  store or binary cache); the hosted embedding server ownership gap (see Deferred).

### Prior decisions this phase builds on
- `.planning/phases/08-spike-k-nix-native-substrate-service/08-CONTEXT.md` — the Nix-native
  rule (D-01/D-02) and its rationale of record; Phase 8's deferral means no measured
  hardening exemption set exists (why D-14 authors no app-unit template).
- `.planning/spikes/010-nixos-wsl-substrate/README.md` — validated substrate mechanics and
  the NixOS-WSL 2605.7.2 line this phase's image builds from; WSL-version landmines are
  Phase 10's concern but inform the image target.
- `.planning/PROJECT.md` — Key Decisions table (D-P1 licensing is REVISED by this phase's
  D-05; D-P3 Axis-3 split; D-P4 Determinate Nix pin — the flake pins which Nix ships).
- `.planning/REQUIREMENTS.md` — FOUND-01/02/03 are this phase's requirements.

### External facts to verify at plan time (researcher)
- Attic (zhaofengli/attic) deployment on AWS EC2 + S3: NixOS module availability, GC
  config, chunking/dedup settings.
- rust-overlay `fromRustupToolchainFile` + crane + importNpmLock current usage on
  nixos-26.05.
- nixosTest driver invocation from `nix flake check` on GitHub Actions standard runners
  (KVM path via nix-installer-action).
- GitHub for Nonprofits plan mechanics for the Deocracy org.

</canonical_refs>

<code_context>
## Existing Code Insights

### Reusable Assets
- **Nothing Nix-shaped exists yet:** the repo has no `.nix` files, no `.github/` directory,
  no `LICENSE`, no git remote, no `rust-toolchain.toml`, no Node `engines` pin. Everything
  this phase ships is greenfield on top of a working Tauri/React tree.
- **Existing lockfiles are the pin source of truth:** `package.json` (react 18.2.0,
  dockview-core 2.0.0 exact) + `package-lock.json` + `src-tauri/Cargo.lock` — D-10 consumes
  them as-is via importNpmLock/crane; do not restructure them.
- **`scripts/verify-fonts.mjs` + `npm test` (vitest)** — existing verification commands the
  `windows-latest` CI job should run in addition to the Tauri build.

### Established Patterns
- **Spike conventions** (`.planning/spikes/CONVENTIONS.md`) bind spike work, not this phase —
  but the WSL_UTF8/PIPESTATUS discipline matters again in Phase 10, not here (this phase
  runs on the NixOS dev host).
- **No worktrees on the shell tree** (Track 1 serial rule) — Phase 9 executes alone.

### Integration Points
- **The shell app tree is untouched** except for: `LICENSE` (new), `rust-toolchain.toml`
  (new), possibly a Node pin file, `.github/workflows/` (new), `flake.nix`/`flake.lock`/
  `nix/` (new). No `src/` or `src-tauri/` source changes expected; `tauri.conf.json` is NOT
  this phase's concern (csp stays Phase 10).
- **Execution host: NixOS dev host** for everything; the `windows-latest` CI job is the only
  Windows leg and runs on GitHub's runners, not the Windows box.

</code_context>

<specifics>
## Specific Ideas

- The user's framing for the license journey: copyleft-the-architecture was the original
  goal; established as legally impossible (idea/expression dichotomy — no license binds
  non-licensees; patents/trade secret are the only instruments for "the way it works").
  PolyForm NC 1.0.0 is the landing point: free noncommercial use, all commercial rights
  retained by Deocracy.
- "Deocracy sourcerer is already up" — the GitHub org/namespace exists; wire, don't create.
- AWS + GitHub Pro chosen specifically because **Deocracy's nonprofit status makes them
  cheaper or free** — cost reasoning should reference the nonprofit programs, not list
  prices.
- The user thinks in terms of the agent-authored-app loop: Pi builds apps inside Sourcerer,
  pushes them to a repo, the store distributes them to other users. Phase 9's flake surface
  (D-09) is the first brick of that loop and should be documented as such.

</specifics>

<deferred>
## Deferred Ideas

- **Hosted embedding server has no owning phase.** AI-PROVIDER-ARCHITECTURE.md names it "a
  second standing production service" with FOUND-03-like cost/runbook treatment, but no
  roadmap phase stands it up; Phases 13/14/15 consume it. Must be resolved (via `/gsd-phase`)
  before Phase 14 renders live engine-backed data.
- **CUDA-as-a-service / rented GPU for the hosted AI tier** — would revise
  AI-PROVIDER-ARCHITECTURE.md's "on its own box" wording; belongs to whichever phase ends up
  owning the embedding server. (FAISS was mentioned; note Databasise's vector path is Cozo —
  reconcile there, not here.)
- **Nightly channel-green CI job** (rebuild against bumped nixpkgs to catch channel rot
  early) — discussed, not selected for Phase 9's CI slate; natural Phase 11 addition when
  the channel becomes real.
- **User-facing local system config** (editable configuration.nix / blessed overlay) is NOT
  the customization path — superseded by the direction of record (agent-authored apps +
  per-part version pinning + agent-maintained power-user config). Any future request for a
  raw local escape hatch reopens D-P3.
- **Counsel review of PolyForm NC fit** (especially the CLA and the noncommercial boundary
  for university/scholar users) — advisable before Phase 12 Store submission / Phase 16
  store opening; not a Phase 9 gate.

</deferred>

---

*Phase: 9-Flake Foundation & Assurance Chain*
*Context gathered: 2026-08-04*
