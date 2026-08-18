# Requirements — Milestone v2.0 Container Platform

Source: `.planning/research/CONTAINER-PLATFORM-PLAN.md` (verifier-revised) + `CONTAINER-PLATFORM.md`. Decisions D-P1…D-P5 answered 2026-08-03 (see PROJECT.md Key Decisions). Two-host milestone: every requirement's UAT states its execution host (NixOS dev box or Windows/WSL box).

## v2.0 Requirements

### Spike remainder (P0)

- [ ] **SPIKE-01** *(rescoped 2026-08-04; **DEFERRED 2026-08-05** — planned, not executed; re-decide at Phase 15 whether to run it or absorb it into the compiler's design)*: Spike K validated on the Windows box — `services.collabora-online` (nixpkgs, cached) runs inside the substrate distro under the §7 hardening baseline, with every relaxed directive and its reason recorded, and the web UI reachable on 127.0.0.1 from the Windows host (re-import NixOS-WSL 2605.7.2 image first; local copy deleted 2026-08-03, sha256 recorded in spike 010 README)

### Foundation & assurance chain (FOUND)

- [x] **FOUND-01**: Developer can clone the repo on a second machine, enter the dev shell, and build the substrate image from the binary cache without compiling
- [x] **FOUND-02**: A red CI run (flake check, seed nixosTest, `windows-latest` Tauri job) blocks any publish
- [x] **FOUND-03**: Binary-cache hosting is decided, costed, and documented with a channel-maintenance runbook (cache is a production service from P2b on)

### Substrate provisioning (SUB)

- [ ] **SUB-01**: A non-technical user on a clean Windows 11 machine gets a working substrate with ≤3 interactions, zero terminal exposure, ≤10 min excluding downloads, at most one reboot
- [ ] **SUB-02**: User data lives on a separate mount from the system image; distro re-import preserves it
- [ ] **SUB-03**: No shell code reaches the substrate except via the substrate-connection transport seam, whose interface includes the auth surface (implementation stubbed)
- [ ] **SUB-04**: User can kill the substrate via a kill switch
- [ ] **SUB-05**: Secrets plumbing works end-to-end (sops-nix at rest + systemd LoadCredential) — mechanism only
- [ ] **SUB-06**: A real `security.csp` ships (the v1.0 `csp: null` acceptance expires by construction with multiwebview panes)

### Update channel (UPD)

- [ ] **UPD-01**: A deliberately broken publish is caught by CI and never reaches a user machine
- [ ] **UPD-02**: User can revert a bad-but-published update with one button (`--rollback`)
- [ ] **UPD-03**: Generations are bounded on disk (`nix.gc.automatic` + retention policy)

### Distribution (DIST)

- [ ] **DIST-01**: Microsoft Store (MSIX) install works on clean hardware, or spike J failure triggers the explicit v1 descope to site-only
- [ ] **DIST-02**: Signed site installer (EV / Azure Trusted Signing) installs on clean hardware with no SmartScreen wall

### Engine in substrate (ENG)

- [ ] **ENG-01**: Databasise engine runs inside the substrate; REST + MCP answer on loopback with a substrate-integration nixosTest gate green. **This is the one place the Nix-native rule's escape hatch applies** — if a Python/ML component fights uv2nix beyond the mandatory 3-day descope trigger, it may ship as a digest-pinned OCI unit, recorded with a written reason and tracked as debt to be replaced. Nowhere else in v2.0. (LightRAG is no longer a live dependency — it was disassembled into Databasise's own code; only vendored remnants remain, nothing to package separately. OCR is NOT part of the engine closure — it ships later as an installable applet through the app layer.)
- [ ] **ENG-02**: Assistant harness runs substrate-side with the `host.ai()` seam unchanged applet-side; Windows-side Python sidecar retired
- [ ] **ENG-03**: A wiki read-view applet renders live Databasise data through the seam (first engine-backed applet slice)
- [ ] **ENG-04**: User can see a substrate status panel (services, versions, current generation)

### App layer (APP)

