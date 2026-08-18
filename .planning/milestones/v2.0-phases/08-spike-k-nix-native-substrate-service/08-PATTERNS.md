# Phase 8: Spike K — Nix-Native Substrate Service - Pattern Map

**Mapped:** 2026-08-04
**Files analyzed:** 5 (all under `.planning/spikes/`; **zero** `src/`/`src-tauri/` files —
CONTEXT.md `<code_context>` is explicit that this phase touches no application tree)
**Analogs found:** 4 / 5 (the 5th — a `configuration.nix` fragment — has no *repo* analog by
design; it lives inside the WSL guest, not this git tree)

## Scope Note

This phase is a systemd/Nix configuration spike, not application code. There are no
controllers/components/services/models to classify in the GSD sense. "Role" below is
reinterpreted for spike artifacts: **driver-script**, **findings-doc**, **index-entry**,
**convention-doc**. Do not search for React/Rust/Tauri analogs — none exist for this phase's
outputs and none should be reported.

## File Classification

| New/Modified File | Role | Data Flow | Closest Analog | Match Quality |
|---|---|---|---|---|
| `.planning/spikes/012-nix-native-substrate-service/run.sh` | driver-script | batch (sequential host→WSL commands, timed, tee-logged) | `.planning/spikes/010-nixos-wsl-substrate/run.sh` | exact, but see **New Pattern Required** below — 010's fixed-leg shape does not cover this phase's indeterminate hardening ladder |
| `.planning/spikes/012-nix-native-substrate-service/run.log` | generated artifact (not authored) | batch output capture | `.planning/spikes/010-nixos-wsl-substrate/run.log` | exact (mechanical `tee` output, nothing to pattern-map — it's what `run.sh` produces) |
| `.planning/spikes/012-nix-native-substrate-service/README.md` | findings-doc | transform (raw run.log → structured findings) | `.planning/spikes/010-nixos-wsl-substrate/README.md` (primary) + `.planning/spikes/011-tauri-multiwebview/README.md` (confirms shape) | exact |
| `.planning/spikes/MANIFEST.md` | index-entry (MODIFY — append row) | CRUD (append one row) | itself — `.planning/spikes/MANIFEST.md` (existing rows 001–011) | exact |
| `.planning/spikes/CONVENTIONS.md` | convention-doc (MODIFY — append, only if new system-level landmines found) | CRUD (append to existing "System-level spikes" section) | itself — `.planning/spikes/CONVENTIONS.md` lines 42-52 | exact |

## No Analog Found

| File | Role | Data Flow | Reason |
|---|---|---|---|
| `configuration.nix` fragment (captured as a README code excerpt, not a repo file) | config | transform (declarative NixOS module config) | Per CONTEXT.md/phase guidance this is captured **inside the README as an excerpt**, never written into this repo — the real file lives at `/etc/nixos/configuration.nix` inside the `SourcererSpike` WSL guest. No repo analog exists or should exist. Use RESEARCH.md's "Architecture Patterns → Recommended iteration loop" declarative-capture snippet (already-verified against nixpkgs source) as the content template, not a repo file. |

No `src/`, `src-tauri/`, applet, or shell-chrome files are in scope — confirmed against
CONTEXT.md `<code_context>` ("Established Patterns: Spikes live entirely under
`.planning/spikes/NNN-name/`... No `src/` or `src-tauri/` file should change in this phase")
and the phase-specific guidance. Do not invent analogs for a tree this phase never touches.

---

## Pattern Assignments

### `.planning/spikes/012-nix-native-substrate-service/run.sh` (driver-script, batch)

**Analog:** `.planning/spikes/010-nixos-wsl-substrate/run.sh` (34 lines, full file — small
enough for one read, extracted whole)

**Header + helpers pattern** (lines 1-13):
```bash
#!/usr/bin/env bash
# Spike 010 driver — every guest interaction goes through wsl.exe exactly as the
# Sourcerer shell would drive it. Logs to run.log with per-step timestamps.
set -uo pipefail
cd "$(dirname "$0")"
LOG="run.log"
DISTRO="SourcererSpike"
STORE='D:\WSL\SourcererSpike'
IMG="$(cygpath -w "$(pwd)/dist/nixos.wsl")"
export WSL_UTF8=1

step() { echo "" | tee -a "$LOG"; echo "=== [$(date +%H:%M:%S)] $*" | tee -a "$LOG"; }
run()  { echo "\$ $*" | tee -a "$LOG"; "$@" 2>&1 | tee -a "$LOG"; return "${PIPESTATUS[0]}"; }
```
Copy this block near-verbatim for 012: same `set -uo pipefail`, same `cd "$(dirname "$0")"`,
same `WSL_UTF8=1` export, same `step()`/`run()` helper pair. `DISTRO="SourcererSpike"` is
reused as-is (per CONTEXT.md's default: leave the distro registered/warm from spike 010 —
012 should try driving it directly first and only fall back to a re-import leg, mirrored
from steps 0-2 below, if `wsl.exe -l -v` shows it gone).

**Fixed-leg + timing pattern** (lines 15-28, representative sample):
```bash
echo "SPIKE 010 RUN — $(date -Iseconds)" | tee "$LOG"

step "0. preexisting state"
run wsl.exe -l -v

step "1. IMPORT (timed)"
T0=$SECONDS
run wsl.exe --import "$DISTRO" "$STORE" "$IMG" --version 2
echo "import_seconds=$((SECONDS-T0))" | tee -a "$LOG"
```
Every timed leg follows `T0=$SECONDS ... run <cmd> ... echo "<name>_seconds=$((SECONDS-T0))"`.
Copy this exactly for 012's timed legs (re-import if needed, `nixos-rebuild switch --dry-run`,
the final declarative `nixos-rebuild switch`).

**Path-wrapping + root-user pattern** (lines 30-39):
```bash
step "3. IDENTITY + default config (logged for the trail)"
run wsl.exe -d "$DISTRO" -- sh -c 'head -2 /etc/os-release; echo "user=$(whoami)"'
run wsl.exe -d "$DISTRO" -u root -- cat /etc/nixos/configuration.nix

step "5. CONFIG CHANGE: add pkgs.hello (sed-insert before final brace)"
run wsl.exe -d "$DISTRO" -u root -- sh -c \
  "sed -i 's|^}\$|  environment.systemPackages = [ pkgs.hello ];\n}|' /etc/nixos/configuration.nix && tail -5 /etc/nixos/configuration.nix"
```
This is the CONVENTIONS.md `sh -c '...'`-wrapping rule in action: every absolute Linux path
(`/etc/nixos/configuration.nix`, `/etc/systemd/system/coolwsd.service.d/`) MUST go inside a
`sh -c '...'` string when invoked via `wsl.exe` from Git Bash — MSYS rewrites bare
`/etc/...`-style args into a Windows path otherwise. 012's systemd drop-in writes (heredoc
into `/etc/systemd/system/coolwsd.service.d/override.conf`, per RESEARCH.md's iteration loop)
must follow this exact wrapping discipline. `-u root` is required for both config edits and
`systemctl`/`nixos-rebuild` — non-root steps (e.g. `curl` reachability checks) omit it.

**Sign-off pattern** (line 70):
```bash
step "DONE — distro left registered for follow-up legs; cleanup: wsl --unregister $DISTRO"
```
012 should end the same way per CONTEXT.md's default (leave `SourcererSpike` registered for
Phase 10), stating the cleanup command without running it.

