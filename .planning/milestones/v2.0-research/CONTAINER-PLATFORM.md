# Container Platform — Nix Substrate Architecture Research & Direction

**Date:** 2026-08-02 · **Method:** brainstorm session (`/gsd-explore`) + 11 web-research agents across 4 verified sweeps (feasibility, Nix stack, Mac App Store adversarial pair, permissions/GUI) + 1 streamed-apps verification agent; load-bearing claims spot-checked by dedicated critics · **Status:** brainstorm-level DIRECTION, not scheduled — current milestone (shell + applet framework) unaffected · **Consumed by:** a future "Platform" milestone; spikes listed in §12 gate commitment. · **Delta 2026-08-02 (side-chat decision): Podman DROPPED — OCI apps run engine-less as hardened systemd units; §5/§6/§7/§11/§12/§13 updated accordingly.**

**User's decision criteria (fixed):** 1) reproducibility — same inputs, same working system, any machine, months later; 2) rapid development with assurance-of-working; 3) non-technical scholars, one-click install; 4) community members author and share packages with LLM help ("vibe coding"); 5) rogue-AI containment; 6) portability between machines.

---

## 1. The architecture at a glance

**Five concepts, three deployment modes, four runtime tiers, four applet tiers.**

```
DEPLOYMENT MODES (substrate location independence — §2)
  local substrate | hosted substrate + browser client | hosted substrate + Tauri client

SHELL (the WARDEN — native, reviewed, signed; never served by the substrate)
  Tauri 2 + React 18 + dockview (CONFIRMED KEEP, §9) — panes, kill switch, grants UI
      │  one transport seam: "substrate connection" (wsl.exe/VM socket ⇄ remote TLS+auth)
SUBSTRATE (one Nix flake = the environment manifest; flake.lock = portability artifact)
  Windows: private custom-named NixOS-WSL distro ("Sourcerer") — wsl --import
  macOS:   NixOS in a Virtualization.framework VM (vfkit / microvm.nix vfkit backend)
           + bundled host-native Metal helper processes for ML
  Linux:   native Nix / NixOS
RUNTIME TIERS inside the substrate (§5)
  1. platform services (engine, sidecars, harness) = native NixOS modules
  2. community large apps = digest-pinned OCI images pulled into the Nix store at build
     time, run ENGINE-LESS as hardened systemd units (no Podman/Docker — §5)
  3. user-toggled tools = nix profile against a Sourcerer-pinned flake (nixpkgs 122k+ catalog)
  4. streamed native Linux GUI apps = headless app + Selkies/KasmVNC → webview pane (§10)
```

**The portability thesis (realized):** the user's whole environment = `flake.nix` + `flake.lock` + data volumes. Copy to any machine → `nixos-rebuild switch --flake` reconstructs bit-identically, months later. Installed community apps are part of system generations, so "roll back my environment" includes which apps were installed. Sharing a configured research environment with a colleague = sending the manifest.

**Two hard boundaries (2026 facts):**
- **No native Windows Nix exists** (nova-nix hit stage-0 June 2026 — years away). Nix owns 100% of the substrate, 0% of the Windows Tauri shell. The `.msi`/MSIX builds conventionally (tauri-cli, `windows-latest` CI) with lockfile-grade pinning; both build worlds read shared version pins.
- **A Linux VM on macOS gets no GPU compute passthrough** → ML on Mac runs as host-native Metal helpers (fine in the App Sandbox), or remote.

## 2. Deployment modes (USER DECISION 2026-08-02: first-class design option)

Many users will only adopt Sourcerer if setup is zero-fuss, or their PC lacks specs. The substrate is the same NixOS closure everywhere; **where it runs is a user choice**:

1. **Local** — substrate in WSL/VM on the user's machine (default).
2. **Hosted + browser client** — log into a website; zero install. Same React UI served as a browser build; all panes are already just URLs.
3. **Hosted + Tauri client** — native app attaches to a substrate on our infrastructure: full desktop UX, no local specs, same environment from every PC (multi-PC roaming).

Migration between modes is a first-class operation (manifest + data sync — "download your environment and go local" / "push local to hosted").