- [ ] **APP-01** *(revised 2026-08-04 — Nix-native)*: Manifest schema v1 + manifest→NixOS-module compiler produce one hardened systemd unit per app, from **nixpkgs-packaged services only** (no OCI branch in v1 — see the Nix-native decision below); the hardening baseline and its exemption set come from spike K's recorded evidence; headless CLI install path exists
- [ ] **APP-02**: User can install Collabora from the catalog, edit a document in a pane (multiwebview per spike 011), uninstall, and a generation rollback restores the prior app set
- [ ] **APP-03**: A `network:none` app demonstrably cannot reach the internet (asserted via the unit's netns) and a unit's `MemoryMax` demonstrably binds
- [ ] **APP-04**: User sees a computed security score at install time that honestly reflects nspawn/systemd depth

### Community store (STORE)

- [ ] **STORE-01** *(revised 2026-08-04 — Nix-native)*: A package authored by a pinned-model + fixed-prompt fixture passes the submission pipeline end-to-end unattended (validate → build module → boot nixosTest → score → cosign-sign). The OCI-unit normalization stage is removed — submissions are Nix expressions against the pinned flake, never image references
- [ ] **STORE-02** *(revised 2026-08-04)*: Unsigned, score-floored, or privileged submissions auto-reject (D-P2, no appeal in v1); **any submission carrying an OCI image reference auto-rejects by construction** — the format has no field for one
- [ ] **STORE-03**: Store ops runbook + automation exists (CVE re-scans of approved images, digest re-pin cadence, submission triage)
- [ ] **STORE-04**: The store format ships but is not publicly opened until the P8 hardening gate passes

### Tools tier & portability (TOOLS)

- [ ] **TOOLS-01**: User can export their environment on machine A and import on a clean machine B, reconstructing a byte-identical closure and working environment ("Move to another PC" wizard)
- [ ] **TOOLS-02**: Tool installs (`nix profile` tier against the pinned flake) never rebuild the system; per-tool rollback works; floating-ref installs are impossible by construction

### Permission grants (GRANT)

- [ ] **GRANT-01**: User can view and revoke per-app grants (network, mounts, secrets) and revocation takes effect without reinstall
- [ ] **GRANT-02**: A portal-style runtime folder grant is scoped to exactly the picked path (no-parent-access test); grants are never install-time
- [ ] **GRANT-03**: Agent tools run as their own DynamicUser units — the unit file is the grant; the agent cannot invoke a tool whose unit doesn't exist

### Hardening close (HARD)

- [ ] **HARD-01**: An exposure-score threshold runs as a CI gate on all compiled units; `systemd-analyze security` audit of the **app-unit fleet** passes with the documented exemption set (seeded by spike K's recorded concessions)
- [ ] **HARD-02**: The store-opening decision is made on audit evidence and `/gsd-audit-milestone` runs for v2.0

## Future Requirements (deferred to Platform v2 / Cloud)

- OCR ingest as an installable applet — a Nix-packaged app delivered through the v2.0 app layer/catalog, surfaced as an applet in the GUI; not part of the engine closure
- macOS substrate (vfkit/microvm.nix, site DMG, MAS stretch) — P9
- Linux channel (tarball/AppImage + `nix run`) — P9
- WASM tier, per-applet webviews, streamed native Linux apps, filtering proxy — P10–P13
- GPU/compute routing (`gpu` manifest key, DeviceAllow, Metal/remote) — P14, consumes spike B
- Hosted cloud (auth impl, browser client, per-tenant substrate, billing) — Cloud milestone, committed (D-P5)

## Nix-native decision (2026-08-04)

**If a component is not in nixpkgs, it gets packaged.** The community catalog is Nix-native by
construction — an OCI image is never a valid submission format (STORE-01/02), and the app-layer
compiler has no OCI branch (APP-01).

**Sole exception:** first-party components that genuinely resist packaging may ship as
digest-pinned OCI units, time-boxed and tracked as debt to be replaced. In v2.0 that is
Phase 13's uv2nix descope trigger only (ENG-01).

**Rationale of record:** provenance (a derivation is built from source with an auditable chain;
an image is an opaque publisher blob), and store page-sharing at commercial scale (per-tenant
microVMs share library pages only via virtio-fs/DAX over one read-only host store, which
requires identical store paths — distinct image userlands never share).

**Explicitly NOT the rationale:** permissions (identical either way since the engine was
dropped — both compile to one systemd unit), disk footprint (amortizes per-image, not
per-tenant), or runtime speed (no difference). Do not re-derive this decision from those.

Full reasoning and the evidence that forced it:
`.planning/phases/08-spike-k-nix-native-substrate-service/08-CONTEXT.md`.

## Out of Scope

- Any container engine in the substrate (Podman/Docker dropped 2026-08-02; the engine-less OCI-unit path that replaced it was then narrowed to a first-party escape hatch 2026-08-04)
- OCI images as a community submission format or as an app-layer compile target (2026-08-04 — see the Nix-native decision above)
- gVisor runtime tier (deferred with Podman; revisit on threat-model change)
- Multi-container compose apps (deferred or curated Nix-native); compose = deferred single-container compat tier
- Nested Hyper-V hostile-agent tier; generic environments-platform productization
- React 19/Compiler; Power Browser rework; Databasise engine features (own roadmap)
- Secrets rotation (v1 = single cosign key, explicit)

## Traceability

| Requirement | Phase | Plan label | Execution host | Status |
|-------------|-------|------------|----------------|--------|
| SPIKE-01 | Phase 8 *(deferred)* | P0 | Windows box | Deferred — planned, re-decide at Phase 15 |
| FOUND-01 | Phase 9 | P6 | NixOS dev host | Wired, not fully live — substituter declared in flake.nix, but the cache is private for reads (401 on anonymous pull); see 09-06-SUMMARY.md Known Gaps |
| FOUND-02 | Phase 9 | P4 | NixOS dev host | Complete — CI green 2026-08-04; publish job added P6, currently red pending an unrelated infra deploy (09-06-SUMMARY.md) |
| FOUND-03 | Phase 9 | P6 | NixOS dev host | Documented — runbook + hosting decision complete; see 09-06-SUMMARY.md Known Gaps for the two live-operation caveats |
| SUB-01 | Phase 10 | P2 | Windows box | Pending |
| SUB-02 | Phase 10 | P2 | Windows box | Pending |
| SUB-03 | Phase 10 | P2 | Windows box | Pending |
| SUB-04 | Phase 10 | P2 | Windows box | Pending |
| SUB-05 | Phase 10 | P2 | Windows box | Pending |
| SUB-06 | Phase 10 | P2 | Windows box | Pending |
| UPD-01 | Phase 11 | P2b | NixOS dev host + Windows UAT | Pending |
| UPD-02 | Phase 11 | P2b | NixOS dev host + Windows UAT | Pending |
| UPD-03 | Phase 11 | P2b | NixOS dev host + Windows UAT | Pending |
| DIST-01 | Phase 12 | P2c | Windows box | Pending |
| DIST-02 | Phase 12 | P2c | Windows box | Pending |
| ENG-01 | Phase 13 | P3a | NixOS dev host | Pending |
| ENG-02 | Phase 14 | P3b | Windows box | Pending |
| ENG-03 | Phase 14 | P3b | Windows box | Pending |
| ENG-04 | Phase 14 | P3b | Windows box | Pending |
| APP-01 | Phase 15 | P4 | NixOS dev host + Windows pane UAT | Pending |
| APP-02 | Phase 15 | P4 | NixOS dev host + Windows pane UAT | Pending |
| APP-03 | Phase 15 | P4 | NixOS dev host + Windows pane UAT | Pending |
| APP-04 | Phase 15 | P4 | NixOS dev host + Windows pane UAT | Pending |
| STORE-01 | Phase 16 | P5 | NixOS dev host | Pending |
| STORE-02 | Phase 16 | P5 | NixOS dev host | Pending |
| STORE-03 | Phase 16 | P5 | NixOS dev host | Pending |
| STORE-04 | Phase 16 | P5 | NixOS dev host | Pending |
| TOOLS-01 | Phase 17 | P6 | Windows box (A + clean B) | Pending |
| TOOLS-02 | Phase 17 | P6 | Windows box (A + clean B) | Pending |
| GRANT-01 | Phase 18 | P7 | NixOS dev host + Windows UAT | Pending |
| GRANT-02 | Phase 18 | P7 | NixOS dev host + Windows UAT | Pending |
| GRANT-03 | Phase 18 | P7 | NixOS dev host + Windows UAT | Pending |
| HARD-01 | Phase 19 | P8 | Windows box + NixOS dev host | Pending |
| HARD-02 | Phase 19 | P8 | Windows box + NixOS dev host | Pending |

**Coverage:** 34/34 v2.0 requirements mapped to exactly one phase — no orphans, no duplicates.