**New Pattern Required — indeterminate iteration ladder (no existing spike analog):**
010's `run.sh` is a **fixed sequence of 12 named legs** — every step is hardcoded because the
substrate lifecycle (import → boot → config → rollback → terminate) has a known, fixed shape.
This phase's hardening ladder (D-10: apply full §7 baseline, then relax knob-by-knob until it
passes, recording every relaxation) does **not** have a known rung count up front — it is
exploratory. Skimming 001-005 and 011 confirms this: **no existing spike script in this repo
does looped/indeterminate iteration** — 001-005 are Node/JS harness spikes with no `run.sh`
at all, and 011 is a single `cargo run` (no iteration loop either). 010 is the only driver
script in the repo, and its shape is "N fixed named steps," not "loop until condition."

**Recommendation for the planner:** don't force the ladder into 010's fixed-step mold. Add
one reusable **rung function** to 012's `run.sh`, built directly from RESEARCH.md's already-
verified "Recommended iteration loop" snippet (Architecture Patterns section), called once per
directive-relaxation with a short label and drop-in content as arguments — e.g.
`rung "full-baseline" "$(full_baseline_dropin)"`, `rung "relax-restrictnamespaces" "$(...)"`.
Each call still routes through the existing `step()`/`run()` helpers (so `run.log`'s tee/
`PIPESTATUS` discipline is unchanged), it's only the *fixed-vs-looped* shape that's new:

```bash
# New helper, pattern-derived from RESEARCH.md's iteration loop + 010's step()/run():
rung() {
  local name="$1" dropin="$2"
  step "RUNG: $name"
  run wsl.exe -d "$DISTRO" -u root -- sh -c "
    mkdir -p /etc/systemd/system/coolwsd.service.d
    cat > /etc/systemd/system/coolwsd.service.d/override.conf <<'EOF'
$dropin
EOF
    systemctl daemon-reload && systemctl restart coolwsd
  "
  sleep 2
  run wsl.exe -d "$DISTRO" -u root -- journalctl -u coolwsd -n 40 --no-pager
}
```
The number of `rung` calls in the finished script is whatever the ladder actually needs — the
planner should not pre-specify an exact count; VALIDATION.md's own gate is "rung count ==
Investigation Trail row count," not a fixed number.

---

### `.planning/spikes/012-nix-native-substrate-service/README.md` (findings-doc, transform)

**Analog:** `.planning/spikes/010-nixos-wsl-substrate/README.md` (primary shape) +
`.planning/spikes/011-tauri-multiwebview/README.md` (confirms the shape is the current,
repo-wide convention, not a 010-specific one-off)

**Frontmatter pattern** (010 lines 1-9, identical field set in 011 lines 1-9):
```yaml
---
spike: 010
name: nixos-wsl-substrate
type: standard
validates: "Given the prebuilt NixOS-WSL 2605.7.2 image, when imported as a private custom-named WSL distro and driven entirely via wsl.exe, then it boots, applies a config change with nixos-rebuild switch, and reverts it with --rollback — no terminal interaction inside the guest"
verdict: VALIDATED
related: [011-tauri-multiwebview]
tags: [nixos-wsl, substrate, wsl, updater, rollback, container-platform, spike-A]
---
```
For 012: `spike: 012`, `name: nix-native-substrate-service`, `type: standard`, `validates:`
should be a single Given/When/Then sentence mirroring D-09's kill-question, `verdict:` starts
undetermined and is filled at the end (`VALIDATED` / `PARTIALLY VALIDATED` per D-10's timebox
clause — a partial exemption set is a valid, recordable outcome, not a failed spike), `related:
[010-nixos-wsl-substrate]` (and `011-tauri-multiwebview` if the multiwebview pane leg's
deferred status is worth cross-referencing), `tags:` should include `collabora-online`,
`systemd-hardening`, `nixpkgs`, `container-platform`.

**Body heading structure** (010, section headers only — same set in 011):
```
# Spike NNN: <Title> (Container Platform "spike <letter>")
## What This Validates
## Research
## How to Run
## What to Expect
## Investigation Trail
## Results
```
Copy this exact heading set and order. `## Results` must open with a bolded verdict line
(`**VERDICT: VALIDATED**` — 010 line 94; 011 line 73), followed by the evidence table.

