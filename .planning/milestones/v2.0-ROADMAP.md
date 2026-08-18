# Roadmap: Sourcerer (Desktop Shell)

## Milestones

- ✅ **v1.0 Desktop Shell MVP** — Phases 1-7 (shipped 2026-07-14) — [archive](milestones/v1.0-ROADMAP.md)
- 🚧 **v2.0 Container Platform** — Phases 8-19 (in progress)

## Phases

**Phase Numbering:**

- Integer phases (8, 9, 10…): Planned milestone work, continuing from v1.0's Phase 7
- Decimal phases (10.1, 10.2): Urgent insertions (marked with INSERTED)

Each v2.0 phase carries its original P-label from `.planning/research/CONTAINER-PLATFORM-PLAN.md` for traceability.

<details>
<summary>✅ v1.0 Desktop Shell MVP (Phases 1-7) — SHIPPED 2026-07-14</summary>

- [x] Phase 1: Shell Foundation (3/3 plans) — completed 2026-07-07
- [x] Phase 2: Workspace Core (6/6 plans) — completed 2026-07-08
- [x] Phase 3: Persistence & Layouts (5/5 plans) — completed 2026-07-10
- [x] Phase 4: Applet Framework (5/5 plans) — completed 2026-07-10
- [x] Phase 5: Notes Applet (2/2 plans) — completed 2026-07-13
- [x] Phase 6: Dashboard Assistant & Home (8/8 plans, incl. 2 UAT gap-closure) — completed 2026-07-14
- [x] Phase 7: Assistant Harness Core (6/6 plans) — completed 2026-07-08

Full phase details: [milestones/v1.0-ROADMAP.md](milestones/v1.0-ROADMAP.md)

</details>

### 🚧 v2.0 Container Platform (In Progress)

**Milestone Goal:** A non-technical scholar on a clean Windows 11 machine installs Sourcerer from the Microsoft Store (or signed site installer), gets a working NixOS-WSL substrate after at most one reboot, sees engine-backed functionality live, installs catalog apps as hardened Nix-native systemd units with visible security scores, reverts a bad update with one button, and migrates their whole environment to a second PC via manifest + data export.

- [~] **Phase 8: Spike K — Nix-Native Substrate Service** *(P0 remainder)* — **DEFERRED 2026-08-05** (planned, not executed). Spike-shaped work that was mis-slotted as a phase; parked with its plans intact. Re-decide at Phase 15: run it then, or absorb the hardening question into the compiler's own design.
- [ ] **Phase 9: Flake Foundation & Assurance Chain** *(P1)* - Repo-root flake, CI, binary cache — nothing publishes that CI didn't prove
- [ ] **Phase 10: Substrate Provisioning & Warden Seam** *(P2)* - Clean Windows machine → working substrate in ≤1 reboot, behind a transport seam, with a real CSP
- [ ] **Phase 11: Update Channel & Revert** *(P2b)* - CI-gated channel publishes, one-button rollback, bounded generations
- [ ] **Phase 12: Distribution Packaging** *(P2c)* - Microsoft Store (MSIX) and/or signed site installer on clean hardware
- [ ] **Phase 13: Engine Services in Substrate** *(P3a — Track 2, Databasise repo)* - Databasise packaged as a hardened NixOS module answering REST + MCP on loopback
- [ ] **Phase 14: Harness Relocation & First Engine-Backed Applet** *(P3b)* - Assistant runs substrate-side; a wiki read-view renders live engine data
- [ ] **Phase 15: App Layer v1** *(P4)* - Manifest→NixOS-module compiler, catalog apps in panes, enforced permissions, visible security scores
- [ ] **Phase 16: Community Store Pipeline** *(P5 — Track 2, store repo)* - Signed, CVE-gated, unattended submission pipeline (not publicly opened until Phase 19)
- [ ] **Phase 17: Tools Tier & Environment Portability** *(P6)* - `nix profile` tool tier and a "Move to another PC" export/import wizard
- [ ] **Phase 18: Permission Grants UI & Portal File Access** *(P7)* - View/revoke per-app grants; runtime folder grants scoped to the picked path
- [ ] **Phase 19: Hardening Close & Milestone Audit** *(P8)* - Exposure-score CI gate, app-unit fleet security audit, store-opening decision, milestone audit

