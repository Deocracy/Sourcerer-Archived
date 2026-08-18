---
phase: 09-flake-foundation-assurance-chain
plan: 03
subsystem: infra
tags: [nix, nixos, nixos-wsl, nixostest, flake, substrate, qemu]

# Dependency graph
requires:
  - phase: 09-02
    provides: "flake.nix devShell, nixpkgs nixos-26.05 pin, rust-overlay, nix/lib.nix"
provides:
  - "nix/substrate/core.nix (D-15 single substrate definition: user, Nix settings, nix-ld, D-12 seed service)"
  - "nix/substrate/wsl-variant.nix (real WSL image target adapter)"
  - "nix/substrate/vm-variant.nix (plain-QEMU adapter, what CI boots)"
  - "nix/checks/seed-boot-test.nix (nixosTest wired into nix flake check, D-12)"
  - "flake.nix nixosConfigurations.substrate-wsl / substrate-vm, packages.substrate-system, packages.substrate-image-builder, checks.seed-boot-test"
  - "flake.lock relocked with nixos-wsl input (tag 2605.7.2)"
affects: ["09-04", "09-05", "09-06", "09-07"]

# Tech tracking
tech-stack:
  added: ["nixos-wsl (nix-community, tag 2605.7.2)", "pkgs.testers.runNixOSTest"]
  patterns:
    - "Shared NixOS module + two thin adapter variants (D-15): substrate contents defined exactly once in core.nix"
    - "nixosTest boots the plain-VM variant only; the WSL adapter is never exercised by CI (documented gap, Phase 10's concern)"
    - "Standalone nixosSystem outputs need a real fileSystems/boot.loader.grub.device stanza to satisfy NixOS bootability assertions for eval purposes only — kept as an anonymous flake.nix module, not inside vm-variant.nix, so that file's imports stay exactly [ ./core.nix ]"

key-files:
  created: [nix/substrate/core.nix, nix/substrate/wsl-variant.nix, nix/substrate/vm-variant.nix, nix/checks/seed-boot-test.nix]
  modified: [flake.nix, flake.lock]

key-decisions:
  - "Comments inside core.nix/vm-variant.nix must not literally contain the acceptance-criteria's forbidden substrings (e.g. writing '0.0.0.0' or 'nixos-wsl' even in prose triggers the plan's own grep -q FAIL checks) — reworded explanatory comments to describe the constraint without repeating the literal string"
  - "Standalone nixosConfigurations.substrate-vm needs a real root filesystem + grub device to evaluate config.system.build.toplevel (a plain nixosSystem, unlike a nixosTest node, gets no automatic qemu-vm disk/boot scaffolding) — added as an anonymous inline module in flake.nix's modules list for substrate-vm, not inside vm-variant.nix itself, preserving the file's D-15-mandated imports = [ ./core.nix ]"

patterns-established:
  - "Seed nixosTest asserts the response body (not just HTTP 200) — the real-service-answers check pattern Phase 13's Databasise engine health check reuses"

requirements-completed: [FOUND-01, FOUND-02]

# Metrics
duration: ~35min (includes ~15min of first-time QEMU/nixosTest dependency builds)
completed: 2026-08-04
---

# Phase 9 Plan 3: Substrate Core, WSL/VM Variants, Seed Boot Test Summary

**One shared NixOS substrate module (`nix/substrate/core.nix`) now backs both the real WSL image target and a plain-QEMU variant that `nix flake check` boots end to end, asserting multi-user.target plus a loopback HTTP service's response body — the exact shape Phase 13's real Databasise health check will reuse.**

## Performance

- **Duration:** ~35 min (dominated by two first-time Nix builds: the full substrate closure and the QEMU nixosTest VM)
- **Started:** 2026-08-04T11:50Z (approx, first Read)
- **Completed:** 2026-08-04T12:10:32Z (last task commit)
- **Tasks:** 3
- **Files modified:** 6 (4 created, 2 modified)

## Accomplishments

