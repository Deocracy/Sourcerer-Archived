# Phase 8: Spike K — Nix-Native Substrate Service - Discussion Log

> **Audit trail only.** Do not use as input to planning, research, or execution agents.
> Decisions are captured in CONTEXT.md — this log preserves the alternatives considered.

**Date:** 2026-08-04
**Phase:** 8-Spike K — Nix-Native Substrate Service
**Areas discussed:** Build host, Hardening depth, Image/service set, Proof-of-reachability bar, OCI policy (emergent), Phase 8 fate (emergent), Cross-phase doc surgery

---

## How this discussion went sideways (and why that was right)

The four gray areas originally presented all assumed the phase's OCI premise. The user
interrupted the first question with *"why do we need docker stuff?"* — which turned out to be
the load-bearing question. Explaining that `dockerTools` is a pure Nix fetcher with no daemon
led the user to propose the actual policy: **if it's not in nixpkgs, we don't use it, or we
make it a nixpkgs** — with an explicit invitation to push back if that was wrong.

Pushing back honestly required checking nixpkgs rather than asserting from memory. That check
found `collabora-online` already packaged, moduled, and cached — which invalidated the phase's
premise outright. Three of the four original gray areas dissolved or transformed as a result.

---

## OCI policy (emergent — replaced the original framing)

Raised by the user, not pre-identified. Discussion sequence: efficiency → RAM sharing →
commercial multi-tenant scale.