## Phase Details

### Phase 8: Spike K — Nix-Native Substrate Service *(DEFERRED)*

> **DEFERRED 2026-08-05 — planned but not executed.** This is spike-shaped work that was
> mis-slotted as a roadmap phase: it ships no application code, and its only output is evidence
> for Phase 15's compiler and Phase 19's audit. Rather than spend a Windows-box day on it now,
> it is parked **with all planning artifacts intact** — 5 plans, RESEARCH, PATTERNS, VALIDATION
> and CONTEXT are committed and checker-passed, so whenever it runs it starts at execute, not at
> discuss.
>
> **Deliberately no downstream edits.** Phase 15 and Phase 19 keep their existing wording. This
> deferred entry plus SPIKE-01's `Deferred` status in REQUIREMENTS.md are the breadcrumb —
> Phase 15's discuss-phase must re-open the question and decide explicitly: run the spike first,
> or derive the hardening directives as part of building the compiler.
>
> **Not renumbered.** Phases 9–19 keep their numbers; the gap is intentional. Those numbers are
> referenced throughout PROJECT.md, the CONTAINER-PLATFORM research docs, and phase CONTEXT
> files ("Phase 9 owns the flake", "Phase 15 owns the compiler", "Phase 19's audit").
>
> When it does run, it becomes spike `012-nix-native-substrate-service` under
> `.planning/spikes/` — no MANIFEST row is added until there is a verdict to record.
>
> **One correction already banked, for free:** research overturned this phase's own D-09. The
> nixpkgs build passes `--disable-setcap`, forcing `coolwsd` onto its unprivileged
> user+mount-namespace jailing path, so `CapabilityBoundingSet` is probably *not* the blocker —
> `RestrictNamespaces` and `SystemCallFilter` are. Whoever picks this up should not re-derive
> that from scratch.

**Original plan label**: P0 (spike battery remainder — the last pending spike)
**Goal**: Prove that a real nixpkgs service survives the §7 hardening baseline inside WSL2 and serves a UI reachable from Windows, before Phase 15 commits to the manifest→unit compiler
**Depends on**: Nothing (v1.0 complete; spikes 010/011 already validated)
**Execution host**: **Windows box** — the substrate distro lives in WSL2. Re-import NixOS-WSL 2605.7.2 first (local image deleted 2026-08-03; verify sha256 `e7180ad555fdcb8e1e057e2ef056de467603a5e502ff8531053738371be3f6b9` recorded in `spikes/010-nixos-wsl-substrate/README.md`); re-verify the `SourcererSpike` distro is registered and warm, otherwise budget the re-import leg.
**Requirements**: SPIKE-01
**Success Criteria** (what must be TRUE):

  1. `services.collabora-online` runs inside the substrate distro from the binary cache, with no compiling and no OCI image
  2. The full §7 hardening baseline is applied first, then relaxed one directive at a time, with every concession and its reason recorded
  3. The Collabora web UI is reachable on 127.0.0.1 from the Windows host
  4. The resulting exemption set is written down so Phase 15's compiler and Phase 19's audit are built on evidence, not guesswork

