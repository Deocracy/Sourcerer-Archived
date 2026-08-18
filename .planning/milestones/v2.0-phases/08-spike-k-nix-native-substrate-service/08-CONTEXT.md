# Phase 8: Spike K — Nix-Native Substrate Service - Context

**Gathered:** 2026-08-04
**Status:** Ready for planning

<domain>
## Phase Boundary

Prove that a real nixpkgs service — `services.collabora-online` — runs inside the NixOS-WSL
substrate distro under the §7 hardening baseline, and that its web UI is reachable on
127.0.0.1 from the Windows host.

**This phase was rescoped on 2026-08-04.** It was originally "Spike K — Engine-less OCI Unit":
ingest a digest-pinned Collabora CODE image via `dockerTools.pullImage` and run the unpacked
rootfs as a `RootDirectory=`/nspawn systemd unit. That framing is obsolete. See
`<decisions>` D-01 for the policy change and its evidence.

**In scope:** re-import the NixOS-WSL substrate; run stock `services.collabora-online` from
the binary cache; apply the full hardening baseline and relax it knob-by-knob; record the
resulting exemption set; prove reachability from Windows.

**Out of scope:** any OCI image path; a second service (Jupyter); the multiwebview pane leg;
the flake (Phase 9 owns it); the manifest schema or compiler (Phase 15 owns them).

</domain>

<decisions>
## Implementation Decisions

### Nix-native policy (milestone-level — reaches past this phase)

- **D-01: Nix-native is the rule.** If a component is not in nixpkgs, it gets packaged.
  The community catalog is Nix-native **by construction** — an OCI image is never a valid
  submission format.
- **D-02: OCI survives only as a time-boxed internal escape hatch.** Permitted for
  *first-party* components that genuinely resist packaging — specifically Phase 13's
  mandatory uv2nix descope trigger for Databasise's Python/ML stack. Using it requires
  writing down why, and it is tracked as debt to be replaced. One image we control and plan
  to remove ≠ N community images forever.
- **D-03: Rationale of record — provenance first, then store page-sharing at commercial
  scale.** A Nix derivation is built from source with an auditable chain; an image is an
  opaque publisher blob that Trivy/Grype scanning does not make equivalent. At the Cloud
  milestone's stated model (per-tenant microVMs, idle-suspend, ~10k tenants / ~200 active),
  guests share library pages only via virtio-fs + DAX over one read-only host Nix store, and
  that **only works when guests reference identical store paths**. Three OCI images carrying
  three near-identical glibcs at three distinct store paths will never share a page. KSM is
  not an acceptable substitute in a multi-tenant deployment (CPU cost + side-channel exposure).
- **D-04: Permissions are explicitly NOT part of the rationale.** Since the Podman drop
  (2026-08-02) native services and OCI apps compile to the same target — one systemd unit,
  with `RootDirectory=` merely pointing at a store path. `DynamicUser`, `ProtectSystem`,
  `PrivateNetwork`, `CapabilityBoundingSet`, `MemoryMax` apply identically to both. Anyone
  re-deriving this decision should not reuse a permissions argument; it does not hold.
- **D-05: Disk footprint is NOT part of the rationale either.** At scale an image's cost is
  paid per distinct image, not per tenant, and amortizes to near-nothing across a large
  tenant base. It was a desktop-scale argument and does not survive the commercial framing.

### Evidence that forced the rescope

- **D-06:** `collabora-online` 25.04.9-4 (MPL-2.0) **is in nixpkgs**, with a full
  `services.collabora-online` module exposing `enable` / `package` / `settings`, where
  `settings` maps declaratively onto `coolwsd.xml`. Verified against nixpkgs unstable via the
  `nixos` MCP on 2026-08-04. The plan was written on the assumption Collabora ships only as
  an image; that assumption was false.
- **D-07:** It is **already in the binary cache** for `x86_64-linux` —
  `/nix/store/82181wy8scpzh0fis39gjjjnzk5462c9-collabora-online-25.04.9-4`, 13.8 MB download
  / 42.0 MB unpacked, zstd. Against ~1.5–2 GB for the CODE image. (That figure is the package,
  not its full closure; the closure's deps are shared with the rest of the system, which is
  the point.)
- **D-08 [informational]:** `services.jupyter` is likewise in nixpkgs. Both Phase 15 catalog apps
  need no image at all. *Not tracked by this phase's plans by design* — D-11 fences Jupyter out
  of Phase 8 entirely and `<deferred>` routes it to Phase 15. It is recorded here as rescope
  evidence, not as work.