| Option | Description | Selected |
|--------|-------------|----------|
| Native rule, internal escape hatch | Community catalog Nix-native by construction; OCI permitted only as a time-boxed first-party escape hatch (Phase 13's uv2nix valve), tracked as debt | ✓ |
| Nix-native only, no exceptions | No OCI path anywhere in v1; simplest compiler; Phase 13 accepts uv2nix risk with no exit | |
| Keep the plan as written | OCI stays the community-app path; Spike K runs as scoped | |

**User's choice:** Native rule with a bounded internal escape hatch.

**Notes:** The argument was assembled over several turns, and two of Claude's initial
supporting claims had to be **withdrawn** as the framing sharpened:

- *Permissions* — the user's original hypothesis was that native packages make permissions
  easier to declare. **Corrected:** since the Podman drop, native and OCI compile to the same
  systemd unit; permissions are identical either way. This was the one premise the user was
  wrong about, and they had explicitly asked to be stopped if so.
- *Disk footprint* — Claude initially argued this as a win. **Withdrawn** once the user
  reframed to commercial scale: an image's cost is per distinct image, not per tenant, and
  amortizes to nothing across 10k tenants. It was a desktop-scale argument.
- *RAM* — Claude initially ranked this the *smallest* win. **Promoted** under the user's
  framing: with per-tenant microVMs there is no cross-guest page cache by default, so the cost
  stops amortizing and becomes per-active-tenant. The fix (virtio-fs + DAX over one read-only
  host store) requires identical store paths, which is precisely what OCI rootfs derivations
  destroy and Nix guarantees.
- *Runtime speed* — established as **zero difference** and explicitly struck from the
  rationale, so nobody later claims a performance benefit.

Final ranked rationale of record: **provenance > store page-sharing at scale > (nothing else)**.

---

## Phase 8 fate (emergent)

| Option | Description | Selected |
|--------|-------------|----------|
| Repurpose as a Nix-native substrate spike | Keep the slot, swap the question: `services.collabora-online` in the substrate, UI on 127.0.0.1 from Windows. Criteria 2/3 survive; 1/4 dropped | ✓ |
| Delete Phase 8, start at Phase 9 | Flake foundation absorbs a Collabora assertion into its seed nixosTest | |
| Repurpose, and add the pane leg | As repurposed, plus spike 011's multiwebview exe and its open fractional-DPI retest | |

**User's choice:** Repurpose without the pane leg.

**Notes:** The re-import leg is needed regardless, so the slot retains value. Pane leg pushed
to Phase 15, which already owns the pane UAT.

---

## Hardening depth

| Option | Description | Selected |
|--------|-------------|----------|
| Full baseline, relax knob-by-knob | Start from the full §7 baseline, let it fail, relax one directive at a time, record every concession + reason | ✓ |
| Stock module, then measure | Run as nixpkgs ships it, confirm reachability, record `systemd-analyze security` as a baseline score | |
| Minimum to boot | Prove it runs and is reachable; nothing else | |

**User's choice:** Full baseline, relax knob-by-knob.

**Notes:** This is what preserves the spike's value after the rescope. The original success
criterion 4 (record image-shim findings for Phase 15) was cut with the OCI framing; the
knob-by-knob record replaces it, and additionally pre-pays Phase 19's documented exemption set.
The kill-question became: does a *self-sandboxing* service (coolwsd forks jailed LibreOffice
kernels via its own chroot + namespaces) survive the baseline under WSL2's non-standard kernel?

---

## Image/service set

| Option | Description | Selected |
|--------|-------------|----------|
| Collabora only | The hard case — self-sandboxing, capability-hungry, named in Phase 15's UAT. Keeps to ~the original budget | ✓ |
| Collabora + Jupyter | Add a contrasting well-behaved service to show the pattern generalizes; ~half a day more | |

**User's choice:** Collabora only.

**Notes:** Originally framed as "is Collabora pathological or representative?" — a concern that
largely evaporated once it turned out to be a first-class NixOS module rather than a
third-party image. Jupyter deferred to Phase 15.

---

## Build host

*Asked, then withdrawn before the user answered — superseded by evidence.*

| Option | Description | Selected |
|--------|-------------|----------|
| Inside SourcererSpike | Build natively in the WSL distro; one host, no transport plumbing | ✓ (by discretion) |
| NixOS dev host + `nix copy` | Build on `legion`, copy the closure over the LAN; closer to the product model | |
| Dev-host dry-run, then WSL for real | Iterate on the dev host, authoritative build in the substrate | |

**Resolution:** The cache lookup showed `collabora-online` is already built and cached
(13.8 MB), so nothing compiles anywhere — the question dissolved. Taken as Claude's discretion:
build in-substrate. The user did not object.

---

## Proof-of-reachability bar

*Resolved implicitly by the Phase 8 fate answer rather than asked directly.*

Settled at curl + browser on 127.0.0.1 from the Windows host. The multiwebview pane leg — the
richer option, which would also have closed spike 011's open fractional-DPI retest — was
declined by selecting the repurpose option without it.

---

## Cross-phase doc surgery

| Option | Description | Selected |
|--------|-------------|----------|
| Right after CONTEXT.md | Rewrite ROADMAP Phase 8/15/16, REQUIREMENTS SPIKE-01/APP-01/STORE-01-02, PROJECT.md Key Decisions, mark CONTAINER-PLATFORM.md §5 superseded | ✓ |
| Later, separately | Write CONTEXT.md now, docs later | |
| Just Phase 8 for now | Fix only Phase 8 + SPIKE-01; leave 15/16 until those phases come up | |

**User's choice:** Immediately after CONTEXT.md.

**Notes:** Motivating risk stated at decision time — stale OCI language left in REQUIREMENTS.md
and ROADMAP.md would be read as *locked* by the Phase 15 planner six phases from now, long
after the reasoning is out of context.

---

## Claude's Discretion

- Build location (in-substrate) — dissolved by the cache finding; user did not object.
- Exact §7 directive list and the relaxation order.
- Whether findings get wrapped into a `spike-findings-*` skill (default: yes) or stay as the
  spike README.
- Whether to leave the `SourcererSpike` distro registered afterwards (default: yes, matching
  spike 010).
- Timebox/abort handling if the hardening ladder proves bottomless (default: record concessions
  reached and stop; a partial exemption set is still usable evidence).

## Deferred Ideas

- `services.jupyter` as a second contrasting service → Phase 15
- Multiwebview pane leg + spike 011's fractional-DPI retest → Phase 15
- The original engine-less OCI unit spike → only if Phase 13 invokes its uv2nix escape hatch
- Packaging workstream for components not yet in nixpkgs → revisit when one actually fails the rule