**Notes**: **Rescoped 2026-08-04** — the original "engine-less OCI unit" framing was obsoleted by the Nix-native decision (see PROJECT.md Key Decisions). `collabora-online` 25.04.9-4 is in nixpkgs with a full `services.collabora-online` module and is cached (13.8 MB download / 42 MB unpacked, vs ~1.5–2 GB for the CODE image), so no image is involved. The real remaining risk moved: `coolwsd` sandboxes itself, forking jailed LibreOffice kernels via its own chroot + namespaces, which collides with `DynamicUser`/`ProtectSystem=strict`/`CapabilityBoundingSet`/`SystemCallFilter` — and WSL2's kernel is not a normal kernel. That collision is this spike's kill-question. Collabora only; Jupyter and the multiwebview pane leg belong to Phase 15.
**Plans**: 5 plans, ready to execute (strictly sequential — one physical Windows host, one distro, one service; the ladder depends on the plan-02 control measurement)
**Wave 1**

- [ ] 08-01-PLAN.md — Substrate precondition + driver scaffold (WSL version gate, conditional sha256-verified re-import, unprivileged-userns pre-flight)

**Wave 2** *(blocked on Wave 1 completion)*

- [ ] 08-02-PLAN.md — Stock service from cache: substitution proof, unhardened control measurement, package provenance

**Wave 3** *(blocked on Wave 2 completion)*

- [ ] 08-03-PLAN.md — The hardening ladder: full §7 baseline as rung 0, then one directive per rung, bounded and resumable with a named stop condition

**Wave 4** *(blocked on Wave 3 completion)*

- [ ] 08-04-PLAN.md — Declarative proof pass (drop-in torn down first) + Windows-side curl/browser reachability

**Wave 5** *(blocked on Wave 4 completion)*

- [ ] 08-05-PLAN.md — Findings: spike 012 README with the exemption set, Investigation Trail, MANIFEST row, and downstream doc updates

### Phase 9: Flake Foundation & Assurance Chain

**Original plan label**: P1 · size M
**Goal**: One flake and one CI pipeline that both build worlds read, so nothing reaches a user machine that CI did not prove
**Depends on**: Nothing. *(Previously listed Phase 8 — dropped 2026-08-05 when Phase 8 was deferred. That dependency was always soft: Spike K "informs nothing structurally", it only meant K's findings would land before the flake's app-unit skeleton. With K deferred, the app-unit skeleton is authored without a measured exemption set; the hardening directives it assumes are provisional until Phase 15 re-decides.)*
**Execution host**: **NixOS dev host** (flake authoring, devShells, cache wiring). CI runs on GitHub Actions; the `windows-latest` Tauri job is the standing regression net between hands-on Windows sessions.
**Requirements**: FOUND-01, FOUND-02, FOUND-03
**Success Criteria** (what must be TRUE):

  1. A developer clones the repo on a second machine, enters the dev shell, and builds the substrate image entirely from the binary cache with no compiling
  2. A red CI run (flake check, seed nixosTest, or `windows-latest` Tauri job) blocks publish
  3. Binary-cache hosting is chosen and costed, with a written channel-maintenance runbook
  4. Shared version pins exist that both the Nix world and the conventional Windows Tauri build read

**Notes**: The cache becomes a **production service** from Phase 11 on — Cachix free (5 GB) will not hold an ML-stack closure across versions; budget paid Cachix or Attic-on-VPS from the start. Keep CI trivially portable (plain GH Actions + swappable cache) — Garnix died July 2026.
**Descope trigger**: none — assembly only; novelty found here is itself a finding.
**Plans**: 7 plans in 5 waves
Plans:
**Wave 1**

- [x] 09-01-PLAN.md — License (PolyForm NC 1.0.0) + manifest license fields + Nix/direnv gitignore [wave 1]
- [x] 09-02-PLAN.md — Version pins (rust-toolchain.toml, .nvmrc) + repo-root flake: dev shell, lib/overlay surface, lock [wave 1]

**Wave 2** *(blocked on Wave 1 completion)*

- [x] 09-03-PLAN.md — Substrate core module + WSL/VM variants + seed nixosTest in flake checks [wave 2]

**Wave 3** *(blocked on Wave 2 completion)*