### Spike design

- **D-09: The kill-question moved.** It is no longer "do real-world images run without an
  engine's shims?" It is: **does a service that sandboxes itself survive the §7 hardening
  baseline under WSL2's kernel?** `coolwsd` forks jailed LibreOffice kernels using its own
  chroot + namespaces, which collides directly with `DynamicUser`, `ProtectSystem=strict`,
  `NoNewPrivileges`, `CapabilityBoundingSet` and `SystemCallFilter`. WSL2's kernel is not a
  normal kernel. That collision is the thing worth a day.
- **D-10: Full baseline first, then relax knob-by-knob.** Start from the complete §7 baseline,
  let it fail, then relax one directive at a time and record **every concession with its
  reason**. Explicitly rejected: "stock module then measure" (tells you where you stand, not
  where the limits are) and "minimum to boot" (produces a green checkmark, not evidence).
  The output *is* the deliverable — it becomes Phase 19's documented exemption set and the
  evidence Phase 15's compiler is written against.
- **D-11: Collabora only.** It is the hard case — self-sandboxing, capability-hungry, and the
  app Phase 15's UAT names. If the baseline survives coolwsd it survives most things. Keeps
  the phase near its original one-day budget plus the re-import leg.
- **D-12: Build in-substrate.** The distro is already NixOS and the package is cached, so
  nothing compiles — it substitutes. One host to reason about, zero cross-machine transport
  plumbing. Rejected: building on the NixOS dev host and `nix copy`-ing the closure over the
  LAN, which turns a transport failure into a spike failure that has nothing to do with the
  question. Phase 9 owns the cache/transport story.
- **D-13: Reachability bar is curl + browser** on 127.0.0.1 from the Windows host. No
  multiwebview pane leg this phase.

### Claude's Discretion

- Exact §7 directive list and the order in which directives are relaxed — derive from
  `CONTAINER-PLATFORM.md` §7 and record the order actually used.
- Whether the recorded findings get packaged as a `spike-findings-*` skill via
  `/gsd-spike --wrap-up`, or stay as the spike README. Default: wrap up, since Phase 15 and
  Phase 19 are both downstream consumers and are many phases away.
- Whether to unregister the `SourcererSpike` distro at the end or leave it warm for Phase 10.
  Default: leave it registered (spike 010 did the same); note the state in the README.
- Timebox/abort handling if the hardening ladder proves bottomless: record the concessions
  reached and stop rather than pushing to a clean pass — a partial exemption set is still the
  evidence Phase 19 needs.

</decisions>

<canonical_refs>
## Canonical References

**Downstream agents MUST read these before planning or implementing.**

### Milestone architecture
- `.planning/research/CONTAINER-PLATFORM.md` — architecture source of truth. **§7** is the
  hardening baseline this spike exercises (tiered baseline; "sandboxer" services get a
  documented exemption set — coolwsd is exactly such a service). **§5 is now partly
  superseded** by D-01/D-02: its "community app layer = engine-less OCI" premise no longer
  holds as the default path. **§11** carries the WSL2 shared-kernel caveat.
- `.planning/research/CONTAINER-PLATFORM-PLAN.md` — the milestone plan. Its P0 spike-K row
  and P4 OCI-unit mechanics are superseded by this phase's decisions; its P3a mandatory
  descope trigger (uv2nix > 3 days → OCI unit) is the one OCI path that **survives** under D-02.

### Prior spike evidence (landmines this phase will hit)
- `.planning/spikes/010-nixos-wsl-substrate/README.md` — the substrate mechanics this spike
  builds on. Carries the NixOS-WSL 2605.7.2 sha256
  (`e7180ad555fdcb8e1e057e2ef056de467603a5e502ff8531053738371be3f6b9`), the deleted-image note,
  and three product landmines: pre-create the distro storage parent before `wsl --import`;
  enforce a **minimum WSL version** (2605 images "Catastrophic failure" on WSL 2.3.26, fine on
  2.7.11); drive `wsl --update` through an explicit elevated flow.
- `.planning/spikes/CONVENTIONS.md` — **binding for this phase's artifact form.** One dir per
  spike with `run.sh` → tee'd `run.log` + README with an Investigation Trail. System-level
  rules: export `WSL_UTF8=1` (raw wsl.exe output is UTF-16 in Git Bash); wrap absolute Linux
  paths in `sh -c '...'` (MSYS rewrites them); never trust `cmd | tail`'s exit code, check
  `PIPESTATUS[0]`.