**Design constraint effective immediately:** the shell must NEVER assume the substrate is localhost. One transport seam ("substrate connection": local exec/socket vs remote TLS endpoint + auth); everything above it identical. Auth is a first-class module, not a localhost afterthought.

**Warden abstraction:** locally the warden is the Tauri shell; hosted, it's a per-tenant orchestrator doing the same jobs (lifecycle, grants, limits). Nothing inside the substrate may assume "my warden is a desktop app."

Browser client is the *unprivileged* client: no VM lifecycle, no kill switch, no Power Browser, no Tier-2 applet isolation. Ship bound to 127.0.0.1; remote exposure opt-in.

## 3. Distribution channels (Axis selections, user 2026-08-02)

| OS | Substrate | Channel | Verified mechanics |
|---|---|---|---|
| Windows | NixOS-WSL private distro | **Microsoft Store (MSIX)** | Proven WSL-distro Store pattern (Ubuntu/Pengwin). First launch: `wsl --install --no-distribution` (UAC + possible reboot) → `wsl --import` bundled image (`--name`/`--location` supported; WSL 2.4.4+ `.wsl` format). One unfixable failure mode: virtualization off in BIOS → guided help screen. Windows Home works. |
| macOS | NixOS in VZ VM + Metal helpers | Site download (Developer ID, primary) + **Mac App Store edition (verified doable)** | MAS precedents live today: Parallels Desktop App Store Edition, UTM. `com.apple.security.virtualization` is public/self-service and sandbox-compatible; multi-GB VM disks in container precedented (UTM); NAT/vsock guest↔host needs nothing restricted. Tauri has an official MAS path. |
| Linux | native Nix/NixOS | Site download / `nix run` | Deliberately low-thought. |

**Mac App Store conditions (adversarial-verified):** (1) the 10px `transparent:true` window card uses Tauri's private-API feature → hard MAS rejection; MAS edition needs square/native corners (site edition keeps the card) — Parallels-style two-edition split, one flag. (2) Present the catalog as environment/workspace config, not "an app store of Linux apps" (guideline 2.5.2/2.4.5 is enforcement-defined; VM framing is protected, catalog framing is reviewer-discretion). (3) VM dies with the app (2.4.5(iii)); no bridged networking (`com.apple.vm.networking` is the only approval-gated entitlement — not needed); Rosetta-x86-in-guest degraded under sandbox (irrelevant: our images are native ARM). Sidecar signing: every bundled executable individually signed with inherit entitlements; never `codesign --deep`. NOTE (original M1 native-Nix pick REVERSED): native Nix on the Mac host = MAS-impossible + second module dialect (nix-darwin ≠ NixOS; x86_64-darwin dies with nixpkgs 26.05) → VM route adopted; also evaluate **Apple Containerization framework** (1.0, June 2026: VM-per-container, sub-second boot) as the macOS substrate alternative.

## 4. The Nix stack (verified Aug 2026)

**Anchors:** NixOS 26.05 "Yarara" (2026-05-30); NixOS-WSL 2511.7.1 (25.11 line; plan for 2605); Determinate Nix 3.21.8 (flakes formally stable; lazy trees = 3x eval); nixpkgs 122k+ pkgs. **Pin which Nix ships in the image** — three-way choice (upstream / Determinate / Lix); users never install Nix, so "experimental flakes" is a non-issue.

