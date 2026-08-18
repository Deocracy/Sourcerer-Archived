# Container Platform — Full Milestone Plan (v2, verifier-revised)

> **RESUME HERE (recorded 2026-08-02 for a future session).** Planning is COMPLETE except two steps:
> 1. **User answers the five decisions D-P1…D-P5** (Prerequisites below). Only D-P1 licensing needs real thought; D-P2/D-P3/D-P4 have recommendations to bless; D-P5 "later" is a valid answer.
> 2. **Run `/gsd-new-milestone`** feeding THIS file + `CONTAINER-PLATFORM.md` — converts the plan into the v2.0 roadmap. v1.0 Desktop Shell MVP already SHIPPED 2026-07-14, so nothing blocks this. **Two things to know when you run it:**
>    - **Answer "Skip research" at step 8.** `workflow.research: true` makes "Research first" the default-presented option, but this file already *is* a verifier-revised milestone plan with spikes 010/011 validated — the research step would regenerate what exists and overwrite `STACK/FEATURES/ARCHITECTURE/PITFALLS/SUMMARY.md` by exact filename. v1.0 copies are archived at `.planning/milestones/v1.0-research/`, so this is a preference, not a data-loss risk.
>    - **It hard-deletes `.planning/phases/`.** `gsd-tools phases clear --confirm` is a bare `fs.rmSync(recursive, force)` with no archive (`bin/lib/milestone.cjs:281`). Everything is committed, so `git checkout HEAD -- .planning/phases/` restores it. Note `clear` skips directories matching `^999`, if you ever want a phase dir to survive one.
>
> Everything else is execution, not planning: **spike K** (engine-less Collabora unit — runs on the Windows box; see the HOST REALITY block below, the "warm distro" claim needs re-verifying and the local image was deleted); spike J before P2c; per-phase PLAN.md files are written one at a time by `/gsd-plan-phase` when each phase starts (standing rule — do NOT pre-plan).
> ~~Housekeeping owed~~ **DONE 2026-08-03:** the phase-05 recovery orphan, its prunable worktree and merged `gsd-reviewfix/05-10299` branch were verified (zero unique commits) and removed; the repo was normalized to LF, the design handoff tracked + deduplicated, ~17 GB of dead Windows build output reclaimed, and the JS toolchain reinstalled for Linux (203 tests green). **Still owed:** `/gsd-secure-phase 3` never ran — and it is worse than one phase, see the carry-forward section at the end of this file. `/gsd-audit-milestone` for v1.0 produced no artifact and appears never to have run (`/gsd-complete-milestone` did — RETROSPECTIVE.md and MILESTONES.md are populated).
> Spike status: 010 (substrate) + 011 (multiwebview) VALIDATED & committed; G/H retired by the Podman-drop delta; F/J/D/B guessed with recorded fallbacks; K pending.

