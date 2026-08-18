# Phase 8: Spike K — Nix-Native Substrate Service - Research

**Researched:** 2026-08-04
**Domain:** systemd service hardening (NixOS) vs. a self-sandboxing service (`coolwsd`), under the WSL2 kernel
**Confidence:** MEDIUM-HIGH — primary-source-verified on the two hardest questions (what `coolwsd` actually does, what WSL2's kernel supports), MEDIUM on how systemd's specific hardening directives will interact in practice (that interaction is exactly what the spike exists to observe)

<user_constraints>
## User Constraints (from CONTEXT.md)

### Locked Decisions

**Nix-native policy (milestone-level):**
- D-01: Nix-native is the rule. If a component is not in nixpkgs, it gets packaged. The community catalog is Nix-native by construction — an OCI image is never a valid submission format.
- D-02: OCI survives only as a time-boxed internal escape hatch, for first-party components that genuinely resist packaging (Phase 13's uv2nix descope trigger only). Tracked as debt.
- D-03: Rationale of record — provenance first, then store page-sharing at commercial scale (per-tenant microVMs share library pages only via virtio-fs/DAX over one read-only host Nix store, which requires identical store paths).
- D-04: Permissions are explicitly NOT part of the rationale — native and OCI compile to the same systemd unit since the Podman drop.
- D-05: Disk footprint is NOT part of the rationale either — amortizes per-image, not per-tenant.

**Evidence that forced the rescope:**
- D-06: `collabora-online` 25.04.9-4 (MPL-2.0) is in nixpkgs, with a full `services.collabora-online` module (`enable`/`package`/`settings`, `settings` maps declaratively onto `coolwsd.xml`). Verified via the `nixos` MCP 2026-08-04.
- D-07: Already in the binary cache for `x86_64-linux` — `/nix/store/82181wy8scpzh0fis39gjjjnzk5462c9-collabora-online-25.04.9-4`, 13.8 MB download / 42.0 MB unpacked, zstd.
- D-08: `services.jupyter` is likewise in nixpkgs (out of scope this phase — Phase 15).

**Spike design:**
- D-09: The kill-question is: does a service that sandboxes itself survive the §7 hardening baseline under WSL2's kernel? `coolwsd` forks jailed LibreOffice kernels using its own chroot + namespaces, which collides directly with `DynamicUser`, `ProtectSystem=strict`, `NoNewPrivileges`, `CapabilityBoundingSet` and `SystemCallFilter`.
- D-10: Full baseline first, then relax knob-by-knob. Record every concession with its reason. Rejected: "stock module then measure" and "minimum to boot."
- D-11: Collabora only — the hard case.
- D-12: Build in-substrate — nothing compiles, it substitutes from cache. No cross-machine transport.
- D-13: Reachability bar is curl + browser on 127.0.0.1 from the Windows host.

### Claude's Discretion
- Exact §7 directive list and the order in which directives are relaxed — derive from `CONTAINER-PLATFORM.md` §7 and record the order actually used.
- Whether findings get packaged as a `spike-findings-*` skill via `/gsd-spike --wrap-up`, or stay as the spike README. Default: wrap up.
- Whether to unregister the `SourcererSpike` distro at the end or leave it warm for Phase 10. Default: leave it registered.
- Timebox/abort handling if the hardening ladder proves bottomless: record the concessions reached and stop.

### Deferred Ideas (OUT OF SCOPE)
- `services.jupyter` as a second, contrasting service → Phase 15.
- The multiwebview pane leg (rendering Collabora in a Tauri pane) → Phase 15.
- The engine-less OCI unit spike (original Phase 8 framing) → only relevant if Phase 13's uv2nix escape hatch fires.
- Packaging work for anything not currently in nixpkgs → revisit per-component, not speculatively.
</user_constraints>

<phase_requirements>
## Phase Requirements

| ID | Description | Research Support |
|----|-------------|------------------|
| SPIKE-01 | `services.collabora-online` (nixpkgs, cached) runs inside the substrate distro under the §7 hardening baseline, with every relaxed directive and its reason recorded, and the web UI reachable on 127.0.0.1 from the Windows host | §7 directive list reconstructed below; `coolwsd`'s actual jailing mechanism verified against nixpkgs source + upstream `Kit.cpp` source at the exact pinned tag; WSL2 kernel capability confirmed against Microsoft's own published kernel config; concrete iteration loop (systemd drop-ins vs full rebuild) and reachability/verification commands given in Code Examples and Validation Architecture |
</phase_requirements>

## Summary

This phase is not really an "OCI unit" spike anymore (that framing is dead per D-01/D-02) — it is a **systemd-hardening compatibility spike** against one specific, already-packaged nixpkgs service. Three primary-source facts, verified directly against the actual nixpkgs module, the actual nixpkgs build derivation, and the actual upstream `coolwsd`/`Kit.cpp` source at the pinned version tag (`cp-25.04.9-4`), change the shape of the kill-question in ways the CONTEXT.md framing didn't anticipate:

1. **The nixpkgs `services.collabora-online` module ships with *zero* hardening.** `systemd.services.coolwsd.serviceConfig` sets only `User = "cool"`, `StateDirectory = "cool"`, `KillMode = "mixed"`, `KillSignal = "SIGINT"`, `TimeoutStopSec = 120`, `Restart = "always"`. No `ProtectSystem`, no `CapabilityBoundingSet`, no `NoNewPrivileges`, no `SystemCallFilter`. The full §7 baseline must be layered on top by the spike itself (via a systemd drop-in or a `configuration.nix` override) — there is no existing hardened starting point to "relax."
2. **The nixpkgs build explicitly disables the capability/setuid path** (`configureFlags = [ "--disable-setcap" ... ]`), and the module sets `mount_namespaces = lib.mkDefault true`. Upstream `Kit.cpp` confirms: with capabilities unavailable, `coolwsd` falls back to **unprivileged user+mount namespaces** (`unshare(CLONE_NEWUSER|CLONE_NEWNS)` inside its own process, then `chroot()` *inside that new namespace* where it already holds a full capability set local to the namespace, then drops all of them). This means the likely actual collision is **not** `CapabilityBoundingSet` (coolwsd needs no capabilities in the *outer* namespace at all under this build) — it is **`RestrictNamespaces`** (must permit at least `user` + `mnt`) and **`SystemCallFilter`** (must permit `unshare`, `clone`/`clone3`, `mount`, `umount2`, `chroot`, likely `setns`). This reframes D-09's kill-question: expect the fight to be over namespace/syscall filtering, not capabilities.
3. **`coolwsd` can silently degrade instead of failing.** If it decides namespaces/capabilities aren't available, it logs `LOG_WRN("Security warning: running without chroot jails is insecure.")` and keeps running **jail-less**. A green `curl` against the UI port is therefore not proof the hardening ladder succeeded — it could just as easily be the "minimum to boot" outcome D-10 explicitly rejects. The spike's validation must check jail status, not just port liveness (see Validation Architecture).

Separately, Microsoft's own published WSL2 kernel config (`microsoft/WSL2-Linux-Kernel`, current default branch) confirms `CONFIG_USER_NS=y`, `CONFIG_NAMESPACES=y` (all sub-namespace types), and `CONFIG_SECCOMP_FILTER=y` — the kernel-level capability the whole spike depends on is present. The open question the spike must answer empirically is the **runtime** posture: the `kernel.unprivileged_userns_clone` sysctl value inside NixOS-WSL, and whether systemd's own hardening directives, stacked on top of coolwsd's *own* internal namespace/chroot/seccomp logic, produce a working service, a jail-less-but-serving service, or a hard failure — at each rung of the ladder.

**Primary recommendation:** Apply the full baseline as a **systemd drop-in override** (fast iterate: `daemon-reload` + `restart`, no `nixos-rebuild` per rung) against the stock `coolwsd.service`/`coolwsd-systemplate-setup.service` units, starting from the literal directive list in `## Architecture Patterns → §7 Baseline Directive List` below. At each rung check three things, not one: (a) `systemctl status`/`journalctl` for hard failures, (b) the specific "running without chroot jails is insecure" log line (jail degraded, not blocked), (c) `curl` reachability. Only relax a directive when (a) or (b) fires, and record the exact log line that forced the relaxation — that log line *is* the evidence Phase 19's audit needs. Once the exemption set is found via drop-ins, capture it declaratively into `configuration.nix` and do one final `nixos-rebuild switch` to prove the real (non-drop-in) path also works, since that's the form Phase 15's compiler will actually emit.

## Architectural Responsibility Map

This project's tiers are the ones defined in `CONTAINER-PLATFORM.md` §1, not a generic web-app stack. Mapping this phase's capabilities onto them:

| Capability | Primary Tier | Secondary Tier | Rationale |
|------------|-------------|----------------|-----------|
| Hardening-baseline enforcement (§7 directives) | SUBSTRATE — systemd unit (native service tier, RUNTIME TIER 1) | — | `services.collabora-online` compiles to a plain systemd unit; hardening is a unit-file property, not something the shell or a container engine mediates (engine dropped 2026-08-02) |
| Jail/sandbox creation inside `coolwsd` (chroot + mount namespace per kit process) | SUBSTRATE — application code, self-managed | — | `coolwsd` manages its own child confinement via `unshare()`/`chroot()`; systemd's hardening directives sit *around* this, they don't replace it |
| Reachability from Windows host | SHELL ⇄ SUBSTRATE boundary (the "substrate connection" transport seam described in §2) | — | This spike proves the seam's *simplest* case (raw `curl`/browser on 127.0.0.1 via WSL2's default localhost forwarding) — no Tauri pane, no auth, per D-13 |
| Distro lifecycle (import/boot/rebuild/rollback) | SUBSTRATE provisioning (spike 010's mechanics) | Windows host (`wsl.exe` driver) | Reused wholesale from spike 010; this phase adds config-edit + hardening-iterate on top, not new lifecycle mechanics |
| Recorded exemption set (the deliverable) | Cross-cutting — consumed by Phase 15 (compiler) and Phase 19 (audit) | — | Not a runtime capability; the artifact itself is the product of this phase |

## Standard Stack

No new libraries are installed by this phase — everything comes from the existing nixpkgs binary cache or is already present in the substrate. The "stack" here is the set of system components in play.

### Core

| Component | Version | Purpose | Source |
|-----------|---------|---------|--------|
| NixOS-WSL | 2605.7.2 "Yearning Yarara" | Substrate distro image | [VERIFIED: sha256 recorded in spike 010 README] `e7180ad555fdcb8e1e057e2ef056de467603a5e502ff8531053738371be3f6b9` |
| `collabora-online` (nixpkgs) | 25.04.9-4, MPL-2.0 | The service under test | [CITED: 08-CONTEXT.md D-06/D-07, verified 2026-08-04 via the `nixos` MCP in the discuss-phase session] — this research session could not re-invoke the `nixos` MCP directly (tool not exposed to this agent's toolset this session); re-verify at plan/execution time per CLAUDE.md's mandate before locking any option name into a task |
| `services.collabora-online` NixOS module | tracks nixpkgs unstable | Declarative `enable`/`package`/`port`/`settings`/`aliasGroups`/`extraArgs` surface; `settings` merges onto `coolwsd.xml` via `yq`/`jq` at build time | [VERIFIED: fetched directly from `raw.githubusercontent.com/NixOS/nixpkgs/master/nixos/modules/services/web-apps/collabora-online.nix`, 2026-08-04] |
| systemd | ships with NixOS-WSL image | Init system inside the substrate (`boot.systemd = true` is the NixOS-WSL default) and the hardening enforcement mechanism itself | [VERIFIED: `nix-community/NixOS-WSL` `modules/wsl-conf.nix` — `boot.systemd` option default `true`] |
| `wsl.exe` | whatever ships with the Windows box (spike 010 measured 2.7.11 after a forced update) | Windows-side driver for every guest interaction | [CITED: spike 010 README] |

### Supporting

| Component | Purpose | When Relevant |
|-----------|---------|----------------|
| `yq-go` / `jq` (nixpkgs, pulled transitively by the module) | Merges `services.collabora-online.settings` into the shipped default `coolwsd.xml` at build time | Only if the spike needs to set `settings.net.listen = "loopback"` or similar — see Reachability below |
| `curl` (inside the substrate, or from Windows via any client) | D-13's reachability bar | Every rung of the ladder — cheapest liveness probe |
| `journalctl -u coolwsd -u coolwsd-systemplate-setup` | Primary forensic signal for each rung | Every rung |
| `systemd-analyze security coolwsd.service` | Machine-readable exposure score + per-directive breakdown | Useful cross-check once a rung boots, but see Pitfall — its score reflects *declared* hardening, not whether `coolwsd` degraded to jail-less operation |

### Alternatives Considered

| Instead of | Could Use | Tradeoff |
|------------|-----------|----------|
| Manual `RestrictNamespaces`/`SystemCallFilter`/`CapabilityBoundingSet` tuning (D-10's mandated approach) | NixOS's `systemd.services.<name>.confinement.enable` (the "NixOS-only superpower" §7 calls out — bind-mounts exactly the service's closure into a `tmpfs` chroot) | `confinement.enable` is a *different* mechanism (systemd-level `RootDirectory=` + closure bind-mounts) from what D-10 asks for (relaxing the D-09-listed directives one at a time). It's worth trying as a documented **alternative rung** after the D-10 ladder completes, since §7 explicitly names it — but it is not a substitute for the ladder itself, since it changes what "the baseline" means rather than relaxing it. Flag as a stretch goal, not the spike's primary path. |
| systemd drop-in override iteration | Full `nixos-rebuild switch` per rung | Drop-ins (`systemctl edit` or a hand-written unit under `/etc/systemd/system/coolwsd.service.d/`) + `daemon-reload`+`restart` iterate in ~seconds. A full rebuild took 109s in spike 010 for a trivial package add — multiplied over "10+ iterations" (per the research brief) that's 15-20+ minutes of pure wait versus under two. Use drop-ins to *find* the exemption set, one final `nixos-rebuild switch` with the found set captured in `configuration.nix` to *prove* the declarative path (Phase 15's compiler will emit declarative config, not drop-ins). |

**Installation:** no `npm install`/`pip install` equivalent. Inside the substrate's `/etc/nixos/configuration.nix`:
```nix
services.collabora-online.enable = true;
# settings/hardening additions go here once the ladder is found (see Code Examples)
```
then `nixos-rebuild switch` (first pass — pulls `collabora-online` + `libreoffice-collabora` closure from the binary cache, no compiling per D-12).

**Version verification:** This session verified the module and derivation directly against nixpkgs source (`raw.githubusercontent.com/NixOS/nixpkgs/master/...`), which is authoritative but reflects `master`/unstable at fetch time, not necessarily the exact channel pin the substrate will use. Before planning locks a channel/commit, re-run `nixos-rebuild switch --dry-run` inside the actual `SourcererSpike` distro (or re-verify via the `nixos` MCP) to confirm `collabora-online` 25.04.9-4 is still what the pinned channel resolves to.

## Package Legitimacy Audit

**Not applicable in the npm/pip/cargo sense this protocol targets.** This phase installs no packages from a language-ecosystem registry — `collabora-online` is pulled from the official NixOS binary cache (`cache.nixos.org`) as a content-addressed store path built by nixpkgs CI from the upstream `CollaboraOnline/online` GitHub source (`fetchFromGitHub`, pinned by commit-hash-derived `hash =` in the derivation — verified directly, see `pkgs/by-name/co/collabora-online/package.nix` excerpt above). There is no `slopcheck`-equivalent hallucination risk here: the package name, module path, and options were all confirmed by reading the actual nixpkgs source tree, not by search-engine or training-data recall of a plausible-sounding attribute path.

The one open provenance gap: this research session could not independently call the `nixos` MCP tool (unavailable in this agent's toolset), so D-06/D-07's "verified via `nixos` MCP" claim is inherited from the prior discuss-phase session, not re-confirmed here. Planner action: re-run the `nixos` MCP check (`mcp__nixos__nix` action `info`, query `collabora-online`, channel matching whatever the substrate pins) before finalizing the plan, per CLAUDE.md's standing instruction to verify option/package names against the `nixos` MCP before writing config.

## Architecture Patterns

### §7 Baseline Directive List (reconstructed — CONTAINER-PLATFORM.md does not print one single itemized checklist)

`CONTAINER-PLATFORM.md` §7 states the general principle ("Hardening baseline appended to every unit — but TIERED: 'sandboxer' services... get a documented exemption set") and gives per-manifest-key compile targets in its table; the *specific directive names* for "the full baseline" are scattered across §7's table, §11's mitigation sentence, and 08-CONTEXT.md's D-09 (which is itself paraphrasing the source doc's vocabulary for this exact spike). The literal union, which is what D-10 means by "the complete §7 baseline":

| # | Directive | Source in CONTAINER-PLATFORM.md | Purpose here |
|---|-----------|----------------------------------|---------------|
| 1 | `DynamicUser=yes` | §7 `fs` row; §11; D-09 | Replaces the module's static `User = "cool"` |
| 2 | `ProtectSystem=strict` | §7 `fs` row; D-09 | Whole-tree read-only except `/dev`,`/proc`,`/sys`; `StateDirectory=cool` is the carve-out |
| 3 | `NoNewPrivileges=yes` | D-09 | Blocks privilege gain via execve — should be low-risk given `--disable-setcap` (no setuid binaries in the closure at all) |
| 4 | `CapabilityBoundingSet=` (minimal/empty) | §11; D-09 | The directive D-09 expected to be the blocker; primary-source evidence below suggests it is *not*, since coolwsd's jailing needs no outer capabilities under `mount_namespaces=true` |
| 5 | `SystemCallFilter=` (an allow-list, e.g. `@system-service`) | §11; D-09 | Expected actual blocker — must permit `unshare`, `clone`/`clone3`, `mount`, `umount2`, `chroot`, `pivot_root`(if used), `setns` |
| 6 | `PrivateNetwork=yes` + socket activation / `IPAddressDeny=any`+`IPAddressAllow=localhost` | §7 `network: none` row (the default tier) | Directly in tension with D-13 (must still be reachable on 127.0.0.1) — see Reachability section; this is the one directive pairing the source doc *itself* explains how to keep working under confinement (host-namespace socket passed as an FD) |
| 7 | `StateDirectory=cool` | already present in the stock module; kept as the writable exception under `ProtectSystem=strict` | Not itself a restriction — record it as already-satisfied, not "relaxed" |

`RestrictNamespaces=` is **not explicitly named** in the source doc's §7/§11/D-09 text, but is the systemd directive that most precisely governs "can this process call `unshare()`/`clone()` to create new namespace types" — given the primary-source finding below (coolwsd's jail depends entirely on being able to create user+mount namespaces), this directive is very likely load-bearing even though the source doc doesn't name it by name. Treat it as an implied member of "`CapabilityBoundingSet`-and-`SystemCallFilter`-adjacent" territory in D-09's phrasing, and add it to the baseline explicitly — flag this addition to the user/planner as a research-session judgment call, not something copied verbatim from CONTAINER-PLATFORM.md.

**Confidence on this list:** MEDIUM. The individual directive *names* are HIGH confidence (quoted/reconstructed verbatim from the source doc). Whether this exact 6-7-directive set is what CONTAINER-PLATFORM.md's authors meant by "the complete baseline" for a `fs`+`network`-needing service like coolwsd is a reasonable reconstruction, not a verbatim single list — flag as an Open Question for the planner to confirm against the source doc's intent, or accept this reconstruction as canonical for the spike.

### `coolwsd`'s self-sandboxing mechanism (verified against upstream source at the pinned tag `cp-25.04.9-4`)

```
# Source: raw.githubusercontent.com/CollaboraOnline/online/cp-25.04.9-4/kit/Kit.cpp
# (fetched directly, 2026-08-04 — this is the exact version nixpkgs 25.04.9-4 builds)

if (capabilities available OR mount_namespaces enabled):
    unshare(CLONE_NEWUSER | CLONE_NEWNS)   # new user+mount namespace, unprivileged
    ... build jail tree (bind mounts) ...
    LOG_INF("chroot(\"" << jailPathStr << "\")")
    chroot(jailPathStr)                     # chroot INSIDE the new namespace
    chdir("/")
    if (usingMountNamespace):
        # "We have a full set of capabilities in the namespace so drop them all"
        dropAllCapabilities()
    else:
        dropCapability(CAP_SYS_CHROOT); dropCapability(CAP_FOWNER); dropCapability(CAP_CHOWN)
else:  # noCapabilities set, namespaces unavailable
    LOG_WRN("Security warning: running without chroot jails is insecure.")
    # continues running WITHOUT any jail — this is a silent degrade, not a failure
```

**What this means for the spike:**
- The nixpkgs build (`--disable-setcap`) forces the **namespace path** — there is no setuid/setcap fallback available in this build at all. `coolwsd` either gets working unprivileged user+mount namespaces, or it degrades to jail-less.
- Because `unshare(CLONE_NEWUSER)` grants the *creating* process a full capability set **inside its own new namespace** (standard Linux `user_namespaces(7)` semantics, independent of what capabilities it holds in the outer/init namespace), `coolwsd` needs **no outer capabilities at all** to build its jail — contrary to D-09's framing, `CapabilityBoundingSet=` (even set to empty) is not expected to be the blocker.
- The actual expected collision surface: (a) whether the kernel allows unprivileged `CLONE_NEWUSER` at all (`kernel.unprivileged_userns_clone` sysctl — kernel *support* confirmed present, runtime *sysctl value* inside NixOS-WSL is unverified, first thing to check), (b) whether `RestrictNamespaces=` on the unit permits `user`+`mnt`, (c) whether `SystemCallFilter=` permits the syscalls `unshare`/`clone`/`clone3`/`mount`/`umount2`/`chroot`.
- `coolwsd` also self-reports jail status via admin-console query params baked into its own startup URL construction: `adms_contained=ok|uncontained`, `adms_info_namespaces=true|false`, `adms_seccomp=ok|none`, `adms_bindmounted=ok|slow|not_recommended` (source: `Kit.cpp` ~line 3915-3925). These are a genuinely better verification signal than port-liveness alone — see Validation Architecture.

### WSL2 kernel capability check (verified against Microsoft's own published kernel config)

```
# Source: raw.githubusercontent.com/microsoft/WSL2-Linux-Kernel (default branch), Microsoft/config-wsl
# fetched directly, 2026-08-04

CONFIG_NAMESPACES=y
CONFIG_USER_NS=y
CONFIG_UTS_NS=y / CONFIG_IPC_NS=y / CONFIG_PID_NS=y / CONFIG_NET_NS=y
CONFIG_SECCOMP=y
CONFIG_SECCOMP_FILTER=y
CONFIG_CGROUPS=y (+ CGROUP_PIDS, CGROUP_DEVICE, CGROUP_FREEZER, MEMCG, BLK_CGROUP, CPUSETS...)
```

Kernel-level support for everything the §7 baseline and coolwsd's own jailing need is present. **This does not settle the runtime question** — `kernel.unprivileged_userns_clone` is a *sysctl*, not a compile-time option, and its default value inside NixOS-WSL specifically is not verified by this research (NixOS in general tends to leave it permissive since Nix's own build sandbox depends on unprivileged user namespaces, but "tends to" is not "confirmed for this image" — first command the spike should run). Confidence: HIGH on kernel *capability*, MEDIUM on runtime *posture* (untested).

### `coolwsd` reachability & the `network:none` tension (D-13 vs §7's default tier)

The module's `settings.net.listen` default is `"any"` (binds `0.0.0.0`); it accepts `"loopback"` explicitly (source: upstream `coolwsd.xml.in` at the pinned tag, `<listen type="string" default="any" ...>any</listen>`). For a hardened, D-13-compliant unit:

```nix
# Source: reconstructed from nixpkgs module (settings passthrough) + upstream coolwsd.xml.in schema
services.collabora-online.settings.net.listen = "loopback";
```

§7's own text describes exactly the pattern needed to reconcile `PrivateNetwork=yes` (no outer network access) with "the shell still reaches UI port": a **socket-activation unit** where the listen socket is created in the **host** network namespace by systemd and passed to the confined process as an inherited FD (`Sockets=`, `ListenStream=127.0.0.1:9980` on a paired `.socket` unit). This is architecturally elegant but **unverified whether `coolwsd` supports systemd socket activation** (`sd_listen_fds()`/`LISTEN_FDS`) — nothing in the fetched source excerpts confirms this, and it wasn't practical to grep the full ~150KB `Kit.cpp`/`COOLWSD.cpp` tree exhaustively this session. Treat as an **open question the spike must test directly**: try the socket-activation pattern first (cleanest §7 compliance); if `coolwsd` doesn't support inherited listen FDs, the fallback is to accept `PrivateNetwork=no` + `IPAddressDeny=any`+`IPAddressAllow=localhost` as a recorded, reasoned concession (which is D-10's whole point — a concession with a documented reason is a valid outcome, not a failure).

WSL2's default **localhost forwarding** (Windows → WSL2 VM, on by default, disableable via `.wslconfig`'s `[wsl2] localhostForwarding=false`) is what makes `curl http://127.0.0.1:9980` from the *Windows* side reach a service bound to loopback *inside* the distro — confirm this hasn't been disabled on the target box before attributing an unreachable UI to hardening.

