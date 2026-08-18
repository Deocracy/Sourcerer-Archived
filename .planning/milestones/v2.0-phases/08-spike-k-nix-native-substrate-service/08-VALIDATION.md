---
phase: 8
slug: spike-k-nix-native-substrate-service
status: planned
nyquist_compliant: true
wave_0_complete: false
created: 2026-08-04
updated: 2026-08-05
---

# Phase 8 — Validation Strategy

> Per-phase validation contract for feedback sampling during execution.
> Derived from `08-RESEARCH.md` § Validation Architecture.
> Task IDs assigned 2026-08-05 during `/gsd-plan-phase 8`.

**Phase character:** this is a systemd/Nix configuration spike, not application code. There is
no unit-test framework and none should be installed. The "test framework" is the forensic
driver-script + log-grep pattern that `.planning/spikes/CONVENTIONS.md` makes binding
(`run.sh` → tee'd `run.log` + README Investigation Trail).

---

## Test Infrastructure

| Property | Value |
|----------|-------|
| **Framework** | None — shell-driven forensic verification per `.planning/spikes/CONVENTIONS.md` |
| **Config file** | `/etc/nixos/configuration.nix` inside the `SourcererSpike` distro (declarative end-state); systemd drop-ins during ladder exploration |
| **Quick run command** | `wsl.exe -d SourcererSpike -u root -- sh -c 'systemctl daemon-reload && systemctl restart coolwsd && sleep 4 && journalctl -u coolwsd -u coolwsd-systemplate-setup -n 40 --no-pager'` |
| **Full suite command** | `wsl.exe -d SourcererSpike -u root -- nixos-rebuild switch` (declarative proof-pass, once per confirmed directive set) |
| **Estimated runtime** | ~5s per drop-in rung · ~110s per `nixos-rebuild switch` (spike 010 measured 109s) |

**Why drop-ins for the ladder:** at ~110s per rebuild, a 10+ rung hardening ladder costs
~20 min of pure rebuild wall-clock. Drop-ins collapse each rung to seconds. The final
declarative pass exists to prove the exemption set survives the real module-config form
Phase 15's compiler will emit.

---

## Sampling Rate

- **Per ladder rung (drop-in iteration):** run the Quick run command — restart + `journalctl`
  grep. Max feedback latency ~5s (4s settle + probe).
- **Per confirmed exemption-set change:** re-run the discovery-endpoint curl to confirm the
  latest relaxation did not silently break reachability.
- **Per rung, mandatory:** the jail-degradation grep (see Manual-Only Verifications). A green
  restart is **not** a pass — `coolwsd` serves jail-less after logging a warning.
- **Phase gate:** one final `nixos-rebuild switch` with the exemption set expressed
  declaratively, plus the Windows-side curl + browser check (D-13), before the spike is
  marked VALIDATED.
- **Max feedback latency:** 5 seconds (drop-in rung) / 110 seconds (declarative pass).

---

## Per-Task Verification Map

| Task ID | Plan | Wave | Requirement | Threat Ref | Secure Behavior | Test Type | Automated Command | File Exists | Status |
|---------|------|------|-------------|------------|-----------------|-----------|-------------------|-------------|--------|
| 08-01-01 | 01 | 1 | SPIKE-01 | — | Driver script exists with binding CONVENTIONS.md discipline (`WSL_UTF8=1`, `sh -c` path wrapping, `PIPESTATUS[0]`) | source | `bash -n run.sh` + grep assertions on helpers, WSL floor, image sha256, storage-parent pre-create | ✅ W0 (this task creates it) | ⬜ pending |
| 08-01-03 | 01 | 1 | SPIKE-01 | — | Substrate distro registered and warm; WSL version ≥ 2.7.11 | smoke | `wsl.exe -l -v` + `wsl.exe --version` → `wsl_version=` / `distro_state=` in `run.log` | ✅ via `run.sh preflight` | ⬜ pending |
| 08-01-02 | 01 | 1 | SPIKE-01 | — | Unprivileged userns available (Open Q2) — measured functionally, not inferred from a sysctl | smoke | `guest 'unshare --user --map-root-user -- id -u'` → `USERNS_OK=` + `USERNS_EVIDENCE=` | ✅ via `run.sh preflight` | ⬜ pending |
| 08-02-01 | 02 | 2 | SPIKE-01 | — | Collabora substitutes from cache; the heavy closure is fetched, not built | smoke | `nixos-rebuild switch --dry-run` — assert the "will be built" list contains no `collabora-online-[0-9]`, `libreoffice`, `poco`, `boost` → `SUBSTITUTED_OK=yes` | ✅ via `run.sh stock` | ⬜ pending |
| 08-02-02 | 02 | 2 | SPIKE-01 | — | Service active under stock (unhardened) module, and the stock jail state recorded as the ladder's control | smoke + log-grep | `systemctl is-active coolwsd` → `STOCK_ACTIVE=active`; `jail_state` → `STOCK_JAIL=`; `reach_state` → `STOCK_REACH=ok` | ✅ via `run.sh stock` | ⬜ pending |
| 08-02-03 | 02 | 2 | SPIKE-01 | — | The build under test is identified — store path, version, closure size, channel (closes RESEARCH A5) | artifact | `systemctl show -p ExecStart coolwsd` + `nix path-info -S` → `collabora_store_path=` / `d07_store_path_match=` | ✅ via `run.sh stock` | ⬜ pending |
| 08-03-01 | 03 | 3 | SPIKE-01 | — | Full §7 baseline applied FIRST and its failure captured verbatim | log-grep | `journalctl -u coolwsd -u coolwsd-systemplate-setup --since <rung>` grepped for `SIGSYS`, `Seccomp`, `Operation not permitted`, `Failed at step`, `Permission denied` | ✅ via `run.sh ladder` | ⬜ pending |
| 08-03-02 | 03 | 3 | SPIKE-01 | — | Each rung's relaxation recorded with directive + observed signature + concession + reason; ladder is bounded and resumable with a named stop condition | artifact | `awk -F'\t' 'NR>1 && ($5=="" \|\| $5=="failed")' rungs.tsv` prints nothing; `grep -c 'LADDER_STOP=' run.log` == 1 | ✅ `rungs.tsv` (this task creates it) | ⬜ pending |
| 08-03-03 | 03 | 3 | SPIKE-01 | — | Jail is real, not silently degraded — no rung passes on reachability alone | log-grep | `awk -F'\t' 'NR>1 && $8=="PASS" && $6=="degraded"' rungs.tsv` prints nothing | ✅ via `run.sh ladder` | ⬜ pending |
| 08-04-01 | 04 | 4 | SPIKE-01 | — | Exemption set survives a real declarative rebuild, with no stale drop-in shadowing it | full | `rm -rf coolwsd.service.d` → `nixos-rebuild switch` → `systemctl show -p <directives> -p DropInPaths coolwsd` → `DECLARATIVE_MATCH=yes` | ✅ via `run.sh declarative` | ⬜ pending |
| 08-04-02 | 04 | 4 | SPIKE-01 | — | UI reachable on 127.0.0.1 from the Windows host; a failure is attributed to the correct layer | smoke + manual | (Windows, outside WSL) `curl.exe -sf http://127.0.0.1:9980/hosting/discovery` → `WINDOWS_REACH=` + `WINDOWS_REACH_CAUSE=` | ✅ manual + `run.log` | ⬜ pending |
| 08-05-01 | 05 | 5 | SPIKE-01 | — | Exemption set written down as evidence — verbatim declarative block, per-rung trail, per-directive disposition | artifact | Investigation Trail rung-row count ≥ `rungs.tsv` data-row count; no blank/"didn't work" signature or concession cell | ✅ `README.md` (this task creates it) | ⬜ pending |
| 08-05-02 | 05 | 5 | SPIKE-01 | — | Index and downstream docs carry the verdict forward without drift | artifact | MANIFEST row 012 `Validates`/`Verdict` byte-identical to README frontmatter; SPIKE-01 status no longer `Pending` | ✅ `MANIFEST.md` | ⬜ pending |
| 08-05-03 | 05 | 5 | SPIKE-01 | — | Findings legible to a reader with no memory of this session | manual | the four cold-read questions answered from `README.md` alone | ✅ manual | ⬜ pending |

*Status: ⬜ pending · ✅ green · ❌ red · ⚠️ flaky*

---

## Wave 0 Requirements

- [ ] `.planning/spikes/012-nix-native-substrate-service/run.sh` — the driver script, descended
      from `.planning/spikes/010-nixos-wsl-substrate/run.sh`. **Closed by task 08-01-01.**
      Carries the assertions above as shell checks with explicit exit codes. Must apply
      CONVENTIONS.md system rules: `export WSL_UTF8=1`, wrap absolute Linux paths in
      `sh -c '...'`, check `PIPESTATUS[0]` never a piped command's exit code.
- [ ] A pre-flight userns check as the driver's **first** coolwsd-relevant step, per RESEARCH
      Open Question 2. **Closed by task 08-01-02.** Note the planner's correction: RESEARCH names
      `kernel.unprivileged_userns_clone`, which is a Debian/Ubuntu/Arch downstream patch and is
      likely absent on Microsoft's mainline-derived WSL2 kernel. The check therefore probes three
      things — that sysctl, the mainline `user.max_user_namespaces`, and a functional
      `unshare --user --map-root-user` run as the UNPRIVILEGED guest user, which is the only
      definitive answer and the only one the verdict is computed from.
- [ ] No framework install. `curl`, `journalctl`, `systemctl`, `nixos-rebuild`, `systemd-analyze`
      and `unshare` all ship in the substrate / on the Windows box already (`curl` and
      `util-linux` are both in NixOS's `requiredPackages`).

---

## Manual-Only Verifications

| Behavior | Requirement | Why Manual | Test Instructions |
|----------|-------------|------------|-------------------|
| Collabora UI renders in a browser on the Windows host | SPIKE-01 (SC-3) | D-13 sets the bar as "curl + browser"; a rendered editor surface cannot be asserted by curl alone | Open `http://127.0.0.1:9980/` in a Windows browser; confirm the Collabora welcome/discovery surface loads, not a connection error. Record what actually rendered in words — "discovery XML shown as text" is a different observation from "the welcome surface loaded" |
| Exemption set is legible to a reader with no memory of this session | SPIKE-01 (SC-4) | Phase 15 and Phase 19 are the consumers and are many phases downstream; legibility is a human judgement | Read the README Investigation Trail cold: each row must name the directive, the observed failure signature, the concession made, and *why* — no row may say only "didn't work" |
| Jail degradation did not go unnoticed | SPIKE-01 (SC-2) | `coolwsd` logs a warning and keeps serving jail-less; a green curl masks it | For every recorded rung, confirm the trail's log excerpt shows jail state explicitly, not just service-active. Cross-check against the plan-02 `STOCK_JAIL` control — a rung marked degraded means nothing if the stock service was already degraded |

---

## Validation Sign-Off

- [ ] All tasks have an `<automated>` verify command or a Wave 0 dependency on `run.sh`
- [ ] Sampling continuity: no 3 consecutive tasks without an automated verify
- [ ] Wave 0 covers all MISSING references (`run.sh` + userns pre-flight)
- [ ] No watch-mode flags
- [ ] Feedback latency < 5s per ladder rung
- [ ] Every ladder rung produced a trail row (rung count == trail row count)
- [ ] `nyquist_compliant: true` set in frontmatter

**Approval:** pending