- **Dev:** raw flake devShells + direnv/nix-direnv (base layer; warm re-entry ~instant). Optional ergonomics: devenv 2.x (bundles patched Nix — pin it), services-flake (vendor-neutral services without devenv), Devbox/Flox (zero-Nix JSON/TOML — study Flox as prior art for our own manifest UX; its WSL support is "experimental"). Repos live in the WSL filesystem (never /mnt/c); VS Code Remote-WSL needs nix-ld/nixos-vscode-server baked into the distro image.
- **Build:** crane (Rust; deps-once caching), buildNpmPackage + importNpmLock (Node/Vite; NOT pnpm — 2025-26 FOD hash-instability trail; NOT dream2nix — unstable by own admission), dockerTools.pullImage/skopeo for ingesting digest-pinned third-party OCI images into the store (the §5 engine-less path); dockerTools.streamLayeredImage/nix2container retained only for first-party images the hosted milestone may need (arion dropped with Podman — §13). **Windows Tauri bundle: cannot be Nix-built (WiX Windows-only; cargo-xwin impure) — scoped exception, conventional pinning.** Python/ML packaging = **the heaviest engineering risk**: uv2nix (poetry2nix is legacy); sanctioned fallback = run exactly those components as digest-pinned containers.
- **Assurance chain (criterion 2 as a mechanism):** one flake.lock shared by dev shells, CI, substrate image → GitHub Actions + nix-installer-action (KVM on free public runners) → **nixosTest boots the whole substrate (native services + engine-less OCI units) and asserts REST/MCP endpoints per PR** → binary cache (Cachix free 5 GB → self-hosted Attic; Harmonia for LAN) — the cache is the assurance TRANSPORT: users only download closures CI already tested. Honest gap: nixosTest boots QEMU-NixOS, not WSL — keep one thin real-WSL smoke test. Vendor lesson: Garnix acquired+shut down July 2026, magic-nix-cache free tier died Feb 2025 → keep CI trivially portable (plain GH Actions + swappable cache).
- **Updates:** never stock `system.autoUpgrade` (flake path deprecated, server-shaped timer). Channel model: CI advances flake.lock on a branch → build + nixosTest → publish cache → app triggers `nixos-rebuild switch --flake <channel>` at an app-chosen moment; `--rollback` wired to a visible "Revert last update" button. Prior art: **comin** (nlewo) implements this pull-based pattern. Ship `nix.gc.automatic` + generation retention from day one. Secrets never in /nix/store (sops-nix/agenix + systemd credentials). User data on a separate mount from the system image.

## 5. Community app layer — ENGINE-LESS OCI (Podman dropped, delta 2026-08-02)

> **⚠ SUPERSEDED IN PART — 2026-08-04 (Nix-native decision).** This section's premise, that the
> community app layer is built on digest-pinned OCI images, **no longer holds as the default
> path**. The rule is now: if a component is not in nixpkgs, it gets packaged. The community
> catalog is Nix-native by construction and an OCI image is never a valid submission format;
> the app-layer compiler has no OCI branch. What forced it: `collabora-online` — the app this
> whole section was written around — is already in nixpkgs (25.04.9-4, MPL-2.0) with a full
> `services.collabora-online` module, cached at 13.8 MB download / 42 MB unpacked against
> ~1.5–2 GB for the CODE image. `services.jupyter` likewise.
>
> **Sole surviving OCI path:** a time-boxed escape hatch for *first-party* components that
> genuinely resist packaging — in v2.0 that is Phase 13's uv2nix descope trigger only, recorded
> with a written reason and tracked as debt.
>
> **Rationale of record:** provenance, and store page-sharing at commercial scale (per-tenant
> microVMs share library pages only via virtio-fs/DAX over one read-only host store, which
> requires identical store paths). Explicitly **not** permissions — since the Podman drop,
> native and OCI compile to the same systemd unit and the §7 vocabulary applies identically to
> both. Do not re-derive this decision from a permissions argument.
>
> §7's hardening baseline below is unaffected and remains the reference.
> Full reasoning: `.planning/phases/08-spike-k-nix-native-substrate-service/08-CONTEXT.md`.

**No container engine ships in the substrate.** OCI apps run engine-less: `dockerTools.pullImage`/skopeo pulls the **digest-pinned** image into the Nix store **at build time** → an unpacked-rootfs derivation → executed by systemd itself, either as a `RootDirectory=` unit or via `systemd-nspawn --oci-bundle`, under the **same systemd hardening vocabulary as native services (§7)**. One isolation mechanism for native and OCI apps alike — no daemon, no image-store state outside Nix, no second enforcement dialect, and the app's rootfs is part of the system closure (generations now include app *content*, not just app *references*).

**Package format:** a community app = small manifest (icon, description, UI port, permissions §7) + a digest-pinned image reference (or a Nix expression for power apps). The store pipeline **normalizes image semantics at submission time** — entrypoint/cmd/env/healthcheck are read from the image config and compiled into the unit (ExecStart, Environment, systemd watchdog/health probe) — so runtime never interprets OCI metadata. **Caveat recorded:** runtime `docker pull` UX is intentionally gone — installing/updating an app is a (cached, CI-prebuilt) rebuild, consistent with the declared model; the binary cache makes this a download, not a build.