### Recommended iteration loop

```bash
# Fast rung-to-rung iteration (seconds, not the 109s/rung a full nixos-rebuild costs):
wsl.exe -d SourcererSpike -u root -- sh -c '
  mkdir -p /etc/systemd/system/coolwsd.service.d
  cat > /etc/systemd/system/coolwsd.service.d/override.conf <<'"'"'EOF'"'"'
[Service]
DynamicUser=yes
ProtectSystem=strict
NoNewPrivileges=yes
CapabilityBoundingSet=
RestrictNamespaces=~cgroup net pid ipc uts
SystemCallFilter=@system-service unshare mount umount2 chroot pivot_root setns clone clone3
EOF
  systemctl daemon-reload
  systemctl restart coolwsd
  sleep 2
  systemctl status coolwsd --no-pager
  journalctl -u coolwsd -n 40 --no-pager
'
```

Note the `RestrictNamespaces=~cgroup net pid ipc uts` starting point *already* permits `user`+`mnt` (the `~` form means "deny the listed types, allow everything else" per `systemd.exec(5)`) — i.e. this example intentionally starts one rung *past* a maximally-restrictive baseline (which would be plain `RestrictNamespaces=yes`, denying all types) specifically because the coolwsd source analysis above makes `user`+`mnt` the near-certain requirement. D-10 asks for the *full* baseline first — if the plan wants a stricter literal start, begin with `RestrictNamespaces=yes` (deny all) and record that as rung 0's failure before loosening to the `user`+`mnt`-permitting form shown above.