- `.planning/spikes/MANIFEST.md` — spike index; this phase adds the next numbered entry
  (012), since 011 is the highest existing.

### Project state
- `.planning/REQUIREMENTS.md` — SPIKE-01 is this phase's single requirement (rewritten
  alongside this context to drop the OCI framing).
- `.planning/PROJECT.md` — Key Decisions table; D-01/D-02 are recorded there as a
  milestone-level decision.
- `.planning/ROADMAP.md` — Phase 8 entry (rewritten 2026-08-04).

### Upstream package facts (verified 2026-08-04 via the `nixos` MCP, channel `unstable`)
- `services.collabora-online.{enable,package,settings}` — NixOS module; `settings` maps onto
  `coolwsd.xml` (attrs → XML tags, `@`-prefixed → XML attributes, lists → repeated tags,
  null → element removal).
- `collabora-online` 25.04.9-4, MPL-2.0 — cached, 13.8 MB / 42.0 MB unpacked.

</canonical_refs>

<code_context>
## Existing Code Insights

### Reusable Assets
- **`.planning/spikes/010-nixos-wsl-substrate/run.sh`** — the timed, tee-logged driver-script
  pattern for host-mutating WSL spikes. This phase's driver should be a direct descendant:
  same `WSL_UTF8=1` / `sh -c` / `PIPESTATUS` discipline, same per-leg timing table in the
  README results section.
- **The `SourcererSpike` distro itself** — if still registered on the Windows box, the
  re-import leg collapses to a verification step. Assume it is gone until proven otherwise;
  the local `nixos.wsl` blob was deleted 2026-08-03 and `dist/` is gitignored, so it was never
  in git.

### Established Patterns
- **Spikes live entirely under `.planning/spikes/NNN-name/`**, never in the app tree. No
  `src/` or `src-tauri/` file should change in this phase.
- **Forensic-first:** every attempt logged including failures; failed-attempt logs stay in the
  README's Investigation Trail. Spike 010's three-attempt trail is the model — its "Attempt 2
  failed, hypothesis: WSL too old" → "Attempt 3 confirmed" structure is what makes the
  landmines reusable.

### Integration Points
- **None in the shell tree.** This phase touches no Tauri, React, or Rust code. Its only
  outputs are the spike directory, the recorded exemption set, and the doc updates listed in
  `<canonical_refs>`.
- **Downstream consumers:** Phase 15's manifest→unit compiler (which hardening directives a
  real service tolerates) and Phase 19's `systemd-analyze security` audit (the documented
  exemption set). Both are far enough away that the findings must be written for a reader with
  no memory of this session.

</code_context>

<specifics>
## Specific Ideas

- The user framed the efficiency argument explicitly in a **commercial multi-tenant setting —
  ~10,000 Sourcerer instances on a server with ~200 concurrently active.** That framing is
  what promoted Nix-native from preference to architecture decision, and it is the reason
  D-03 is written in terms of virtio-fs/DAX store sharing rather than desktop disk usage.
  Any future agent re-opening this decision should evaluate it at that scale, not at
  single-desktop scale.
- The user's standing instruction on this topic: *"if it's not a nixpkgs we don't use it, or
  we make it a nixpkgs."* D-02's escape hatch is a deliberate, bounded exception to that rule,
  not a softening of it.
- Runtime CPU/latency was explicitly established as **identical** between native and OCI paths
  since the engine was dropped. Nobody should claim a speed benefit for this decision.

</specifics>

<deferred>
## Deferred Ideas

- **`services.jupyter` as a second, well-behaved contrasting service** — would show the
  hardening pattern generalizes rather than being tuned to one app. → Phase 15, where the
  compiler needs exactly that breadth.
- **The multiwebview pane leg** — render Collabora in a Tauri pane using spike 011's preserved
  `spike011-multiwebview.exe`, closing its still-open fractional-DPI retest. → Phase 15, which
  already owns the pane UAT and names that retest.
- **The engine-less OCI unit spike** (the original Phase 8) — genuinely useful *if* Phase 13
  ever invokes its uv2nix escape hatch under D-02. Not before, and not speculatively.
- **Packaging work for anything not currently in nixpkgs** — D-01 makes this a real, recurring
  workstream, but nothing in v1's catalog needs it (Collabora and Jupyter are both packaged).
  Revisit when a specific component fails the rule.

</deferred>

---

*Phase: 8-Spike K — Nix-Native Substrate Service*
*Context gathered: 2026-08-04*