- [x] 09-04-PLAN.md — Publish repo under Deocracy org + CI (flake check, windows-latest Tauri, drift gate) [wave 3, checkpoint]
- [x] 09-05-PLAN.md — Attic cache on Deocracy's ohio1 prod box (pivoted from AWS EC2+S3), nonzero GC retention, scoped CI push token [wave 3, checkpoint]

**Wave 4** *(blocked on Wave 3 completion)*

- [x] 09-06-PLAN.md — Cache substituter in flake + publish job gated on green + channel-maintenance runbook [wave 4]

**Wave 5** *(blocked on Wave 4 completion)*

- [ ] 09-07-PLAN.md — Standing cold-build gate (FOUND-01) + red-blocks-publish evidence (FOUND-02) [wave 5]

### Phase 10: Substrate Provisioning & Warden Seam

**Original plan label**: P2 · size M
**Goal**: A non-technical user on a clean Windows 11 machine ends up with a working, killable substrate that the shell only ever touches through one seam
**Depends on**: Phase 9
**Execution host**: **Windows box** — provisioning UAT requires clean Windows 11 hardware, elevated `wsl --install`, and a physical reboot. Image build and seam code are authored on the NixOS dev host.
**Requirements**: SUB-01, SUB-02, SUB-03, SUB-04, SUB-05, SUB-06
**Success Criteria** (what must be TRUE):

  1. A clean Windows 11 machine reaches a working substrate in ≤3 interactions, zero terminal exposure, ≤10 min excluding downloads, and at most one reboot
  2. User data survives a distro re-import (it lives on a separate mount from the system image)
  3. No shell code reaches the substrate except through the substrate-connection transport seam, whose interface includes the auth surface (implementation stubbed)
  4. The user can kill the substrate from a kill switch in the shell
  5. Secrets flow end-to-end via sops-nix at rest + systemd `LoadCredential`, and the app ships a real `security.csp`

**Notes**: **The v1.0 `csp: null` acceptance expires here by construction** — `src-tauri/tauri.conf.json` still reads `"csp": null`, and multiwebview panes plus container-served UIs break the written condition Phase 1 accepted it under. Carry spike 010's landmines: pre-create the distro storage parent before `wsl --import`; enforce a minimum WSL version (2605 images fail on WSL 2.3.26, fine on 2.7.11); drive `wsl --update` through an explicit elevated flow (stalls invisibly behind UAC; concurrent paths collide with error 1618). BIOS-virtualization-off is the one unfixable failure → guided help screen.
**Descope trigger**: provisioning UX unachievable in bounds → guided semi-manual first-run accepted for v1, tracked as debt.
**Plans**: TBD
**UI hint**: yes

### Phase 11: Update Channel & Revert

**Original plan label**: P2b · size M
**Goal**: Users get updates that CI proved, and can undo a bad one with a single button
**Depends on**: Phase 10
**Execution host**: **NixOS dev host** for channel/CI mechanics; **Windows box** for the live switch/rollback UAT inside the real substrate.
**Requirements**: UPD-01, UPD-02, UPD-03
**Success Criteria** (what must be TRUE):

  1. A deliberately broken publish is caught by CI and never reaches a user machine
  2. A bad-but-published update is reverted by one visible "Revert last update" button (`--rollback`)
  3. Generations stay bounded on disk via `nix.gc.automatic` plus a retention policy

**Notes**: Consumes spike A's extended updater leg (measured: externally-driven `nixos-rebuild switch` 109 s, rollback 6 s). Phase 9's channel-maintenance runbook becomes practice here. Binary cache is a production service from this phase on.
**Descope trigger**: app-driven switch proves unreliable despite spike A → adopt comin (pull-based prior art) instead of in-app orchestration.
**Plans**: TBD
**UI hint**: yes

### Phase 12: Distribution Packaging