Once a working set is found, capture it declaratively for the "real" proof pass:
```nix
# /etc/nixos/configuration.nix inside SourcererSpike
systemd.services.coolwsd.serviceConfig = {
  DynamicUser = true;
  ProtectSystem = "strict";
  NoNewPrivileges = true;
  CapabilityBoundingSet = [ ];  # or the found minimal set
  RestrictNamespaces = [ "~cgroup" "net" "pid" "ipc" "uts" ];  # or the found set
  SystemCallFilter = [ "@system-service" "unshare" "mount" "umount2" "chroot" "pivot_root" "setns" "clone" "clone3" ];
};
```
then `nixos-rebuild switch` once, to prove the module-override form (what Phase 15's compiler will actually generate) also works — not just the drop-in.

### Anti-Patterns to Avoid

- **Treating a green `curl` as ladder success.** Per the `coolwsd` degrade-path finding above, a reachable UI can mean "hardening survived" or "coolwsd quietly gave up on the jail." Always cross-check the jail-status log line / `adms_*` params before recording a rung as passing.
- **Starting from `RestrictNamespaces=yes` (deny-all) and never explaining why it had to move to a `user`+`mnt`-permitting form.** D-10 requires the *reason* for every relaxation recorded — "it didn't work" is not a reason; the specific `journalctl`/seccomp-audit line is.
- **Iterating via full `nixos-rebuild switch` for every rung.** Correct per D-12's "nothing compiles" framing for the whole spike, but 10+ full rebuilds (even cache-only ones) will eat the phase's time budget for no benefit over drop-ins during exploration.

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|--------------|-----|
| Confinement of a service to its own Nix closure | A bespoke chroot/bind-mount script | `systemd.services.<name>.confinement.enable` (NixOS's own module, §7 calls it out by name) | It's the "NixOS-only superpower" — knows the Nix closure automatically; worth trying as a documented alternative rung, not reinventing |
| Detecting whether a systemd hardening directive actually took effect | Manual `/proc/<pid>/status` capability parsing | `systemd-analyze security coolwsd.service` + `systemctl show -p <Directive> coolwsd` | Built-in, per-directive breakdown; still needs the coolwsd-specific jail-status cross-check above, but don't reimplement systemd's own introspection |
| Reachability verification | A custom TCP probe script | `curl -sf http://127.0.0.1:9980/` (or the coolwsd `/hosting/discovery` endpoint, which returns XML and is a stronger signal than a bare TCP connect) | Matches D-13's stated bar exactly; a bare port-open check doesn't distinguish "coolwsd answered" from "something else is listening" |
| Distro import/boot/rebuild driving | New Windows-side automation | Spike 010's `run.sh` pattern (timed, tee-logged, `wsl.exe`-only) | Directly reusable; this phase's driver is a descendant, not a rewrite |

**Key insight:** almost everything this spike needs already exists — the nixpkgs module, the confinement module, systemd's own audit tooling, and spike 010's driver pattern. The only genuinely new work is the hardening ladder itself and the forensic recording of it, which is unavoidably manual/iterative by the nature of D-10's methodology.

## Common Pitfalls

### Pitfall 1: Attributing "port unreachable" to hardening when it's WSL2 localhost forwarding
**What goes wrong:** `.wslconfig`'s `localhostForwarding` setting (or a firewall rule) blocks the Windows→WSL2 hop, and the spike log records a false "hardening blocked reachability" finding.
**Why it happens:** the failure mode (connection refused/timeout from Windows) looks identical whether the block is at coolwsd, at the unit's `PrivateNetwork`, or at the WSL2 forwarding layer.
**How to avoid:** verify reachability *from inside the distro* (`curl http://127.0.0.1:9980/` run via `wsl.exe -d SourcererSpike --`) before testing from Windows. If in-guest curl works but Windows-side doesn't, the cause is outside this spike's scope (WSL networking config, not §7 hardening) — record it as such, don't fold it into the exemption set.
**Warning signs:** in-guest curl succeeds, Windows-side curl times out (not "connection refused" — a refusal usually means the hardening changed the bind address; a timeout usually means forwarding).

### Pitfall 2: Treating `coolwsd`'s silent jail-degrade as a passing rung
**What goes wrong:** the ladder logs a rung as "PASS" because the UI loaded, when `coolwsd` actually fell back to jail-less operation (`LOG_WRN("Security warning: running without chroot jails is insecure.")`).
**Why it happens:** `coolwsd` was deliberately designed to keep serving rather than hard-fail when isolation isn't available — a reasonable choice for the upstream project, a trap for a spike whose entire purpose is measuring isolation depth.
**How to avoid:** grep `journalctl -u coolwsd` for the exact warning string on every rung, or fetch `adms_info_namespaces`/`adms_contained` from the kit's own reported state before recording a pass.
**Warning signs:** a rung "passes" immediately after a relaxation that *shouldn't* have mattered (e.g. relaxing `MemoryMax`, which has nothing to do with jailing, "fixing" reachability) — that's a sign the jail was already silently disabled by an earlier rung and nothing since has actually been the blocker.

### Pitfall 3: Confusing `coolwsd`'s own internal seccomp with systemd's `SystemCallFilter`
**What goes wrong:** `coolwsd` has `hasSeccomp`-tracked internal seccomp behavior of its own (visible in the `adms_seccomp=ok/none` telemetry param), layered on top of whatever systemd's `SystemCallFilter=` allows. A syscall failure could originate from either layer, and the two produce different, easily-conflated `journalctl` signatures (systemd's seccomp kill shows as the unit being terminated by `SIGSYS`/`Killed` with a `systemd[1]:` log source; coolwsd's own internal seccomp failure shows as an application-level log line from the coolwsd/coolkit process itself).
**Why it happens:** two independent seccomp filters stack (kernel enforces the intersection); neither log source says "the *other* filter also exists."
**How to avoid:** when a syscall-related failure appears, check both the systemd journal's `SIGSYS`/`Seccomp` kill markers *and* coolwsd's own `adms_seccomp` telemetry / internal log lines before deciding which filter to relax.
**Warning signs:** a `SystemCallFilter` relaxation that should have fixed a specific syscall doesn't change the observed failure at all — likely means the block is coolwsd's own internal filter, not systemd's.

### Pitfall 4: Editing `configuration.nix` mid-iteration and losing track of what's a drop-in vs. what's declarative
**What goes wrong:** the exploratory systemd drop-in and the "real" `configuration.nix` diverge — a working exemption set found via drop-in never gets captured into git, or a stale drop-in survives a `nixos-rebuild switch` and masks what the declarative config actually produces.
**Why it happens:** drop-ins under `/etc/systemd/system/coolwsd.service.d/` persist independently of `configuration.nix` and are not cleared by `nixos-rebuild switch` unless the module itself manages that path (it doesn't, by default).
**How to avoid:** `rm -rf /etc/systemd/system/coolwsd.service.d/` before the final proof-pass rebuild, so the declarative `configuration.nix` alone is what's tested.
**Warning signs:** the "final" `nixos-rebuild switch` behaves identically to the drop-in exploration even though `configuration.nix` wasn't touched for that rung — the drop-in is still active and shadowing the declarative config.

## Code Examples

See **Architecture Patterns → Recommended iteration loop** above for the full drop-in + `journalctl` pattern and the declarative capture pattern. One additional verification snippet:

### Discovery-endpoint check (stronger signal than a bare port probe)
```bash
# Source: pattern reconstructed from Collabora's documented /hosting/discovery
# endpoint (standard COOL/CODE deployment convention, not spike-specific)
wsl.exe -d SourcererSpike -- curl -sf http://127.0.0.1:9980/hosting/discovery | head -5
```
A COOL/CODE deployment answers this endpoint with an XML capabilities document when the WOPI-facing HTTP layer is actually up — a positive here is a stronger "the service is really serving" signal than a bare TCP connect, though it does **not** by itself confirm jail status (see Pitfall 2 — check `journalctl` for the chroot-warning line separately).

## State of the Art

| Old framing (original Phase 8) | Current framing (this research) | When Changed | Impact |
|---|---|---|---|
| "Engine-less OCI unit": `dockerTools.pullImage` Collabora's CODE image, run as `RootDirectory=`/nspawn unit | Stock `services.collabora-online` from nixpkgs, run as a plain systemd unit | 2026-08-04 (D-01/D-02, forced by D-06/D-07 evidence) | The kill-question moved from "do OCI images run engine-less" to "does a self-sandboxing native package survive hardening" — a materially different technical question, answered above |
| Assumed `coolwsd` needs `CAP_SYS_CHROOT`/`CAP_SYS_ADMIN` (D-09's original framing) | `coolwsd`'s nixpkgs build (`--disable-setcap`) forces the unprivileged-namespace path, which needs **no outer capabilities** — the likely blocker is `RestrictNamespaces`/`SystemCallFilter`, not `CapabilityBoundingSet` | This research session, verified against upstream `Kit.cpp` source | Materially changes which directive the ladder should expect to concede first — plan the ladder ordering accordingly |

**Deprecated/outdated:** the entire "engine-less OCI unit" framing (`dockerTools.pullImage`, digest-pinned image ingestion for Collabora) — superseded milestone-wide by the Nix-native decision; do not resurrect it for this phase.

## Assumptions Log

| # | Claim | Section | Risk if Wrong |
|---|-------|---------|---------------|
| A1 | The reconstructed "§7 baseline directive list" (7 items) is the complete/correct set CONTAINER-PLATFORM.md's authors intended for D-10's "full baseline first" — the source doc itself doesn't print one canonical list | Architecture Patterns → §7 Baseline Directive List | Low-medium: if the planner's intended baseline is broader or narrower, the ladder's starting rung is miscalibrated, but D-10's methodology (relax-and-record) self-corrects as long as the starting point is *at least as strict* as intended |
| A2 | `RestrictNamespaces` is implicitly part of "the baseline" even though CONTAINER-PLATFORM.md never names it | Architecture Patterns → §7 Baseline Directive List | Low: this is a defensible technical inference from the coolwsd-jailing analysis, but it's this session's addition, not a copied fact — flag to the user before locking it as a task requirement |
| A3 | `kernel.unprivileged_userns_clone` defaults to permissive inside NixOS-WSL specifically (kernel *capability* is confirmed; the *sysctl default for this image* is not) | Architecture Patterns → WSL2 kernel capability check | Medium: if the sysctl is actually restrictive by default, the very first rung fails for a reason unrelated to systemd hardening at all — must be checked and possibly set explicitly (`boot.kernel.sysctl."kernel.unprivileged_userns_clone" = 1;`) before the ladder proper begins |
| A4 | `coolwsd` supports systemd socket activation (`sd_listen_fds()`) for the `PrivateNetwork=yes`+passed-FD reachability pattern §7 describes | Architecture Patterns → Reachability & the network:none tension | Medium: if unsupported, that specific §7-compliant pattern is unavailable and the concession (accepting `PrivateNetwork=no` + `IPAddressAllow=localhost`) needs to be recorded instead — not a blocker to the spike, but changes what "ideal" looks like |
| A5 | The module and package source fetched from nixpkgs `master` this session exactly matches what the actual pinned/channel-resolved substrate will build (this session fetched `master`, not a specific channel tag) | Standard Stack → Version verification | Low-medium: nixpkgs module code for a stable, already-merged service rarely churns week to week, but re-verify against the actual channel before the plan locks specific option names |

**If this table is empty:** N/A — table is populated; several items need planner/user confirmation before becoming locked task requirements.

## Open Questions

1. **Does `RestrictNamespaces` need to be in the D-10 "full baseline," or is it out of scope since CONTAINER-PLATFORM.md never names it?**
   - What we know: the coolwsd source analysis strongly implies it's load-bearing.
   - What's unclear: whether the planner should treat it as part of "the complete §7 baseline" (D-10's phrasing) or as a separate, spike-discovered addition.
   - Recommendation: include it in the starting rung regardless (per Anti-Patterns above, starting *too* strict and recording the relaxation is always safe under D-10's methodology; starting too loose risks missing a finding).

2. **Is `kernel.unprivileged_userns_clone` permissive by default in NixOS-WSL 2605.7.2?**
   - What we know: the WSL2 kernel *supports* `CONFIG_USER_NS`; NixOS in general tends to leave the sysctl permissive since Nix's build sandbox itself depends on it.
   - What's unclear: the actual runtime value inside this specific image, unverified.
   - Recommendation: make this literally the first command the spike's driver script runs (`sysctl kernel.unprivileged_userns_clone`), before touching `coolwsd` at all — if it's `0`, that's a distinct, non-hardening-related finding to record separately.

3. **Does `coolwsd` support systemd socket activation?**
   - What we know: nothing found confirming or denying it in the source excerpts fetched this session.
   - What's unclear: whether the "PrivateNetwork + passed FD" §7 pattern is achievable for this specific service, or whether the concession ladder should assume `PrivateNetwork=no` from the start for the `network` dimension.
   - Recommendation: test it directly as an early rung (cheap to try, `Sockets=coolwsd.socket` + `ListenStream=127.0.0.1:9980` in the unit) — a failure here is itself a valid, recordable finding, not a spike blocker.

## Environment Availability

This research session ran on the NixOS dev host (no `wsl.exe`, no `/mnt`), per the two-host milestone split (STATE.md). The execution host for this phase is the **Windows box**, which this session could not probe directly.

| Dependency | Required By | Available (this session's host) | Available (execution host) | Fallback |
|------------|------------|:---:|:---:|----------|
| `wsl.exe` | Driving the substrate at all | ✗ (not this host) | Unconfirmed this session — spike 010 measured WSL 2.7.11 on 2026-08-03; re-verify current version before running (spike 010's minimum-version lesson: 2605-line images fail "Catastrophic failure" on WSL < ~2.6) | none — hard blocker if absent below the minimum version |
| `SourcererSpike` distro (registered, warm) | Skipping the re-import leg | ✗ | **Unconfirmed — per STATE.md blocker, the local `nixos.wsl` image was deleted 2026-08-03; budget a full re-import leg unless proven still registered** | Re-download NixOS-WSL 2605.7.2, sha256-verify against `e7180ad555fdcb8e1e057e2ef056de467603a5e502ff8531053738371be3f6b9`, re-run spike 010's `run.sh`-style import |
| `nixos` MCP tool | Verifying option/package names before locking them in the plan | ✗ (not exposed to this research agent) | Unconfirmed — should be available in the planning/execution session per CLAUDE.md's standing instruction | Cross-check against nixpkgs source directly via `raw.githubusercontent.com` (what this session did) as a fallback |
| Binary cache reachability (`cache.nixos.org`) from the Windows box | D-12's "nothing compiles" premise | N/A | Assumed reachable (spike 010 pulled ~103 MiB from cache.nixos.org successfully on 2026-08-02/03) | If cache is unreachable, the phase's whole "nothing compiles" premise breaks — treat as a hard blocker, not a soft one, if it recurs |

**Missing dependencies with no fallback:**
- A WSL version below the NixOS-WSL 2605-line's minimum — no software fallback, requires the elevated `wsl --update` flow spike 010 documented.

**Missing dependencies with fallback:**
- `SourcererSpike` distro not registered → re-import leg (budgeted, per CONTEXT.md's "Execution host" note in the phase brief).
- `nixos` MCP unavailable → direct nixpkgs source verification (this session's own fallback, already exercised).

## Validation Architecture

### Test Framework
This phase has no code-level test framework (it's a systemd/Nix configuration spike, not an application with unit tests). The "test framework" is the forensic driver-script + log-grep pattern established by spike 010/`CONVENTIONS.md`.

| Property | Value |
|----------|-------|
| Framework | None (shell-driven forensic verification, per `.planning/spikes/CONVENTIONS.md`) |
| Config file | `configuration.nix` inside the `SourcererSpike` distro (declarative end-state); systemd drop-ins during exploration |
| Quick run command | `wsl.exe -d SourcererSpike -u root -- sh -c 'systemctl daemon-reload && systemctl restart coolwsd && sleep 2 && journalctl -u coolwsd -n 30 --no-pager'` |
| Full suite command | `wsl.exe -d SourcererSpike -u root -- nixos-rebuild switch` (the declarative proof-pass, run once at the end per directive-set) |

### Phase Requirements → Test Map

| Req ID | Behavior | Test Type | Automated Command | File Exists? |
|--------|----------|-----------|-------------------|-------------|
| SPIKE-01a | `services.collabora-online` runs from cache, nothing compiles | smoke | `wsl.exe -d SourcererSpike -u root -- nixos-rebuild switch --dry-run` (check for `will be built:` — expect empty, only `will be fetched:`) | ❌ — write into the spike's `run.sh`, Wave 0 |
| SPIKE-01b | Full §7 baseline applied first, fails, then relaxed knob-by-knob with reasons recorded | manual + log-grep | `journalctl -u coolwsd -n 60 --no-pager` after each drop-in change, grep for `SIGSYS`/`Seccomp`/`Operation not permitted`/the chroot-warning string | ❌ — this *is* the spike's core work, no pre-existing file |
| SPIKE-01c | Web UI reachable on 127.0.0.1 from Windows | smoke | (from Windows, outside WSL) `curl.exe -sf http://127.0.0.1:9980/hosting/discovery` | ❌ — Wave 0 |
| SPIKE-01d | Exemption set written down, evidence not guesswork | artifact check | the spike README's Investigation Trail table (per `CONVENTIONS.md`'s binding artifact form) — verified by human read, not automatable | ❌ — the deliverable itself |

### Sampling Rate
- **Per rung (drop-in iteration):** the "Quick run command" above — restart + `journalctl` grep, seconds per rung.
- **Per confirmed exemption-set change:** re-run the discovery-endpoint curl (Code Examples) to confirm reachability wasn't silently broken by the latest relaxation.
- **Phase gate:** one final declarative `nixos-rebuild switch` with the found exemption set captured in `configuration.nix`, plus the Windows-side curl/browser check (D-13), before the spike is marked VALIDATED.

### Wave 0 Gaps
- [ ] `.planning/spikes/012-*/run.sh` — the driver script itself, descended from spike 010's pattern (per `CONVENTIONS.md`); does not exist yet.
- [ ] A first-command sysctl check (`sysctl kernel.unprivileged_userns_clone`) baked into the driver before any coolwsd-specific steps, per Open Question 2.
- [ ] No framework install needed — everything used (`curl`, `journalctl`, `systemctl`, `nixos-rebuild`) ships in the substrate/Windows box already.

## Sources

### Primary (HIGH confidence)
- `raw.githubusercontent.com/NixOS/nixpkgs/master/nixos/modules/services/web-apps/collabora-online.nix` — fetched directly 2026-08-04; the actual NixOS module source (options, default serviceConfig, settings-merge mechanism)
- `raw.githubusercontent.com/NixOS/nixpkgs/master/pkgs/by-name/co/collabora-online/package.nix` — fetched directly 2026-08-04; confirms `--disable-setcap` build flag and the `fetchFromGitHub` pin to `cp-25.04.9-4`
- `raw.githubusercontent.com/CollaboraOnline/online/cp-25.04.9-4/kit/Kit.cpp` — fetched directly 2026-08-04; the exact upstream source at the version nixpkgs 25.04.9-4 builds — confirms the unshare/chroot/drop-capabilities jailing logic and the silent-degrade log line
- `raw.githubusercontent.com/CollaboraOnline/online/cp-25.04.9-4/coolwsd.xml.in` — fetched directly 2026-08-04; confirms `net.listen` default `"any"`, accepts `"loopback"`
- `raw.githubusercontent.com/microsoft/WSL2-Linux-Kernel` `Microsoft/config-wsl` (default branch) — fetched directly 2026-08-04; confirms `CONFIG_USER_NS=y`, `CONFIG_NAMESPACES=y`, `CONFIG_SECCOMP_FILTER=y`, cgroup support
- `raw.githubusercontent.com/nix-community/NixOS-WSL/main/modules/wsl-conf.nix` and `modules/systemd/default.nix` — fetched directly 2026-08-04; confirms `boot.systemd = true` default and native-systemd-only (no `syschdemd` shim) posture
- `raw.githubusercontent.com/NixOS/nixpkgs/master/nixos/modules/security/systemd-confinement.nix` — fetched directly 2026-08-04; confirms `confinement.enable`'s bind-mount-into-chroot mechanism as an alternative worth trying

### Secondary (MEDIUM confidence)
- `manpages.debian.org/unstable/systemd/systemd.exec.5.en.html` — WebFetch summary 2026-08-04 (the canonical `freedesktop.org` man page returned HTTP 403 to this session's WebFetch); confirms `DynamicUser=` implies `ProtectSystem=strict`+`ProtectHome=read-only`+`RemoveIPC=yes`+disconnected `PrivateTmp=`, and that several sandboxing options "require unprivileged user namespaces support... via the `kernel.unprivileged_userns_clone=` sysctl" — MEDIUM because it's a mirror/summary, not the primary doc itself, though the content is standard and stable systemd behavior
- `forum.collaboraonline.com/t/is-security-capabilities-option-going-away/3978` — WebSearch/WebFetch 2026-08-04; corroborates (does not solely establish — cross-verified against `Kit.cpp` directly) the capabilities→mount_namespaces fallback behavior and the specific warning strings

### Tertiary (LOW confidence)
- General WebSearch results on WSL2 systemd/cgroup support (Microsoft devblogs, community posts) — used only for orientation before finding the primary kernel-config source above; not relied on for any specific claim in this document

## Metadata

**Confidence breakdown:**
- Standard stack (what's already packaged/cached): HIGH — directly verified against nixpkgs source
- Architecture (coolwsd's jailing mechanism, WSL2 kernel capability): HIGH on mechanism/capability, MEDIUM on runtime posture (untested this session, by design — that's the spike's job)
- Pitfalls: MEDIUM-HIGH — grounded in primary-source reading of `Kit.cpp`, not speculation, but the *interaction* with systemd's specific directive stack is inherently something only the spike's own execution can confirm
- §7 baseline directive list: MEDIUM — reconstructed from scattered source-doc mentions, not a single verbatim checklist; flagged explicitly in Assumptions Log

**Research date:** 2026-08-04
**Valid until:** ~14 days (fast-moving: nixpkgs `master` module code, WSL2 kernel config, and the exact NixOS-WSL image in use can all drift; re-verify option names against the `nixos` MCP before planning locks them, per CLAUDE.md)