**Compose status (re-scoped):** `docker-compose.yml` as a submission format is now a **deferred compatibility tier** — single-container composes may later be accepted and normalized into the same OCI-unit form; **multi-container compose apps are deferred or arrive as curated Nix-native packages** (their orchestration semantics don't map cleanly onto single units). Umbrel's "any git repo is a community store" model still adopted. nixpkgs underneath = zero-authoring tool catalog via `nix profile` against OUR pinned flake (never floating refs; profile list exported into the manifest). Store CI: normalize → build module → boot in nixosTest → **cosign-sign the approved artifact; the substrate verifies signatures before an image is ingested into the store** (review must be enforced, not advisory) + Trivy/Grype scan. Tag-pinned images forbidden — digests only.

## 6. GPU / compute providers

Packages declare compute needs; platform routes: **local CUDA** via `DeviceAllow=` on the unit (engine-less — no nvidia-container-toolkit/CDI; WSL2's GPU-PV path is `/dev/dxg` + `/usr/lib/wsl/lib`, UNVERIFIED → spike B); **host-native Metal** helper on macOS; **remote GPU service** (vLLM/Ollama-style endpoints; GPU-over-IP à la rCUDA is dead). Prebuilt CUDA images ingested engine-less still dodge Nix's CUDA-packaging pain (the image ships its own userland; the unit only grants the device). Podman's cgroup re-parenting trap is gone with Podman — `MemoryMax`/`CPUQuota` on the unit now bind directly for OCI apps too.

## 7. Permission system (manifest-declared, platform-compiled)

**Core principle:** security policy is DECLARED in package manifests and COMPILED by a Nix function into kernel-backed enforcement — never trusted to app code. NixOS-only superpower: `systemd.services.<name>.confinement.enable` chroots a service to exactly its Nix closure + declared mounts (possible only because Nix knows the closure). `systemd-analyze security` exposure score = CI gate on compiled units.

**Since the Podman drop, native services and OCI apps share ONE compile target — a systemd unit** (an OCI app is just a unit with a `RootDirectory=`/nspawn rootfs from the store). The table collapses to service / WASM / webview:

| Manifest key | Compile target (service [native OR OCI-unit] / WASM / webview) |
|---|---|
| `network: none` (default) | PrivateNetwork + **socket activation** (host-ns socket passed as FD — shell still reaches UI port; doubles as on-demand start); IPAddressDeny=any + IPAddressAllow=localhost for units needing in-netns sockets / no wasi:sockets import / CSP `connect-src 'self'` |
| `network.internet: true` | omit IPAddressDeny; score-penalized, review-gated |
| `network.internet: [domains]` | **filtering-proxy subsystem** (own phase): no direct egress + HTTP(S)_PROXY to a domain-allowlist proxy — Anthropic **sandbox-runtime** pattern (bwrap --unshare-net + localhost proxy pair; verified). IPAddressAllow is IP-only. WASM: wasi-http allowed-hosts. Deno's --allow-net = vocabulary prior art |
| `fs` (private state, implicit) | DynamicUser + StateDirectory + ProtectSystem=strict; OCI-unit: rootfs mounted read-only from the store, StateDirectory as the only writable path (`confinement.enable`-equivalent by construction) / one preopened dir |
| `fs.user-folders` | **runtime portal grant, never a manifest flag** — the user's file-picker choice IS the permission (Flatpak's one great lesson), compiled to a scoped BindPaths=/preopen/brokered access |
| `gpu` | DeviceAllow (=/dev/dri, or /dev/dxg under WSL GPU-PV) + DevicePolicy=closed + SupplementaryGroups; false → PrivateDevices |
| `resources` | MemoryMax/CPUQuota/TasksMax on the unit — binds directly for OCI units too (Podman's cgroup re-parent trap is gone) / wasmtime fuel+memory caps |
| `secrets: [names]` | systemd LoadCredential (never env/store) + sops-nix at rest; warden grants named credentials |
| `host.api: [scopes]` | warden-enforced: WIT world imports (WASM) / postMessage bridge implementing only granted scopes (webview); host.ai() stays the only AI seam at every tier |
| `privileged`/escape hatches | hard-reject at submission (solo-maintainer policy) |
| `meta.score` (computed) | HA-style visible 1–6 score from the manifest, shown before install (NixOS substitutes for HA's AppArmor components: confinement bonus + exposure-score gate — NixOS MAC story is weak) |

Prior-art lessons baked in: Flatpak (static flags get cargo-culted → few, coarse, review-gated; portals for user-meaningful grants), Home Assistant add-ons (visible score creates least-privilege pressure), Snap (reviewer-gated auto-connect for dangerous grants), Android (runtime prompts in context beat install-time lists). Hardening baseline appended to every unit — but TIERED: "sandboxer" services (agent harness, nspawn-launcher units) get a documented exemption set (bwrap/nspawn need namespaces the baseline blocks). Agent tools: each tool = its own DynamicUser socket-activated unit — the unit definition IS the grant; Stapelberg's microvm.nix coding-agent pattern in reserve as the hostile tier; Landlock for light per-invocation fs scoping.

## 8. Applet tiers (shell side)

**Final verdict (Figma's Realms escape; Caja archived; ShadowRealm unshipped at Stage 2.7): third-party code sharing the shell's DOM can NEVER be meaningfully sandboxed.**

- **Tier 0** first-party applets: full-trust React as today; host-API seam = engineering convention, not a security boundary (no SES lockdown — wrong tax).
- **Tier 1** vibe-coded logic: WASI 0.2 components host-side under **wasmtime** in the Rust backend (never in-webview; jco shims experimental). Versioned WIT world (Zed's api-version pattern); JS authors via QuickJS-in-WASM — **Extism** as pragmatic v1 SDK. No DOM; host renders their UI. Preopens + wasi-http allowed-hosts + fuel/memory caps (Shopify-proven).
- **Tier 2** community applets with custom UI: **one webview per applet**, own custom-protocol origin, ZERO Tauri IPC by default (CVE GHSA-57fm-592m-34r7: iframes reached IPC — dedicated webviews are the advisory's own recommendation), postMessage bridge implementing only granted scopes. WebView2: user-data-folder per TRUST CLASS (not per pane — memory); WKWebView: own WebContent process per webview by default.
- **Tier 3** anything wanting OS powers/services/runtimes: a substrate app, not a shell applet.

## 9. GUI verdict — React-in-Tauri stays (not close)

Wins 5/6 fixed criteria: full reuse; best dev loop; best LLM authorability (Web-Bench: largest corpus); only stack docking many foreign webviews with bespoke CSS; Nix-buildable frontend; official MAS path. All alternatives fail somewhere fatal: 6 Rust GUIs can't dock foreign webviews (Leptos now "lightly maintained"; gpui/Iced pre-1.0); Flutter has NO official Windows webview; Svelte/Solid forfeit the shipped shell for margins irrelevant in a desktop shell; Vue = only sidegrade with a dockview binding. **PWA-instead-of-shell rejected** (warden must be a local signed binary; kill switch must render when the VM is dead; browser can't provision WSL/VM or give Tier-2 isolation; Power Browser hits X-Frame-Options) — but **browser build = the unprivileged second client** (§2 mode 2). **Plasma/DE-as-GUI rejected** (warden inversion, C++/Qt maintenance, no MAS, design fidelity). Planning notes: React 19 + Compiler upgrade as its own later phase; **spike Tauri multiwebview early** (`unstable` flag, open resize bugs, Tauri 3.0 may reshape it ~2027); Verso/Servo = watch item; dockview single-maintainer = survivable fork risk.

## 10. Streamed native Linux apps (fourth pane type)

Any Linux desktop app (desktop LibreOffice, GIMP, Zotero…) runs headless in the substrate and streams into a webview pane — same plumbing as every other pane (a pane is a URL). **Pipeline:** linuxserver.io `baseimage-selkies` pattern (single app + Selkies WebSocket striped JPEG/H.264 — 60fps CPU-only, clipboard/audio/drag-drop; lsio publishes docker-libreoffice/docker-gimp today); Nix-declarable as systemd units, identical in WSL and the Mac VM. **Fallback:** bare KasmVNC 1.5 (GPL-2.0, H.264/AV1, best CJK/IME) — keep the pane contract protocol-agnostic so it's a drop-in swap. **Web-UI LibreOffice remains Collabora CODE** (single container, ~0.5–1.5 GB, ~100-line WOPI host); ZetaOffice (LibreOffice→WASM) = watch item. **Dead ends (verified, do not revisit):** WSLg window reparenting (msrdc-owned HWNDs, unsupported cross-process SetParent, Windows-only), waypipe (needs a host Wayland compositor; none on Win/Mac), Guacamole (extra transcode tier, no codec path), Wolf/Moonlight (no browser client). Risks: Selkies upstream maintainer-starved (bet is on lsio productization); IME/fractional-DPI/shortcut-capture polish is the unsolved 10%.

## 11. Security caveats + business models

- **WSL2 shared-kernel caveat (biggest):** all WSL2 distros share ONE utility VM and kernel (documented escape research exists) — Windows's "VM boundary" is softer than macOS's dedicated VM. Mitigation (post-Podman-drop): **nspawn/systemd hardening depth** (DynamicUser, seccomp SystemCallFilter, CapabilityBoundingSet, read-only store rootfs, PrivateNetwork) + the VM boundary — **honestly reflected in the security score** (gVisor's syscall-interception defense-in-depth is deferred with Podman, §13; if a future threat model demands it, runsc can return as an nspawn alternative or a dedicated Hyper-V VM becomes the hostile-agent tier).
- **Business models the architecture supports unchanged:** (1) open-core hosting (GitLab/Nextcloud model) — "Sourcerer Cloud" = one substrate per subscriber + browser client + auth/billing; tenant = flake + data volumes; idle-suspend mandatory for economics. (2) per-user cloud environments (Replit = Nix-per-user at scale, proof; Codespaces/JupyterHub/Kasm) — tenancy boundary = microVM per tenant (Firecracker/cloud-hypervisor; never nixos-containers — not a security boundary). (3) local core + paid cloud services (Nabu Casa/Obsidian Sync model): remote-access relay, manifest+data sync/backup, remote GPU. Sellable Nix-only features: fork/share/publish environments; generational rollback as a button; local⇄cloud symmetry. **Licensing decision eventually forced** (AGPL / source-available vs others hosting the core) — deliberate pass when the milestone gets real. Ordering: desktop → hosted Sourcerer → generic environments platform (last, not first).

## 12. Spikes (gate commitment; none scheduled)

- **A — substrate proof (VALIDATED 2026-08-02 as repo spike 010, A-lite scope):** import prebuilt image under private name → boot → externally-driven `nixos-rebuild switch` → `--rollback`. Landmines recorded in the spike README (pre-create storage parent; enforce minimum WSL version; elevated `wsl --update` flow). Fallback if NixOS-WSL itself disappoints: **system-manager on stock Ubuntu-WSL** (numtide/system-manager). Collabora/pane leg superseded by spike K.
- **B — GPU reality (Windows):** GPU app unit with `DeviceAllow=/dev/dxg` (WSL GPU-PV) inside the private distro — engine-less, no container toolkit. Kill-question: does the guest see and use the GPU through a hardened unit?
- **C — macOS:** vfkit/microvm.nix VM boot + Metal helper shape; **separate MAS spike**: sandboxed dev build + virtualization entitlement + boot NixOS aarch64 + curl guest service from webview (1 day, tests every MAS risk). Compare Apple Containerization framework.
- **D — portability proof:** export manifest + data on machine A → reconstruct on clean machine B, zero manual steps. Kill-question: does reconstruction converge?
- **E — Tauri multiwebview (VALIDATED 2026-08-02 as repo spike 011):** panes over full-size shell webview, DOM-aligned bounds, live tracking; fractional-DPI retest owed at P4.
- **F — Python/ML packaging:** LightRAG/OCR under uv2nix; fallback = run those components as engine-less OCI units (§5).
- **G — egress isolation: RETIRED (Podman drop, 2026-08-02)** — per-app networks/netavark/nftables no longer exist; `network:none` compiles to systemd `PrivateNetwork`+socket-activation / `IPAddressDeny`, which spike-free standard systemd behavior covers.
- **H — hardened runtime: RETIRED (Podman drop, 2026-08-02)** — gVisor-under-podman is gone; its slot is the P8 nspawn/systemd-hardening audit. Revisit runsc only if a future threat model demands syscall interception.
- **I — streamed-app pipeline:** one lsio-style Selkies app as an engine-less unit → pane; measure latency/IME/DPI.
- **K — engine-less OCI unit (NEW, replaces A's Collabora leg):** `dockerTools.pullImage` the digest-pinned Collabora CODE image into the store → run as a `RootDirectory=` (or nspawn `--oci-bundle`) systemd unit inside the existing `SourcererSpike` distro → reach its web UI on 127.0.0.1. Kill-question: do real-world images (users, /proc, tmpfs, entrypoint env) run under plain systemd confinement without an engine's compatibility shims?
- **Later research (no spike):** WASM tier runtime bake-off (wasmtime/Extism), capability-grant UX, filtering-proxy subsystem, auth module for remote clients, comin adoption vs in-app updater.

## 13. Rejected alternatives (recorded)

k3s/Kubernetes-in-VM (desktop-hostile complexity) · full-VM appliance images (opaque 20–60 GB, no dedup) · Flatpak/AppImage as the platform (Linux-only, no services) · pure-WASM (can't run LibreOffice-class apps) · native Nix on macOS host as product substrate (M1 pick reversed — §3) · nixos-containers (not a security boundary) · **Podman + `virtualisation.oci-containers` + quadlet-nix/compose2nix runtime translation (DROPPED 2026-08-02: engine-less OCI-units replace them — one enforcement dialect, no daemon, app rootfs inside the closure; compose demoted to a deferred single-container compatibility tier; multi-container compose apps deferred or curated Nix-native)** · **gVisor runsc as the community runtime (DEFERRED with Podman: defense-in-depth now nspawn/systemd + VM boundary, honestly scored; revisit on threat-model change)** · arion (dropped with Podman) · Electron & browser+daemon shells (warden/pane analysis, §9) · Plasma/any DE as GUI · same-realm JS sandboxing (SES/ShadowRealm/Caja) as the applet boundary · WSLg reparenting, waypipe, Guacamole, Wolf for panes · pnpm-on-Nix, dream2nix, Hydra, hosted Nix-CI (Garnix dead) · stock system.autoUpgrade.

## Key sources (per-section detail in session agent reports)

NixOS-WSL: github.com/nix-community/NixOS-WSL (2511.7.1; tarballBuilder; custom `--name`) · Determinate: determinate.systems/blog (installer schism, lazy trees, Nix 3.21.8) · confinement: nixpkgs nixos/modules/security/systemd-confinement.nix (PR #57519) · oci-containers/compose2nix/quadlet-nix (historical — Podman path dropped 2026-08-02, retained as research citations) · sandbox-runtime: github.com/anthropic-experimental/sandbox-runtime · HA score: developers.home-assistant.io/docs/apps/presentation · MAS precedents: apps.apple.com id1085114709 (Parallels ASE), id1538878817 (UTM); Tauri v2.tauri.app/distribute/app-store · Tauri iframe CVE: GHSA-57fm-592m-34r7 · WASM: zed.dev/blog/zed-decoded-extensions; extism.org; Figma plugin-security blog · Selkies/lsio: linuxserver.io Webtop 3.0 blog; docker-baseimage-selkies · KasmVNC: github.com/kasmtech/KasmVNC (1.5.0, 2026-07-29) · nixosTest-on-Actions: determinate.systems/blog/kvm-on-github-actions · WSL2 shared kernel: Trend Micro WSL2/Docker escape research; learn.microsoft.com WSL compare-versions · uv2nix: github.com/pyproject-nix/uv2nix · comin: github.com/nlewo/comin · microvm.nix: github.com/microvm-nix/microvm.nix; Stapelberg 2026-02-01 coding-agent-microvm post · Apple Containerization: github.com/apple/container · Replit-on-Nix: blog.replit.com/powered-by-nix.