**Original plan label**: P2c · size S–M
**Goal**: The install claim is literal — a real user installs from the Microsoft Store or a signed site download without hitting a SmartScreen wall
**Depends on**: Phase 11
**Execution host**: **Windows box** — MSIX packaging, Partner Center identity, Windows code signing, and clean-hardware install tests are all Windows-only. Spike J (MSIX driving elevated `wsl --install` + reboot-resume + `wsl --import` from package context) runs here first and sets this phase's shape.
**Requirements**: DIST-01, DIST-02
**Success Criteria** (what must be TRUE):

  1. Either a Microsoft Store (MSIX) install succeeds on clean hardware, or spike J's failure triggers the explicit, documented v1 descope to site-only
  2. A signed site installer (EV cert / Azure Trusted Signing) installs on clean hardware with no SmartScreen wall
  3. The end-state install wording in PROJECT.md matches whichever channel actually shipped

**Notes**: Windows code signing is a real standing cost line item — without it the non-technical promise dies off-Store. Store certification queues are measured in days; this phase is wait-dominated.
**Descope trigger**: Store certification stalls > 2 iterations → ship site-signed, queue Store as follow-up milestone work, update the end-state wording honestly.
**Plans**: TBD

### Phase 13: Engine Services in Substrate

**Original plan label**: P3a · size L · **Track 2 — separate repo (Databasise), runs parallel to Phases 10/11**
**Goal**: The Databasise engine runs inside the substrate as a hardened NixOS module, proving the first "app" (a trusted one, ours) through every seam before untrusted code arrives
**Depends on**: Phase 9 (flake + CI); parallel to Phases 10-11, joins the shell track at Phase 14
**Execution host**: **NixOS dev host** — Nix packaging and nixosTest (QEMU) are native here. One thin real-WSL smoke test runs on the Windows box, since nixosTest boots QEMU-NixOS, not WSL.
**Requirements**: ENG-01
**Success Criteria** (what must be TRUE):

  1. Databasise is packaged as a NixOS module with the §7 hardening baseline and starts inside the substrate
  2. REST and MCP answer on loopback inside the substrate
  3. A substrate-integration nixosTest asserts the engine endpoints and goes green; endpoint-contract breakage fails CI

**Notes**: **LightRAG is NOT a separate dependency** — it was disassembled into Databasise's own code; only vendored remnants remain, nothing to package separately. **OCR is NOT part of the engine closure** — it ships later as an installable applet through the Phase 15 app layer. Track 2 lives in the Databasise repo, so the no-worktrees rule (which binds only the shell tree) does not serialize it.
**Descope trigger** (mandatory): any component fighting uv2nix > 3 days → run it as a digest-pinned engine-less OCI unit, no appeal. **This is the sole surviving OCI path in v2.0** under the 2026-08-04 Nix-native decision — first-party only, requires a written reason, and is tracked as debt to be replaced.
**AI providers**: read `.planning/research/AI-PROVIDER-ARCHITECTURE.md` before planning — embeddings are hosted-default (open-source, self-hosted, own box) with the same pinned model as the local tier; check whether Databasise's embedding path can ride `onnxruntime`/`llama-cpp` before committing to the uv2nix fight.
**Plans**: TBD

### Phase 14: Harness Relocation & First Engine-Backed Applet

**Original plan label**: P3b · size L-
**Goal**: Everything the user already had keeps working with the engine and harness substrate-side, and the first live engine-backed data appears in an applet
**Depends on**: Phase 10 (seam) + Phase 13 (engine) — this is where the two tracks join
**Execution host**: **Windows box** — the harness and engine now run inside the WSL substrate; the applet/assistant UAT exercises the live substrate.
**Requirements**: ENG-02, ENG-03, ENG-04
**Success Criteria** (what must be TRUE):

  1. All existing applet and assistant functionality works with the harness running substrate-side, with `host.ai()` unchanged applet-side
  2. The Windows-side Python sidecar is retired
  3. A wiki read-view applet renders live Databasise data through the seam
  4. The user can open a substrate status panel showing services, versions, and the current generation