**Investigation Trail entry pattern — the "Attempt N failed, hypothesis X → Attempt N+1
confirmed" form** (010 lines 60-79, representative two-attempt excerpt):
```markdown
- **Attempt 1 (18:08):** every step failed cascading from step 1 `Wsl/ERROR_PATH_NOT_FOUND`.
  Root cause: `wsl --import <name> D:\WSL\SourcererSpike <img>` does **NOT create the parent
  directory** — `D:\WSL` didn't exist. Product lesson for plan P2: the shell installer must
  create the storage directory before import; the raw WSL error is opaque enough that a
  non-technical user could never self-diagnose it. Also confirmed: spaces in the image path
  (`D:\Vibe Coding\...`) were NOT the problem (error persisted path-side, arg passed intact).
- **Attempt 2 (18:09):** with the parent dir created, **import succeeded** (36s, 2.7 GB
  ext4.vhdx) but every boot failed `Wsl/Service/E_UNEXPECTED "Catastrophic failure"` in ~3s.
  Hypothesis: WSL 2.3.26 (late-2024 vintage) too old for the 2605.7.2 image.
```
The load-bearing structure per bullet: **timestamp** → **what failed, verbatim error string**
→ **root cause** → **product lesson, tied to a specific downstream plan/phase**. 012's trail
must follow this exactly for every ladder rung: which directive was relaxed, the exact
`journalctl`/seccomp-audit line that forced it (per RESEARCH.md's Anti-Patterns: "'it didn't
work' is not a reason"), and — critically, per RESEARCH.md Pitfall 2 — whether the rung's
"pass" was cross-checked against the `coolwsd` jail-degradation warning, not just a green
`curl`. VALIDATION.md's Manual-Only Verifications row is explicit: "no row may say only
'didn't work.'"

**Results table pattern** (010 lines 96-106, full table copied as the shape to reuse):
```markdown
| Leg | Result | Time |
|---|---|---|
| Import (private name, D: storage) | ✓ | 41 s |
| First boot | ✓ (`boot-ok`, user `nixos`) | 4 s |
| Config change applied externally (`nixos-rebuild switch`, +pkgs.hello) | ✓ rc=0 (103 MiB from cache.nixos.org) | 109 s |
```
For 012, the ladder's version of this table is per-rung: `| Rung | Directive relaxed | Signature observed | Concession | Result |` — RESEARCH.md's Validation Architecture and VALIDATION.md's Per-Task Verification Map already specify these exact columns' content; this Results-table *shape* (from 010) is what carries them.

**Provenance callout pattern** (010 lines 81-90, "Source image provenance" — a named
subsection outside the fixed heading set, used when there's a specific artifact/hash worth
isolating):
```markdown
## Source image provenance

NixOS-WSL 2605.7.2 "Yearning Yarara" — `github.com/nix-community/NixOS-WSL` releases,
asset `nixos.wsl`, sha256
`e7180ad555fdcb8e1e057e2ef056de467603a5e502ff8531053738371be3f6b9`.
```
012 has an analogous callout candidate: the exact nixpkgs channel/commit `collabora-online`
25.04.9-4 resolved from (RESEARCH.md flags this as unverified against the actual pinned
channel — Assumption A5). Add a short "## Package provenance" subsection recording the
resolved store path (`/nix/store/82181wy8scpzh0fis39gjjjnzk5462c9-collabora-online-25.04.9-4`
per D-07, confirmed or corrected at execution time) the same way 010 isolates the image hash.

**Caveats subsection pattern** (011 lines 89-97, used when a spike's verdict has open,
non-blocking edges):
```markdown
**Caveats (open, not blocking):**
- `devicePixelRatio` was 1 on this display — the fractional-DPI alignment case (125%/150%
  scaling) is UNTESTED; retest on a scaled display before P4 relies on pixel-exact rects.
```
012 should use this exact form for RESEARCH.md's Open Questions that remain open at spike end
(e.g. whether `coolwsd` supports systemd socket activation, if the socket-activation rung
wasn't reached or wasn't conclusive) — one bullet per caveat, each naming the downstream
consumer it's deferred to (Phase 15/19), matching 011's "retest before P4 relies on..." form.

---

### `.planning/spikes/MANIFEST.md` (index-entry, CRUD append)

**Analog:** itself — existing table, `.planning/spikes/MANIFEST.md` lines 20-32

**Exact row format to match** (header + two most recent rows, lines 20-21, 31-32):
```markdown
| # | Name | Type | Validates | Verdict | Tags |
|---|------|------|-----------|---------|------|
| 010 | nixos-wsl-substrate | standard | Given the prebuilt NixOS-WSL 2605.7.2 image, when imported as a private custom-named WSL distro and driven entirely via wsl.exe, then it boots, applies a config change with nixos-rebuild switch, and reverts with --rollback (Container Platform plan P0 "spike A-lite"; Podman/Collabora/pane legs descoped by user — guess-with-fallbacks) | VALIDATED | nixos-wsl, substrate, updater, rollback, container-platform |
| 011 | tauri-multiwebview | standard | Given a Tauri 2 window with the `unstable` feature, when a full-size shell webview + two external-origin child webviews are composed Sourcerer-style and one child's bounds are animated, then children render above the shell at DOM-aligned coords without jank and the shell stays healthy (Container Platform plan P0 "spike E") | VALIDATED | tauri, multiwebview, panes, webview2, container-platform |
```
New row: `# = 012`, `Name = nix-native-substrate-service`, `Type = standard`, `Validates =`
a Given/When/Then sentence built from D-09's kill-question (mirrors the frontmatter
`validates:` field verbatim — 010/011 keep these two in sync), `Verdict =` filled at spike end
(`VALIDATED` / a partial-outcome value per D-10's timebox clause — check whether the manifest
vocabulary needs a new status word or whether `VALIDATED` covers "ladder completed with a
recorded exemption set" even if not every directive relaxed to zero), `Tags =`
`collabora-online, systemd-hardening, nixpkgs, container-platform` at minimum.

**Note:** MANIFEST.md's `## Requirements` section (lines 7-16) is a project-wide list, not
per-spike — no edit needed there for 012 unless this spike surfaces a genuinely new
cross-cutting requirement (unlikely; D-11 scopes this spike to one already-covered surface).

---

### `.planning/spikes/CONVENTIONS.md` (convention-doc, CRUD append — conditional)

**Analog:** itself — `.planning/spikes/CONVENTIONS.md` lines 42-52, the existing
"System-level spikes (Windows host, established in 010/011)" section

**Exact section to extend, verbatim (binding, do not restate wrong):**
```markdown
## System-level spikes (Windows host, established in 010/011)

- Drive WSL only via `wsl.exe` with `WSL_UTF8=1` exported (raw output is UTF-16 in Git Bash).
- Wrap absolute Linux paths in `sh -c '...'` when invoking through Git Bash — MSYS path
  conversion silently rewrites direct `/etc/...`-style args into `C:/Program Files/Git/...`.
- Never trust `cmd | tail`'s exit code — check `PIPESTATUS[0]`.
- Timed, tee-logged driver scripts (`run.sh` → `run.log`) are the forensic layer for
  host-mutating spikes; log every attempt, keep failed-attempt logs in the README trail.
- Tauri scratch apps: own crate inside the spike dir (never the main app tree), `icons/icon.ico`
  copied from src-tauri (tauri-build hard-requires it on Windows even with bundling off),
  detached launch + PowerShell `CopyFromScreen` screenshots as visual evidence.
```
These five bullets are **binding for this phase's artifact form** per CONTEXT.md's own
canonical_refs citation of this file. 012's `run.sh`/README MUST NOT restate or contradict
them. If the ladder surfaces a genuinely new system-level landmine (e.g. a WSL2-specific
sysctl gotcha, a `nixos-rebuild` non-interactive quirk not covered by the existing bullets),
append **one new bullet** to this same section — do not create a new heading, do not touch
the unrelated `## Stack`/`## Structure`/`## Patterns`/`## Tools & Libraries` sections above it
(those govern the harness-selection spikes 001-005 and are out of scope for this phase).

---

## Shared Patterns

### WSL driving discipline
**Source:** `.planning/spikes/CONVENTIONS.md` lines 44-47 (binding) + `010/run.sh` lines 4,10,13
**Apply to:** `012-.../run.sh` in full
```bash
set -uo pipefail
export WSL_UTF8=1
run()  { echo "\$ $*" | tee -a "$LOG"; "$@" 2>&1 | tee -a "$LOG"; return "${PIPESTATUS[0]}"; }
```
Every `wsl.exe` invocation in 012 goes through this `run()` wrapper; every absolute guest
path goes inside `sh -c '...'`; every exit-code check reads `PIPESTATUS[0]`, never a bare `$?`
after a pipe.

### Forensic-first Investigation Trail
**Source:** `.planning/spikes/CONVENTIONS.md` line 49 + `010/README.md` Investigation Trail
(lines 58-79) + `011/README.md` Investigation Trail (lines 60-69)
**Apply to:** `012-.../README.md`
Every attempt — pass or fail — gets a dated/timestamped bullet: what happened (verbatim
error/log string), root cause, product/spike lesson. Failed attempts are never deleted or
summarized away; they are the evidence. For this phase specifically, "pass" additionally
requires the jail-status cross-check (RESEARCH.md Pitfall 2) — a rung is not loggable as PASS
on `curl` success alone.

### Spike index consistency
**Source:** `.planning/spikes/MANIFEST.md` table format
**Apply to:** the `012` row in `MANIFEST.md` + the `validates:`/`verdict:`/`tags:` frontmatter
in `012-.../README.md`
Keep the `validates:` sentence and the manifest's `Validates` column text in sync (010/011
both do this — same sentence, not paraphrased twice) and keep `verdict:`/`Verdict` in sync
between the two files at spike close.

---

## Metadata

**Analog search scope:** `.planning/spikes/` (all 11 existing spike directories +
`MANIFEST.md` + `CONVENTIONS.md` + `WRAP-UP-SUMMARY.md`). No search of `src/`/`src-tauri/`
per phase scope (this phase creates no application code).
**Files scanned:** `010-nixos-wsl-substrate/run.sh` (full), `010-nixos-wsl-substrate/README.md`
(full), `011-tauri-multiwebview/README.md` (full), `CONVENTIONS.md` (full),
`MANIFEST.md` (full), `001-pi-headless-embed/README.md` (frontmatter skim, confirms shape
consistency across all 11 spikes, no divergence worth extracting).
**Pattern extraction date:** 2026-08-04