> **⚠ HOST REALITY — RESOLVED 2026-08-03 (user decision). The WSL substrate premise STANDS.**
> This plan was written on a **Windows 11 host with WSL2**. Development has since moved to a **NixOS Linux host** (`wsl.exe` absent, no `/mnt`), but **the Windows machine remains available**. Decision: this becomes a **two-host milestone** — NixOS is the day-to-day dev/editor host, and the Windows box is where every WSL-dependent leg is executed and UAT'd. No architectural change; the substrate stays NixOS-WSL. Concrete corrections the roadmapper must carry:
> - **Spike K is no longer 1 day of pure work.** Its budget assumed "the `SourcererSpike` distro is still registered at D:\WSL, 3.4 GB, warm." Re-verify that distro exists on the Windows box; if not, add a re-import leg. **The local `nixos.wsl` image was deleted 2026-08-03** during migration cleanup — re-download NixOS-WSL 2605.7.2 and verify against sha256 `e7180ad555fdcb8e1e057e2ef056de467603a5e502ff8531053738371be3f6b9` (recorded in `spikes/010-nixos-wsl-substrate/README.md`). Spike 010's measured timings (import 41 s, boot 4 s, rebuild 109 s) still hold.
> - **Spike J** (MSIX + elevated `wsl --install` + reboot-resume) and **P2c** (Store submission, Windows code signing) **stay as real v1 phases** — they run on the Windows box, not here.
> - **P4/P8 pane and hardening UAT** need the Windows host too (spike 011's fractional-DPI retest is still open; its built `spike011-multiwebview.exe` was preserved for exactly this).
> - **P1's CI** `windows-latest` job stays, and becomes the regression net between hands-on Windows sessions.
> - **Every phase must state which host it executes on.** That is the one new planning requirement this decision creates — a phase whose UAT silently assumes the dev host will strand.
> - `CONTAINER-PLATFORM.md` carries 16 further WSL references; this file carries 8. All remain valid under this reading.

**Date:** 2026-08-02 · **Status:** DRAFT for `/gsd-new-milestone` ingestion — NOT active; current milestone (shell + applet framework) continues unchanged · **Architecture source of truth:** `CONTAINER-PLATFORM.md` (§ references) · **Verification:** goal-backward dependency check + solo-maintainer scope/risk check, both run 2026-08-02; all blockers/majors incorporated (v1 draft's ghost deliverables, phase splits, spike reordering) · **Planning rule:** milestone-level roadmap only; per-phase PLAN.md written one phase at a time when each starts.

## End state (what "done" means — every claim traces to a phase)

A non-technical scholar on a clean Windows 11 machine installs Sourcerer **from the Microsoft Store** (P2c), gets a working substrate after at most one reboot (P2), sees **engine-backed functionality live — Notes, the assistant, and a first Databasise-backed wiki read-view — served from the substrate** (P3a/P3b), installs LibreOffice (Collabora) from the catalog with a visible security score (P4), uses it in a pane (P4), rolls back a bad update with one button (P2b), and moves their whole environment to a second PC via manifest + data export (P6). Every substrate closure was CI-built and tested (P1); community packages run under a hardened runtime (P4/P8); the shell never assumes the substrate is local (P2).

## Prerequisites

1. **Current milestone ships** (Notes proof + `05-notes-applet` review-fix recovery). The shell is the warden; stable first.
2. **Spike battery passes** (P0 gate below).
3. **User decisions** (during P0; none block spikes): **D-P1** licensing (AGPL vs source-available vs proprietary — before any public repo/store) · **D-P2** store review hard-reject line (rec: privileged/docker-api-class auto-rejected, no appeal in v1) · **D-P3** Axis-3 blessing (system-flake + profile-tier split) · **D-P4** Nix implementation pin (rec: Determinate) · **D-P5** hosted timing (design ready; business commitment deferred).

## Standing-cost budget (verifier-surfaced; owned, not footnoted)

- **Binary cache is a production service from P2b on** (users pull updates from it): hosting decision + cost model + channel-maintenance runbook are P1 deliverables. Cachix free (5 GB) will NOT hold an ML-stack closure across versions — budget paid Cachix or Attic-on-VPS (~$5–10/mo) from the start. Channel-green upkeep (nixpkgs bumps break; someone owns the update branch building forever) is a named recurring duty.
- **Windows code signing for the site channel** (EV cert / Azure Trusted Signing) or SmartScreen kills the non-technical promise off-Store — P2c line item.
- **Store operations** (CVE re-scans of approved images, digest re-pin cadence, submission triage) — runbook + automation is a P5 deliverable, recurring thereafter.
- **Calendar calibration:** unlike v1.0's in-editor phases, this milestone is wait-dominated (30–90 min closure builds, physical reboot UAT, clean-machine tests, Store certification queues measured in days). Plan-counts hold; wall-clock roughly doubles.

---

## Phase P0 — Spike battery (the gate) — size M–L total

| Spike | Question it kills | Box | Failure consequence |
|---|---|---|---|
| **A — substrate proof** (extended) | Custom NixOS-WSL image (tarballBuilder) registers under private name, boots, renders a served UI in a Tauri webview pane (Collabora leg now spike K, engine-less); **then: flip channel → `nixos-rebuild switch` driven via wsl.exe → `--rollback`** (updater mechanics proven here, not in P2b) | 2.5 days | Fall back to **system-manager on stock Ubuntu-WSL** (numtide/system-manager — recorded in CONTAINER-PLATFORM.md §12); plan survives |
| **J — MSIX provisioning** (NEW, verifier) | Can an MSIX-packaged app drive elevated `wsl --install --no-distribution` + reboot-resume + `wsl --import` from its package context? (Ubuntu/Pengwin are distro-launcher MSIXes — a different pattern; unproven for a Tauri app) | 1.5 days | P2c descopes: v1 ships site-download (signed) only; Store becomes follow-up milestone work |
| **F — Python/ML packaging** | LightRAG + OCR under uv2nix? | 2 days | Those components ship as digest-pinned engine-less OCI units (per-component mix is architectural); plan survives |
| ~~**H — hardened runtime**~~ | **RETIRED (Podman drop, 2026-08-02)** — gVisor-under-podman no longer exists; its slot becomes P8's nspawn/systemd-hardening audit | — | — |
| **E — Tauri multiwebview** | `add_child` under `unstable`: bounds sync, per-origin isolation, stability? | 1 day | Child windows / WebView2 distinct-origin iframes for panes; Tier-2 applets delayed |
| ~~**G — egress isolation**~~ | **RETIRED (Podman drop, 2026-08-02)** — no per-app networks/netavark/nftables; `network:none` compiles to systemd `PrivateNetwork`+socket-activation / `IPAddressDeny` (standard behavior, no spike needed) | — | — |
| **K — engine-less OCI unit** (NEW, Podman-drop delta) | `dockerTools.pullImage` digest-pinned Collabora into the store → run as `RootDirectory=`/nspawn systemd unit inside the existing `SourcererSpike` distro → web UI reachable on 127.0.0.1? (Do real images run under plain systemd confinement without an engine's shims?) | 1 day | Fall back to `systemd-nspawn --oci-bundle` first; if both fail for real-world images, reopen the engine decision with evidence |
| **D — portability proof** | Manifest + data from machine A reconstructs identically on clean machine B? | 1 day (after A) | Core selling point broken — STOP and diagnose; not allowed to fail |
| **B — GPU on WSL** *(informational — consumed by no v1 phase; a failure reshapes nothing in v1)* | GPU unit with `DeviceAllow=/dev/dxg` (WSL GPU-PV, engine-less) in the custom distro? | 1 day | GPU routing (§6) stays v2 ("works on Linux, remote elsewhere") |

**Gate:** A + F + D must pass (fallbacks count as passes). J decides P2c's shape. **K decides P4's OCI-unit mechanics** (its fallback ladder is internal — RootDirectory → nspawn — before any engine reopening). E shapes scope. **Spike-A UX criterion operationalized:** ≤3 user interactions, zero terminal exposure, ≤10 min on reference hardware (excluding downloads), at most one reboot.

**P0 status (2026-08-02, user chose lean path — A-lite + E now, rest guessed with fallbacks as defaults):**
- **A-lite → VALIDATED** as spike `010-nixos-wsl-substrate` (import 41 s · boot 4 s · externally-driven `nixos-rebuild switch` 109 s · **rollback 6 s** · terminate/auto-restart 3 s · 3.4 GB VHDX; zero in-guest interaction). New P2 requirements discovered: pre-create the distro storage parent before `wsl --import`; **enforce a minimum WSL version** (2605 images "Catastrophic failure" on WSL 2.3.26, fine on 2.7.11); drive `wsl --update` through an explicit elevated flow (stalls invisibly behind UAC; concurrent update paths collide with error 1618). Collabora/pane legs superseded by spike K (engine-less OCI unit) per the Podman-drop delta.
- **E → VALIDATED** as spike `011-tauri-multiwebview` (external-origin panes render above a full-size shell webview, DOM-aligned bounds, live set_position tracking, shell heartbeat healthy, +8 WebView2 procs for 3 webviews). Caveat: 100%-scaling display only — add a fractional-DPI retest to P4's matrix; `unstable`-flag API risk stands (pane-host abstraction already planned).
- **F, J, D, B → GUESSED with recorded fallbacks** (F: run Python components as engine-less OCI units by default; J: site-signed installer default, Store attempt at P2c; D: validated naturally in P6; B: informational). **G, H → RETIRED by the Podman-drop delta** (their questions dissolved with the engine). **K → PENDING** (new; the one engine-less-OCI unknown worth a day before P4).

---

## MILESTONE "Platform v1" — two-track pipeline

**Track 1 (shell tree, serial — no-worktrees rule):** P1 → P2 → P2b → P2c → P4 → P6 → P7 → P8.
**Track 2 (separate repos, parallel):** P3a (engine packaging, Databasise repo) runs alongside P2/P2b; P5 (store repo + CI) runs alongside P6/P7. P3b (integration) joins the tracks after P2 + P3a.

### Phase P1 — Flake foundation & assurance chain — size M
**Delivers:** repo-root `flake.nix` (devShells + packages + nixosConfigurations skeleton, one lock); direnv dev loop in WSL (nix-ld/vscode-server baked into the image); CI: nix-installer-action + `nix flake check` + seed nixosTest + `windows-latest` conventional Tauri job; **cache hosting decision + cost model + channel-maintenance runbook** (verifier: cache = production service); cache wired push/pull; shared version pins both build worlds read.
**Success:** clean clone on machine 2 enters dev shell + builds substrate image from cache, no compiling; CI red blocks publish.
**Descope trigger:** none — assembly only; if this phase finds novelty, that's itself a finding.

### Phase P2 — Substrate provisioning + warden seam — size M
**Delivers:** custom `.wsl` image as flake output; first-launch flow (`wsl --install --no-distribution` → reboot-resume → `wsl --import`) + BIOS-virtualization help screen; **substrate-connection transport seam — interface includes the auth surface (impl stubbed)** (verifier: Cloud milestone claims it, so P2 must define it); kill switch; **secrets plumbing (sops-nix at rest + systemd LoadCredential) — mechanism only, P3 needs it** (verifier: was stranded in P7); user data on separate mount from system image.
**Success:** clean Windows machine → working substrate ≤1 reboot within spike-A's operationalized UX bounds; distro re-import preserves user data; no shell code reaches the substrate except via the seam.
**Descope trigger:** provisioning UX unachievable in bounds → guided semi-manual first-run accepted for v1, tracked as debt.

### Phase P2b — Update channel + revert — size M
**Delivers:** channel model (CI advances lock → build + nixosTest → cache publish → app-triggered `nixos-rebuild switch`); **"Revert last update" button** (`--rollback`); `nix.gc.automatic` + generation retention; channel-green runbook activated (P1's doc becomes practice).
**Success:** deliberately broken publish is caught by CI, never reaches a machine; bad-but-published update reverted by button; generations bounded on disk.
**Descope trigger:** app-driven switch proves unreliable despite spike A → adopt comin (§4-updates prior art) instead of in-app orchestration.
**Consumes:** spike A (extended updater leg).

### Phase P2c — Distribution packaging (Store + signed site) — size S–M
**Delivers (shape set by spike J):** MSIX packaging + Partner Center identity + Store submission (if J passed) OR explicit v1 descope to site-only; **signed site installer** (EV / Azure Trusted Signing — SmartScreen line item) either way.
**Success:** the end state's install claim is literal — a Store (or signed-site) install on clean hardware, no SmartScreen wall.
**Descope trigger:** Store certification stalls > 2 iterations → ship site-signed, queue Store as follow-up; end-state wording updates honestly.

### Phase P3a — Engine services in substrate — size L *(Track 2; parallel with P2/P2b; Databasise repo)*
**Delivers:** Databasise engine + LightRAG + OCR toolchain packaged per spike F (uv2nix, or engine-less OCI unit per component) as NixOS modules with the §7 hardening baseline; REST + MCP bound loopback; substrate-integration nixosTest asserting engine endpoints.
**Success:** engine answers REST/MCP inside the substrate with the nixosTest gate green; endpoint contract breakage fails CI.
**Descope trigger (mandatory, verifier):** any component fighting uv2nix > 3 days → containerize it, no appeal (F proves the happy path, not the long tail).

### Phase P3b — Harness relocation + first engine-backed applet — size L-
**Delivers:** assistant harness relocated into the substrate (host.ai() seam unchanged applet-side); Windows-side Python sidecar retired; **first Databasise-backed applet slice — wiki read-view consuming substrate REST** (verifier: end state demanded it; nothing delivered it); **substrate status panel** (services, versions, current generation — the cheapest visible value + debugging UI, pulls a P7 slice forward).
**Success:** all existing applet/assistant functionality works with substrate-side engine + harness; the wiki read-view renders live engine data through the seam; re-read `phase02-phase07-ownership-boundary` memory before planning.
**Descope trigger:** harness relocation destabilizes assistant UX → harness stays host-side one more phase (seam permits), relocation re-queued; wiki slice ships regardless.

### Phase P4 — App layer v1 (panes onto engine-less OCI units) — size L
**Delivers (pre-structured: compiler/enforcement waves complete and testable BEFORE any UI wave — verifier; Podman-drop delta applied):** manifest schema v1 (identity, UI port, permissions: network none/internet, fs.state, resources, autostart — **gpu EXCLUDED from v1 schema**, §6 is v2); **manifest→NixOS-module compiler ONLY** — one compile target for native and OCI apps alike: a hardened systemd unit, with OCI apps as `dockerTools.pullImage`-ingested rootfs derivations run via `RootDirectory=`/nspawn per spike K (`network:none` = `PrivateNetwork` + socket activation / `IPAddressDeny` — no per-app networks, no netavark/nftables deliverables); headless CLI install path; then UI wave: two curated apps — **Collabora CODE (WOPI host is a plan, not a parenthetical)** and Jupyter; pane host per spike E; install/remove UI with computed security score (score honestly reflects nspawn/systemd depth, not gVisor); installs recorded in generations. **Caveat owned:** runtime `docker pull` UX intentionally gone — app install/update = cached rebuild (a download, given the P1 cache), consistent with the declared model.
**Success:** install Collabora → edit in pane → uninstall → generation rollback restores prior app set; `network:none` app demonstrably cannot reach the internet (test asserts via unit's netns); a unit's `MemoryMax` demonstrably binds (the old podman cgroup trap is structurally gone — assert it anyway).
**Descope trigger:** an important real-world image resists `RootDirectory=` confinement → nspawn `--oci-bundle` per app; if a class of images resists both, that class waits for the deferred compat tier rather than reintroducing an engine ad hoc.

### Phase P5 — Community store pipeline — size M (conditions) *(Track 2; parallel with P6/P7; own repo)*
**Delivers (Podman-drop delta: the compose translator shrinks to an "OCI-unit normalizer"):** store-as-git-repo format (Umbrel model); submission CI: manifest validation → **OCI-unit normalization** (read image config; compile entrypoint/cmd/env/healthcheck into unit ExecStart/Environment/watchdog — semantics fixed at submission time, never interpreted at runtime; reuses P4's compiler wholesale — the M sizing condition) → build module → boot in nixosTest → score → **cosign-sign** (single key, no rotation in v1 — explicit) → **substrate verifies signatures before ingesting an image into the store**; hard-reject per D-P2; Trivy/Grype gate; authoring template + vibe-coding guide (image-ref + manifest, simpler than compose was); **ops runbook + automation: CVE re-scan of approved images, digest re-pin cadence, triage** (verifier: recurring cost owned). **Compose submissions: deferred compatibility tier** (single-container composes normalized into the same OCI-unit form later; multi-container apps deferred or curated Nix-native). **The store format ships but is NOT publicly opened until P8 passes** (verifier: ordering-tension fix).
**Success (operationalized, verifier):** a package authored by a **pinned model + fixed prompt harness (reproducible fixture)** passes end-to-end unattended; unsigned, score-floored, or tag-pinned submissions auto-reject.
**Descope trigger (mandatory):** pipeline can't pass the LLM fixture unattended → v1 descopes to first-party-curated catalog only; community store moves to v2.

### Phase P6 — Tools tier + environment manifest UX — size M
**Delivers:** `nix profile` tier against the Sourcerer-pinned flake (curated catalog UI, per-tool generations, profile exported into the manifest); environment export/import — "Move to another PC" wizard; per-tool rollback UI.
**Success:** spike D as product: export A → import on clean B → byte-identical closure, working environment; tool install never rebuilds the system; floating-ref installs impossible by construction.
**Descope trigger:** none expected (D proved mechanics); wizard polish can trail.

### Phase P7 — Permission grants UI + portal file access — size M
**Delivers:** grants panel (view/revoke per app: network, mounts, secrets — **GPU removed until §6 lands in v2**, verifier); portal-style runtime file/folder grants (shell picker → scoped mount compiled on the fly; never install-time); secrets grants UI over P2's plumbing; per-app activity visibility (extends P3b's status panel); agent tools as own DynamicUser units — unit file IS the grant.
**Success:** revoke takes effect without reinstall; folder grant scoped to exactly the picked path (no-parent-access test); agent cannot invoke a tool whose unit doesn't exist.
**Descope trigger:** portal-mount UX too rough → v1 falls back to brokered copy-in/copy-out for user files, tracked as debt.

### Phase P8 — Hardening close + milestone audit — size M
**Delivers (Podman-drop delta: gVisor leg replaced):** exposure-score threshold as CI gate on all compiled units (tiered baseline; sandboxer exemption set documented — agent harness, nspawn-launcher units); **nspawn/systemd-hardening audit of the OCI-unit fleet** (`systemd-analyze security` on every app unit; namespace/CapabilityBoundingSet/seccomp coverage review; confirm the security score honestly reflects nspawn/systemd + VM-boundary depth — gVisor's syscall-interception tier is deferred, revisit only on threat-model change) → **decision: open the community store publicly, or hold**; WSL2 shared-kernel caveat in user-facing security notes; `/gsd-audit-milestone`.
**Success:** no unit ships above threshold; audit passes; store-opening decision made on evidence.
**Descope trigger:** n/a — this phase IS the descope checkpoint.

---

## MILESTONE "Platform v2" — outline only (plan when v1 ships)

P9 macOS substrate (vfkit/microvm.nix + Metal helpers + site DMG; MAS edition stretch; **+ Linux channel: site tarball/AppImage + `nix run`** — verifier: was silently dropped) · P10 WASM tier (wasmtime/Extism, WIT world) · P11 per-applet webviews (Tier 2) · P12 streamed native Linux apps (Selkies, spike I) · P13 filtering proxy (domain-granular egress) · **P14 compute routing (§6: gpu manifest key, DeviceAllow compile [engine-less], Metal/remote providers — consumes spike B)** (verifier: §6 was a silent drop, spike B an orphan).

## MILESTONE "Cloud" — outline only, gated on D-P5 + D-P1

Auth module implementation (interface exists from P2) → browser build as unprivileged second client (localhost → LAN/Tailscale opt-in) → hosted substrate per tenant (microVM isolation, idle-suspend) → billing.

## Explicitly out of scope (all above)

GPU/compute routing in v1 (→P14) · **any container engine in the substrate (Podman/Docker dropped 2026-08-02 — engine-less OCI units only; compose = deferred single-container compat tier; multi-container compose apps deferred or curated Nix-native)** · **gVisor runtime tier (deferred with Podman; revisit on threat-model change)** · nested Hyper-V hostile-agent tier (reserve) · generic environments-platform productization (third ring, last) · React 19/Compiler (own track) · Power Browser rework (decision stands; shares only spike E's mechanism) · Databasise engine features (own roadmap) · secrets rotation (v1 = single key, explicit).

## Sequencing rationale (goal-backward, verifier-revised)

Trust chain: P8←P5←P4←P2b←P1 — no community code before enforcement (P4), no enforcement before an updatable/revertible substrate (P2/P2b), nothing published a CI didn't prove (P1). P3a/P3b put the first "app" (trusted: our engine) through every seam before untrusted code arrives. The two-track split exists because the engine and store live in separate repos — the no-worktrees rule binds only the shell tree. Spikes front-load the ways the plan could be wrong (image mechanics, MSIX provisioning, Python packaging, updater mechanics, engine-less OCI units, pane mechanism, portability) before any phase commits; the updater late-validation → spike-A leg was a verifier-forced move, and the Podman-drop delta (2026-08-02) retired spikes G/H while adding K — one enforcement dialect means fewer distinct ways to be wrong.

## Carried forward from v1.0 (do not lose in the milestone rollover)

- **`security.csp = null` acceptance EXPIRES with this milestone.** Phase 1 accepted it (`01-SECURITY.md`, T-01-14) on an explicit written condition: *"Remains acceptable ONLY while the webview loads no remote/untrusted content. The first phase that introduces remote URLs, external iframes, or user-supplied HTML MUST set a real `security.csp` before shipping."* v2.0 breaks that condition **by construction** — multiwebview panes rendering container-served UIs, and `CONTAINER-PLATFORM.md` already assumes `connect-src 'self'`. `src-tauri/tauri.conf.json:25` still reads `"csp": null`. Setting a real CSP is a **P2 deliverable**, not a nice-to-have.
- **The security baseline is thinner than STATE.md implies.** `01-SECURITY.md` is the **only** SECURITY.md in the repo. Phases 02–07 have none — including Phase 07, the Pi sidecar, which is the one component with a real trust boundary. STATE.md's Deferred Items row says only "phase 3 outstanding", understating it by five phases. Note `workflow.security_enforcement` is now `false` in `.planning/config.json`, so `/gsd-secure-phase` no-ops until it is flipped back. v2.0 is explicitly about running **untrusted community OCI images**, so this baseline is worth settling before P4/P5 threat-modelling rather than after.