- `nix/substrate/core.nix` defines the substrate exactly once: `sourcerer` user, Nix flakes/trusted-users, `programs.nix-ld` (the vscode-server mechanism per D-14, with a comment steering future readers away from re-adding the retired `nixos-vscode-server` input), `system.stateVersion = "26.05"`, and the D-12 seed placeholder (`python3Minimal -m http.server` on `127.0.0.1:8080` serving a docroot whose body is `sourcerer-substrate-seed-ok`) — no hardening template, no WSL specifics
- `nix/substrate/wsl-variant.nix` and `nix/substrate/vm-variant.nix` each import only `./core.nix` plus their own adapter layer (`wsl.enable`/`wsl.defaultUser` vs. nothing); the VM variant's header comment records the intentional gap (proves contents, not the WSL adapter — a real-WSL smoke test is Phase 10's job)
- `flake.nix` gained the `nixos-wsl` input pinned to spike-010-validated tag `2605.7.2` (`follows = "nixpkgs"`), `nixosConfigurations.substrate-wsl`/`substrate-vm`, `packages.substrate-system` (the cacheable closure FOUND-01/Plan 06 target), and `packages.substrate-image-builder` (the `.wsl` tarball builder script, Phase 10's to run)
- `nix/checks/seed-boot-test.nix` boots the plain-VM variant under `pkgs.testers.runNixOSTest`, asserting `multi-user.target`, `seed-placeholder.service`, open port 8080, a response body containing `sourcerer-substrate-seed-ok` (body-asserted, not just a 200), and a successful `nix --version` inside the guest; `curl` is added only in the test's own node, never in `core.nix`
- **Proved, not just evaluated:** `nix build .#substrate-system` succeeded end to end (first full substrate closure build), `nix build .#checks.x86_64-linux.seed-boot-test` succeeded (a real QEMU VM boot with every assertion passing), and `nix flake check` exited 0 with all three flake checks (devShell, packages, the nixosTest) green

## Task Commits

1. **Task 1: Author the shared substrate core module** - `024f921` (feat)
2. **Task 2: Author both variants and wire them into the flake** - `9ec081f` (feat)
3. **Task 3: Author the seed boot test and wire it into flake checks** - `3b353c7` (feat)

## Files Created/Modified

- `nix/substrate/core.nix` - the D-15 single substrate definition (user, Nix settings, nix-ld, D-12 seed service, `stateVersion`)
- `nix/substrate/wsl-variant.nix` - `core.nix` + `nixos-wsl.nixosModules.default`, `wsl.enable`/`wsl.defaultUser` only
- `nix/substrate/vm-variant.nix` - `core.nix` only, header comment records the WSL-adapter gap
- `nix/checks/seed-boot-test.nix` - `pkgs.testers.runNixOSTest` booting the VM variant, asserted boot + loopback service response
- `flake.nix` - `nixos-wsl` input; `nixosConfigurations.substrate-wsl`/`substrate-vm`; `packages.substrate-system`/`substrate-image-builder`; `checks.x86_64-linux.seed-boot-test`
- `flake.lock` - relocked to include the `nixos-wsl` node at tag `2605.7.2`

## Decisions Made

- **Comments must avoid literally repeating the plan's own forbidden-substring checks.** The plan's automated verification greps `core.nix` for the absence of `0.0.0.0` and `vm-variant.nix` for the absence of `nixos-wsl` (to prove no non-loopback bind / no WSL module import). Explanatory comments that mentioned these strings by name (e.g. "never bind 0.0.0.0", "do NOT import nixos-wsl.nixosModules.default") tripped the same `grep -q` checks meant to catch a real leak. Reworded both comments to describe the constraint without repeating the literal forbidden string, while keeping the documentation value intact.
- **Standalone `nixosConfigurations.substrate-vm` needs a real root filesystem + grub device stanza to evaluate `config.system.build.toplevel`.** A `nixosTest` node gets automatic qemu-vm disk/boot scaffolding from the test framework; a plain `nixosSystem` output (needed so the plan's own verification can `nix eval .#nixosConfigurations.substrate-vm.config.system.build.toplevel.drvPath`) does not, and NixOS's bootability assertions (`fileSystems."/"` unset, no `boot.loader.grub.devices`) fail evaluation without one. Added a small anonymous inline module (`fileSystems."/"` + `boot.loader.grub.device`) to `flake.nix`'s `modules` list for `substrate-vm`, deliberately kept out of `nix/substrate/vm-variant.nix` itself so that file's `imports` stay exactly `[ ./core.nix ]` as D-15 and the plan's acceptance criteria require. This is eval-only scaffolding for the standalone system output, not substrate content, and does not affect the `nixosTest`'s own boot (which imports `vm-variant.nix` directly, not the `substrate-vm` `nixosConfigurations` output).

## Deviations from Plan

### Auto-fixed Issues

**1. [Rule 1 - Bug] Comment text tripped the plan's own forbidden-substring greps**
- **Found during:** Task 1 (core.nix verification) and Task 2 (vm-variant.nix verification)
- **Issue:** Explanatory comments in `core.nix` ("never bind 0.0.0.0") and `vm-variant.nix` ("Do NOT import nixos-wsl.nixosModules.default") contained the exact literal strings the plan's `grep -q "0.0.0.0"` / `grep -q "nixos-wsl"` checks treat as failure signals, causing false-positive verification failures despite correct code.
- **Fix:** Reworded both comments to convey the same guidance without repeating the literal forbidden substring (e.g. "never bind any wildcard address"; "the NixOS-WSL flake input's nixosModules.default").
- **Files modified:** nix/substrate/core.nix, nix/substrate/vm-variant.nix
- **Verification:** Re-ran the plan's exact verify blocks for both tasks; all greps passed.
- **Committed in:** `024f921` (Task 1), `9ec081f` (Task 2)

**2. [Rule 3 - Blocking issue] Standalone `nixosConfigurations.substrate-vm` failed to evaluate without a root filesystem/bootloader stanza**
- **Found during:** Task 2 (`nix eval .#nixosConfigurations.substrate-vm.config.system.build.toplevel.drvPath`, an explicit acceptance criterion)
- **Issue:** `config.system.build.toplevel` failed NixOS's bootability assertions (`fileSystems` doesn't specify root; `boot.loader.grub.devices` unset) — a plain `nixosSystem` output has none of the automatic qemu-vm scaffolding a `nixosTest` node gets.
- **Fix:** Added an anonymous inline module (`fileSystems."/"`, `boot.loader.grub.device`) to the `substrate-vm` entry's `modules` list in `flake.nix`, with a comment explaining it's eval-only scaffolding, not substrate content — kept out of `nix/substrate/vm-variant.nix` so that file's `imports` stay exactly `[ ./core.nix ]`.
- **Files modified:** flake.nix
- **Verification:** `nix eval --raw .#nixosConfigurations.substrate-vm.config.system.build.toplevel.drvPath` now succeeds; the `nixosTest` (which imports `vm-variant.nix` directly, unaffected by this change) still boots correctly with its own qemu-vm plumbing.
- **Committed in:** `9ec081f` (Task 2 commit)

---

**Total deviations:** 2 auto-fixed (1 Rule 1 — comment wording collided with the plan's own verification greps; 1 Rule 3 — blocking Nix evaluation issue on the standalone VM system output). Both fixes are load-bearing for the plan's own stated acceptance criteria and introduce no scope creep — no substrate content changed, only comment phrasing and eval-only flake scaffolding.

## Issues Encountered

- `nix eval`/`nix build` against the local flake require new files to be `git add`-ed before Nix's Git-tree filter will see them (a standard Nix flakes behavior, not a bug) — staged each new file with `git add` before evaluating, consistent with per-task atomic commits.
- First-time builds of the full substrate closure (`nix build .#substrate-system`) and the QEMU nixosTest (`nix build .#checks.x86_64-linux.seed-boot-test`) each took several minutes (real VM boot + a large NixOS-WSL closure download/build) — run in the background per the environment note, both completed successfully with no retries needed.

## User Setup Required

None — no external service configuration required.

## Next Phase Readiness

- Plan 04 (CI + toolchain drift gate) can now wire `nix flake check` (which already includes the seed-boot-test) into GitHub Actions without further flake changes.
- Plan 05/06 (Attic binary cache + substituter wiring) has a real, buildable `packages.substrate-system` closure to push and pull — FOUND-01's "clone on a second machine, build from cache, no compiling" claim is provable once the cache is populated.
- `packages.substrate-image-builder` (the `.wsl` tarball builder) is evaluable and ready for Phase 10 to actually run and produce a distributable image; this phase deliberately stopped short of running it.
- The nixosTest/WSL gap is documented in `nix/substrate/vm-variant.nix`'s own header comment, not left implicit — Phase 10 owns closing it with a real-WSL smoke test.

---
*Phase: 09-flake-foundation-assurance-chain*
*Completed: 2026-08-04*

## Self-Check: PASSED

All six plan-listed files confirmed present on disk (nix/substrate/core.nix, nix/substrate/wsl-variant.nix, nix/substrate/vm-variant.nix, nix/checks/seed-boot-test.nix, flake.nix, flake.lock), plus this SUMMARY.md. All three task commit hashes (024f921, 9ec081f, 3b353c7) confirmed in `git log --oneline --all`.