**Notes**: Re-read the `phase02-phase07-ownership-boundary` memory before planning this phase. The status panel deliberately pulls a Phase 18 slice forward — it is the cheapest visible value and the debugging UI every later phase wants. **AI providers**: `host.ai()` grows a provider concept here (hosted-default / local-override per capability) — read `.planning/research/AI-PROVIDER-ARCHITECTURE.md`.
**Descope trigger**: harness relocation destabilizes assistant UX → harness stays host-side one more phase (the seam permits it), relocation re-queued; the wiki slice ships regardless.
**Plans**: TBD
**UI hint**: yes

### Phase 15: App Layer v1

**Original plan label**: P4 · size L
**Goal**: A user installs a real app from the catalog, uses it in a pane, and the permissions it declared are actually enforced by the kernel
**Depends on**: Phase 12 (shell track) + Phase 14 (engine proved through the seams)
**Execution host**: **NixOS dev host** for the manifest schema, compiler, and enforcement tests (nixosTest asserts netns and `MemoryMax`); **Windows box** for the pane UAT — including spike 011's still-open fractional-DPI retest (its `spike011-multiwebview.exe` was preserved for exactly this).
**Requirements**: APP-01, APP-02, APP-03, APP-04
**Success Criteria** (what must be TRUE):

  1. A manifest (identity, UI port, permissions, resources, autostart) compiles to one hardened systemd unit from a **nixpkgs-packaged service** — no OCI branch (Nix-native decision 2026-08-04); the baseline and its exemption set come from Phase 8's recorded evidence; a headless CLI install path exists
  2. The user installs Collabora from the catalog, edits a document in a pane, uninstalls, and a generation rollback restores the prior app set
  3. A `network:none` app demonstrably cannot reach the internet (asserted via the unit's netns) and a unit's `MemoryMax` demonstrably binds
  4. The user sees a computed security score at install time that honestly reflects nspawn/systemd depth (not gVisor)

**Notes**: Structurally pre-ordered — **compiler and enforcement waves complete and are testable BEFORE any UI wave**. `gpu` is EXCLUDED from the v1 manifest schema (deferred to Platform v2's P14). Collabora's WOPI host is a real plan item, not a parenthetical. App install/update is a cached rebuild — a download, given Phase 9's cache. **Nix-native (2026-08-04):** both catalog apps are already packaged (`services.collabora-online` 25.04.9-4 cached at 13.8 MB; `services.jupyter`), so the compiler has one branch and no image path. Jupyter and spike 011's still-open fractional-DPI pane retest were both deferred here from Phase 8. **AI providers**: Ollama is a planned catalog app (one-click, GPU variant selection, multi-GB runtime weight download → the manifest schema needs a declared-size fetch-and-verify concept, decided at schema design) — read `.planning/research/AI-PROVIDER-ARCHITECTURE.md`.
**Descope trigger**: a catalog service resists the hardening baseline → widen its documented exemption set for that app class and record why (per Phase 8's method), rather than dropping the baseline globally. An app that is not in nixpkgs waits until it is packaged — it does not get an ad-hoc image path.
**Plans**: TBD
**UI hint**: yes

### Phase 16: Community Store Pipeline

**Original plan label**: P5 · size M · **Track 2 — separate repo, runs parallel to Phases 17/18**
**Goal**: A community-authored package can go from submission to signed, CVE-scanned, boot-tested artifact without a human in the loop
**Depends on**: Phase 15 (reuses its compiler wholesale — that reuse is the M sizing condition)
**Execution host**: **NixOS dev host** — store repo, submission CI, nixosTest, cosign signing.
**Requirements**: STORE-01, STORE-02, STORE-03, STORE-04
**Success Criteria** (what must be TRUE):

  1. A package authored by a pinned-model + fixed-prompt fixture passes the pipeline end-to-end unattended: validate → build module → boot nixosTest → score → cosign-sign (the OCI-unit normalization stage is removed — Nix-native decision 2026-08-04)
  2. Unsigned, score-floored, and privileged submissions auto-reject; a submission carrying an OCI image reference auto-rejects by construction, since the format has no field for one
  3. A store ops runbook plus automation exists for CVE re-scans of approved images, digest re-pin cadence, and submission triage
  4. The store format ships but is **not publicly opened** — opening waits on Phase 19's audit

**Notes**: D-P2 stands — privileged submissions are auto-rejected with no appeal in v1. Single cosign key, no rotation in v1 (explicit). **Nix-native (2026-08-04):** the OCI-unit normalization stage is gone with the image path — a submission is a Nix expression against the pinned flake plus a manifest, so entrypoint/env/healthcheck semantics are declared, never inferred from image config. Compose is likewise gone as a compatibility tier. This narrows what a community author can submit, which raises the authoring bar — expect the catalog to stay first-party-curated longer than the original plan assumed, and treat that as the design, not a descope.
**Descope trigger** (mandatory): pipeline can't pass the LLM fixture unattended → v1 descopes to a first-party-curated catalog only; community store moves to Platform v2.
**Plans**: TBD

### Phase 17: Tools Tier & Environment Portability

**Original plan label**: P6 · size M
**Goal**: The portability thesis becomes a product feature — the user moves their whole environment to another PC, and installs tools without ever rebuilding the system
**Depends on**: Phase 15
**Execution host**: **Windows box** — the export/import proof needs machine A and a clean machine B, both real Windows hosts with the substrate.
**Requirements**: TOOLS-01, TOOLS-02
**Success Criteria** (what must be TRUE):

  1. The user exports their environment on machine A and imports it on a clean machine B, reconstructing a byte-identical closure and a working environment via the "Move to another PC" wizard
  2. Tool installs (`nix profile` against the pinned flake) never rebuild the system, and per-tool rollback works from the UI
  3. Floating-ref installs are impossible by construction — the profile tier only ever resolves against the Sourcerer-pinned flake

**Notes**: This is where spike D gets validated naturally (it was guessed with a recorded fallback in P0). The installed-profile list is exported into the manifest, so "roll back my environment" includes which tools were installed.
**Descope trigger**: none expected (D's mechanics are sound); wizard polish can trail.
**Plans**: TBD
**UI hint**: yes

### Phase 18: Permission Grants UI & Portal File Access

**Original plan label**: P7 · size M
**Goal**: The user can see and take back what each app is allowed to do, and file access is granted by picking a file rather than by an install-time flag
**Depends on**: Phase 17
**Execution host**: **NixOS dev host** for grant compilation and the no-parent-access test; **Windows box** for live revoke-without-reinstall UAT against the running substrate.
**Requirements**: GRANT-01, GRANT-02, GRANT-03
**Success Criteria** (what must be TRUE):

  1. The user views and revokes per-app grants (network, mounts, secrets) from a grants panel, and revocation takes effect without reinstalling the app
  2. A portal-style runtime folder grant is scoped to exactly the picked path — a no-parent-access test proves it — and grants are never install-time
  3. Agent tools run as their own DynamicUser units, so the unit file IS the grant and the agent cannot invoke a tool whose unit doesn't exist

**Notes**: GPU grants are removed from this panel until §6 lands in Platform v2. Secrets grants sit on top of Phase 10's plumbing; per-app activity visibility extends Phase 14's status panel.
**Descope trigger**: portal-mount UX too rough → v1 falls back to brokered copy-in/copy-out for user files, tracked as debt.
**Plans**: TBD
**UI hint**: yes

### Phase 19: Hardening Close & Milestone Audit

**Original plan label**: P8 · size M
**Goal**: Every unit that ships is proved hardened, and the decision to open the community store is made on audit evidence rather than optimism
**Depends on**: Phase 18 (shell track) + Phase 16 (store format must exist to decide on opening it)
**Execution host**: **Windows box** for the hardening UAT and `systemd-analyze security` sweep of the live app-unit fleet inside the real substrate; **NixOS dev host** for the CI gate and audit artifacts.
**Requirements**: HARD-01, HARD-02
**Success Criteria** (what must be TRUE):

  1. An exposure-score threshold runs as a CI gate on all compiled units, and no unit ships above it
  2. A `systemd-analyze security` audit of the app-unit fleet passes with a documented exemption set (agent harness, self-sandboxing services per Phase 8's recorded concessions)
  3. The store-opening decision is made on that evidence — open publicly, or hold — and the WSL2 shared-kernel caveat appears in user-facing security notes
  4. `/gsd-audit-milestone` runs for v2.0 and produces an artifact

**Notes**: This phase IS the descope checkpoint — no descope trigger of its own. The security score must honestly reflect nspawn/systemd + VM-boundary depth; gVisor's syscall-interception tier is deferred, revisited only on threat-model change. Carried debt worth settling before this gate: `/gsd-secure-phase` never ran for Phases 02-07 (only `01-SECURITY.md` exists), and `workflow.security_enforcement` is currently `false` in config.json.
**Plans**: TBD

## Progress

**Execution Order:**
Track 1 (shell tree, serial): 8 → 9 → 10 → 11 → 12 → 15 → 17 → 18 → 19
Track 2 (separate repos, parallel): 13 alongside 10/11 · 16 alongside 17/18
Join points: 14 needs 10 + 13 · 19 needs 18 + 16

| Phase | Milestone | Plans Complete | Status | Completed |
|-------|-----------|----------------|--------|-----------|
| 1. Shell Foundation | v1.0 | 3/3 | Complete | 2026-07-07 |
| 2. Workspace Core | v1.0 | 6/6 | Complete | 2026-07-08 |
| 3. Persistence & Layouts | v1.0 | 5/5 | Complete | 2026-07-10 |
| 4. Applet Framework | v1.0 | 5/5 | Complete | 2026-07-10 |
| 5. Notes Applet | v1.0 | 2/2 | Complete | 2026-07-13 |
| 6. Dashboard Assistant & Home | v1.0 | 8/8 | Complete | 2026-07-14 |
| 7. Assistant Harness Core | v1.0 | 6/6 | Complete | 2026-07-08 |
| 8. Spike K — Nix-Native Substrate Service | v2.0 | 0/TBD | Not started | - |
| 9. Flake Foundation & Assurance Chain | v2.0 | 6/7 | In Progress|  |
| 10. Substrate Provisioning & Warden Seam | v2.0 | 0/TBD | Not started | - |
| 11. Update Channel & Revert | v2.0 | 0/TBD | Not started | - |
| 12. Distribution Packaging | v2.0 | 0/TBD | Not started | - |
| 13. Engine Services in Substrate | v2.0 | 0/TBD | Not started | - |
| 14. Harness Relocation & First Engine-Backed Applet | v2.0 | 0/TBD | Not started | - |
| 15. App Layer v1 | v2.0 | 0/TBD | Not started | - |
| 16. Community Store Pipeline | v2.0 | 0/TBD | Not started | - |
| 17. Tools Tier & Environment Portability | v2.0 | 0/TBD | Not started | - |
| 18. Permission Grants UI & Portal File Access | v2.0 | 0/TBD | Not started | - |
| 19. Hardening Close & Milestone Audit | v2.0 | 0/TBD | Not started | - |

---
*v2.0 phase structure derived from `.planning/research/CONTAINER-PLATFORM-PLAN.md` (verifier-revised 2026-08-02) with the host-reality decision of 2026-08-03 applied — every phase states its execution host. Calendar note: unlike v1.0's in-editor phases, this milestone is wait-dominated (30–90 min closure builds, physical reboot UAT, clean-machine tests, Store certification queues measured in days). Plan counts hold; wall-clock roughly doubles.*
