# Project Research Summary

**Project:** Sourcerer v3.0 "Theia Foundation"  
**Domain:** Branded desktop IDE/workbench application (fork of Eclipse Theia)  
**Researched:** 2026-08-18  
**Confidence:** HIGH (official Theia docs + reference implementations across multiple adopters)

## Executive Summary

Sourcerer v3.0 pivots from a bespoke Tauri shell to a fork of **Eclipse Theia IDE** (via the `theia-ide`/`theia-blueprint` product template), which already ships the entire docking/panel system, extension model, command palette, settings, theming, and AI framework that the v1.0 hand-built shell labored to create. The research confirms this is the correct architectural decision: Theia is explicitly designed to be forked as a branded product (not modified at the framework level), and multiple real adopters (Arduino IDE 2, Theia IDE itself) prove the pattern works. The core finding is a complete architectural reversal in approach: v1.0 built custom infrastructure, v3.0 composes an existing platform.

The recommended approach is to fork `eclipse-theia/theia-ide` (the product template, not the framework monorepo), consume `@theia/*` packages as versioned npm dependencies, extend only via Theia's documented extension/contribution-point mechanism in a separate `sourcerer-product` extension package, and keep all custom code isolated from the framework core. This preserves the ability to pull upstream Theia updates (monthly release cadence, currently at 1.74.1) without carrying a permanent merge burden — the entire value proposition of adopting Theia depends on this discipline.

Key risks: (1) forking the wrong repo (the framework monorepo instead of the product template) is the single most consequential decision — get it wrong and every later phase inherits merge-conflict debt, (2) native module ABI differs between Electron and Node targets, requiring separate rebuilds, (3) Node/Yarn toolchain versions drift across Theia releases and must be re-verified at every upgrade, (4) EPL-2.0 licensing is weak copyleft (file-level only, not repository-level) but trademark reuse is a separate compliance risk that needs a dedicated legal review before public release.

## Key Findings

### Recommended Stack

Theia 1.74.x (latest monthly release as of 2026-08-18) is the fork target, pinned in `theia-ide`'s own `package.json` at `1.74.100` with `@theia/*` core packages at `1.74.1`. Node >=22 is a hard floor (native module ABI requirement), Yarn Classic 1.7+ (not Berry/2+), and Electron's version is determined by the fork itself (read from `applications/electron/package.json` once scaffolded). Lerna + Yarn workspaces come pre-wired in the fork; don't introduce additional tooling.

**Core technologies:**
- **Fork target: `eclipse-theia/theia-ide`** (the product template, formerly "Theia Blueprint") — Consume as a composed, branded desktop product, not by editing framework source. This is the Eclipse Foundation's official pattern for exactly this scenario (fork Theia into your own branded IDE).
- **@theia/core + @theia/* packages at 1.74.1** — Versioned npm dependencies, not framework monorepo submodules. Monthly release cadence, use current release, not Community Release track (Sourcerer is doing foundational work, not running stable production yet).
- **Node.js >=22 (24 recommended)** — Hard engine floor for native-module ABI; Yarn's `@theia/cli rebuild:electron` step is ABI-sensitive to Node/Electron pairing.
- **Yarn Classic 1.7+ (NOT Yarn Berry 2/3/4)** — Engine-pinned in `theia-ide`; Theia's native-rebuild scripts and workspace resolution authored against Classic semantics only.
- **Electron (version TBD from fork)** — Single desktop app target for this milestone; the fork ships both `applications/electron` and `applications/browser` targets, but Sourcerer runs Electron only (local workbench, not hosted/multi-user).

**No component library, no pixel-perfect shell geometry port.** The green accent (`#86A38C`) and IBM Plex fonts carry forward via Theia's native theme system (CSS custom-properties, VS Code theme-JSON compatible), but Theia's own Lumino-based shell geometry is a given — forcing the old bespoke 40px/0-radius tokens onto it fights the framework.

### Expected Features

**Must have (P1 — table stakes for a foundation milestone):**
- Reproducible local build (`yarn`/`theia build`, documented and verifiable on a clean checkout) — this is the actual deliverable, not a side effect
- Application name/window title (`applicationName` config in `applications/*/package.json`)
- Product icons (all platforms: `.ico` Windows, `.icns` macOS, `.png` set Linux) — Theia ships defaults; sourcing Sourcerer-branded icons is a low-cost branding surface
- Product Extension package (`sourcerer-product` extension) — centralized branding layer (About dialog, Getting Started widget, window title behavior) following Theia's own reference pattern
- Getting Started / Welcome widget rebranded — zero stock Theia/Eclipse branding visible on first launch, or the rebrand reads as incomplete
- Sensible default first-launch layout (explorer/editor/empty panel coherent default, doesn't need to resemble v1.0's home dashboard yet)
- Documented dev workflow (equivalent of the v1.0 `tauri dev` landmine notes — capture which `theia build`/`theia start` commands actually work reliably here)

**Should have (P2 — low-risk parallelize, improves but not required for v1):**
- Theme identity port (green accent `#86A38C` + IBM Plex via Theia's theme system) — mechanical, safe to parallelize with P1 branding work

**Defer (P3 — explicitly out of scope, next milestone):**
- Theia AI framework adoption (Dashboard Assistant port) — PROJECT.md flags carry-over review as deferred; different architecture enough from v1.0's `host.ai()` that rushing this now risks re-doing it wrong twice
- Applet framework / host API mapping onto Theia extensions — research-worthy decision on how to model applets as Theia extensions vs VS Code plugins; explicitly part of the deferred carry-over review
- Feature stripping (debugger, task runner, extension marketplace, terminal, etc.) — Default to hiding via preferences, not deleting packages; Theia's value proposition is exactly this IDE machinery; over-stripping early forfeits the reason to be on Theia and re-adding removed features later is harder than just leaving them disabled

### Architecture Approach

Theia's three-process model (Electron main → Frontend + Backend + Plugin host) replaces Tauri's two-process model, with a key difference: the backend is always a separate Node.js process (even in Electron target), connected to the frontend via JSON-RPC over WebSocket, same as the browser target. This architecture is inherited directly; forking doesn't change it. Product-specific code lives in a separate `sourcerer-product` extension package (or sibling `theia-extensions/*` packages for future applets), never as patches to core Theia files.

**Major components:**
1. **Frontend (Lumino-based widget system, Monaco editor, Inversify DI container)** — Runs in Electron renderer or browser, hosts dockable panels, command palette, menus, keybindings, theming. Theia's own UI shell (the "workbench") is the descendant of what v1.0's dockview-core + Zustand tried to build custom.
2. **Backend (Node.js, Express + WebSocket)** — Filesystem access, workspace management, LSP/DAP servers, terminals, Git, backend halves of extension services. RPC-proxied to frontend; Theia auto-generates the proxy from TypeScript interfaces.
3. **Plugin Host (separate Node process, spawned by backend)** — Runs VS Code (`.vsix`) extensions in isolation via the VS Code extension API translation layer; webviews render back in the frontend.
4. **Electron main (thin, window/lifecycle only)** — Doesn't contain product logic; spawns backend, handles OS chrome, app lifecycle.

**Extension mechanisms (load-bearing distinction):** Theia extensions are compile-time, npm packages with full Inversify DI access (rebind core services, contribute widgets, reach filesystem/spawn processes); VS Code plugins are runtime-sandboxed, API-limited, distributable independently. Branding and shell-privileged infrastructure (the assistant panel, future applets needing host-API-style access) belong in Theia extensions; reusable ecosystem pieces belong in VS Code plugins.

### Critical Pitfalls

1. **Forking `eclipse-theia/theia` (framework monorepo) instead of `theia-ide` (product template)** — The framework repo is explicitly not designed to be forked per community discussion. Sourcerer-specific code belongs in a separate `sourcerer-product` extension, consuming `@theia/*` as versioned npm dependencies. Forking the framework repo turns "consume Theia as a maintained upstream" into "maintain a real fork" — the entire value proposition of the pivot collapses.

2. **Native module ABI differs between Electron and browser targets** — Native modules compiled for one target's Node ABI don't load in the other. The fork provides separate rebuild scripts (`yarn rebuild:electron` vs `yarn rebuild:browser`); switching targets requires re-running the rebuild step. Treating this as a one-off fluke per machine instead of a standing requirement per target leads to silent breakage.

3. **Node.js and Yarn version drift across Theia upgrades** — Theia pins version ranges in `package.json` `engines` per release, and these change. A floating "whatever's current" Node/Yarn, or a version pinned once and never revisited, silently breaks after a Theia version bump with confusing downstream errors (native ABI mismatch, Yarn resolution differences). Prevention: pin via Nix devShell (flake) keyed to the exact Theia version in use, and re-verify at every deliberate upgrade as an explicit step.

4. **Depending on `@experimental` Theia APIs that break on minor bumps** — Theia's policy explicitly permits breaking `@experimental` APIs in any minor release with no deprecation cycle (only `@stable` APIs get semver guarantees). Prevention: check every Theia API/extension-point Sourcerer's code depends on for its `@experimental`/`@stable`/`@deprecated` JSDoc tag before adopting it; keep a registry of those tags reviewed at every Theia version bump.

5. **EPL-2.0 and trademark compliance misunderstood** — EPL-2.0 is weak (file-level only) copyleft: modifications to EPL-covered files themselves must stay EPL-licensed if distributed, but separately-distributed modules that merely interoperate are not derivatives (this is exactly what Theia's product-extension pattern is designed to be). Separately, license and trademark are different: EPL-2.0 grants copyright/patent permissions, but using "Eclipse Theia" name/logo requires separate authorization. Prevention: structure all Sourcerer code as a separate product extension, keep original copyright notices intact, do not name the product "Theia" anything, and get a dedicated legal review before public/commercial distribution (this research pass is a starting map, not sign-off).

## Implications for Roadmap

Based on research, suggested phase structure for v3.0:

### Phase 1: Foundation & Scaffolding
**Rationale:** The milestone's stated goal is "working, rebranded fork building locally; dev workflow documented." Nothing else is testable until the fork builds and runs. This phase answers the kill question: does a Theia fork scaffold at all on this machine, and what are the build/Electron/native-module quirks we need to document.

**Delivers:** 
- Unmodified `theia-ide` fork building locally on the dev machine (`yarn build:dev`, `yarn rebuild:electron`)
- Electron app launching and window chrome appearing (pixel content can be vanilla Theia for now)
- Nix devShell pin (Node/Yarn versions locked to this Theia's `engines` requirements)
- Documented, reproducible dev workflow (which commands work reliably, which don't, native-rebuild step placement)

**Addresses features (P1):** Reproducible local build  
**Avoids pitfalls:** Pitfall 1 (repo fork strategy), Pitfall 3 (toolchain drift)  
**Research flags:** May need deep dive into NixOS Electron sandbox setup if native-module rebuild fails; document the actual commands/workarounds found.

### Phase 2: Branding & Product Extension
**Rationale:** Once the unmodified fork proves buildable, layering on Sourcerer-specific branding is a mechanical, low-risk extension using Theia's designed-for pattern (the Product Extension + rebind mechanism). This is the first "our fork, not vanilla Theia" proof; it's where the rebranding work concentrates.

**Delivers:**
- `sourcerer-product` extension package created (`theia-extensions/sourcerer-product/`), wired into the app's `package.json` dependency list
- Product icons sourced and installed (all three platforms: Windows `.ico`, macOS `.icns`, Linux `.png`)
- Branding applied via rebind pattern: window title, About dialog, Getting Started widget (Sourcerer name/content, zero Eclipse/Theia boilerplate)
- Product Extension wired into the build (generated bundle includes the rebindings)
- Verified visually: window title reads "Sourcerer," About shows Sourcerer info, Getting Started shows Sourcerer welcome

**Addresses features (P1):** Application name, window icons, Product Extension, Getting Started rebrand, About dialog  
**Uses architecture:** Product Extension pattern (rebind via Inversify DI), Theia's contribution-point mechanism  
**Avoids pitfalls:** Pitfall 5 (EPL/trademark — product code kept separate, not patching core), Pitfall 6 (not stripping IDE features)  
**Research flags:** None — this is a well-documented, standard Theia pattern (Arduino IDE 2 and Theia IDE itself use this exact mechanism).

### Phase 3: Theme Identity & UX Polish
**Rationale:** Green accent color and IBM Plex font port are mechanically independent of Phase 2's branding work (can parallelize, but low priority for foundation milestone); they're safe P2 work that improves but isn't required for a functional fork.

**Delivers:**
- Green `#86A38C` accent applied via Theia's native theme system (CSS custom-properties, VS Code theme-JSON compatible)
- IBM Plex Sans/Mono/Serif imported and set as default fonts
- Default first-launch layout tweaked if Theia's out-of-box default feels wrong (but doesn't need to resemble v1.0's dashboard geometry — treat as "coherent and usable," not "pixel-identical to old shell")

**Addresses features (P2):** Theme identity port  
**Avoids pitfalls:** Avoids Pitfall 6 (not trying to port v1.0's exact pixel geometry, which fights Theia's native chrome)  
**Research flags:** None — Theia's theme system is standard, CSS-based, with no novel integration surprises expected.

### Phase 4: Carry-Over Review & Integration Decisions (explicitly deferred, next milestone)
**Rationale:** PROJECT.md explicitly defers all carry-over decisions (Pi harness, Databasise integration, applet-framework-to-Theia mapping, assistant port) to a later phase. These are architecture decisions (not implementation details) that need their own research pass — trying to fit them into the foundation phase risks either shipping incomplete or re-doing them wrong twice.

**Out of scope this milestone:**
- Theia AI framework adoption (dashboard assistant port) — Different architecture from v1.0's `host.ai()` seam; requires deciding which Theia extension patterns (contribution points, DI bindings) to receive it
- Applet framework mapping — Deciding whether applets become Theia extensions (compile-time, full access) or VS Code plugins (runtime, sandboxed); both are viable, needs explicit decision per applet type
- Databasise integration mode (REST/MCP endpoints, sidecar spawning, backend extension wiring) — Currently undecided per PROJECT.md; flag for the next milestone's architecture work
- Sourcerer v1.0 workspace-layout persistence strategy (does Theia's built-in layout system replace it, or do we preserve the old `workspace.json` pattern?) — Explicitly a carry-over-review topic

### Phase Ordering Rationale

- **Phase 1 first:** Nothing else is testable without a building fork. Phase 2's branding work landed on a fork that doesn't run is wasted.
- **Phase 2 after Phase 1:** Branding is the next lowest-risk, highest-confidence work; it proves the Product Extension pattern and answers "does our fork look branded or vanilla?"
- **Phase 3 parallel or after Phase 2:** Theme/font work is independent, safe to parallelize with Phase 2 or defer to after; it's strictly polish, not blocking anything else.
- **Phase 4 deferred:** Carry-over decisions are too large and interdependent (Pi harness, Databasise, applet architecture) to rush into Phase 3.0; they need their own milestone with their own research pass.

### Research Flags

Phases likely needing deeper research during planning:
- **Phase 1 (Foundation):** NixOS Electron native-module setup if the `yarn rebuild:electron` step fails — this is Electron-on-NixOS specific; research may need to dive into sandbox-helper / build-tools setup per the user's environment.

Phases with standard patterns (skip research-phase):
- **Phase 2 (Branding):** Well-documented in official Theia docs and multiple adopter examples (Arduino IDE 2, Theia IDE's own `theia-blueprint-product`); the Product Extension + rebind pattern is the reference implementation.
- **Phase 3 (Theme):** Theia's theme system is CSS-based, standard; no novel integration risks.

## Confidence Assessment

| Area | Confidence | Notes |
|------|------------|-------|
| **Stack** | HIGH | Fork target confirmed via official GitHub + npm registry queries; version numbers directly verified; Yarn Classic and Node >=22 hard-confirmed from `theia-ide` package.json and `eclipse-theia/theia` docs. |
| **Features** | HIGH | Theia's built-in capabilities (dock/panel shell, extension system, settings, theming, AI framework) are documented official features; MVP for a "foundation" phase is well-established across adopter examples (Theia IDE, Arduino IDE 2). |
| **Architecture** | HIGH | Theia's three-process model and extension mechanisms documented in official docs; Project Structure mirrors `theia-blueprint` reference and `theia-ide` real implementation; Inversify DI pattern is standard across both. |
| **Pitfalls** | MEDIUM–HIGH | Critical pitfalls (Pitfall 1 fork strategy, Pitfall 2 native ABI) confirmed via official docs + community discussions; EPL-2.0 compliance from official license text + FOSSA legal summary (HIGH for weak-copyleft mechanics, MEDIUM for trademark risk — trademark needs dedicated legal review). NixOS/Electron-specific pitfalls (sandbox, native modules) are general Electron-on-NixOS knowledge applied by analogy (not Theia-specific docs), flagged LOW where applicable. |

**Overall confidence:** HIGH — Stack, features, and architecture are HIGH-confidence across official sources; pitfalls are well-documented with clear prevention strategies. One known gap: trademark/licensing compliance for public release needs legal review (out of scope for this research pass).

### Gaps to Address

1. **NixOS-specific Electron build environment** — Research covered general "Electron on NixOS" gotchas (sandbox helper, native modules) but didn't run a test build on the actual dev machine. Phase 1 scaffolding will surface the real requirements.

2. **Databasise integration mode (REST vs MCP vs embedded)** — Explicitly undecided per PROJECT.md; flagged as a carry-over-review topic for v3.0 or later. Architecture is independent of this milestone (Phase 1–3 don't touch backend integration).

3. **AGPL + EPL-2.0 licensing interaction** — PROJECT.md commits to AGPL for the Sourcerer repo, but base layer is EPL-2.0/GPL-2.0-classpath. Mechanics are complex enough that a dedicated legal review pass is needed before any public release. Current foundational work (forking, building locally) is not blocked by this.

4. **Exact Electron version** — Will be determined by the fork's `applications/electron/package.json` at scaffold time; research documented the principle (read it off the fork, pin it with Node for ABI compatibility) but not the exact current version (which changes monthly with Theia releases).

## Sources

### Primary (HIGH confidence)
- [github.com/eclipse-theia/theia-ide](https://github.com/eclipse-theia/theia-ide) — Repository identity, fork structure, product template confirmation
- [github.com/eclipse-theia/theia-blueprint (redirect)](https://github.com/eclipse-theia/theia-blueprint) — Historical confirm (blueprint renamed to theia-ide); multiple public forks prove the pattern works
- **Raw package.json** (theia-ide master, eclipse-theia/theia master) — Version numbers, engines, workspace layout, verified directly against live GitHub
- **npm registry direct queries** (@theia/core, @theia/cli, @tauri-apps/plugin-store versions, license fields) — Authoritative for version state as of 2026-08-18
- [theia-ide.org — official docs](https://theia-ide.org/docs/) — Extension system, product composition, branding customization, Theia AI framework
- [eclipse.org/legal/epl-2.0](https://www.eclipse.org/legal/epl-2.0/) — Official EPL-2.0 license text and mechanics

### Secondary (MEDIUM confidence)
- [GitHub Discussion #9862 (Theia)](https://github.com/eclipse-theia/theia/discussions/9862) — Community confirmation of "fork theia-ide, not theia monorepo" pattern
- [Arduino IDE 2.0 Adopter Story](https://blogs.eclipse.org/post/john-kellerman/theia-adopter-story-new-arduino-ide-20/) — Real-world example of fork + branding pattern working in production
- [eclipsesource.com — Theia release blog series](https://eclipsesource.com/blogs/2026/08/11/eclipse-theia-1-74-release-news-and-noteworthy/) — Monthly release cadence and dates confirmed
- [Celebrating Eclipse Theia's Full VS Code Extension API Compatibility](https://eclipse-foundation.blog/2023/12/18/celebrating-eclipse-theias-milestone-full-compatibility-with-vs-code-extension-api/) — Plugin-host architecture and VS Code compat confirmed
- Project's own `.planning/PROJECT.md` — Milestone scope, AGPL decision, carry-over review deferrals

### Tertiary (LOW confidence, needs validation)
- General NixOS/Electron sandbox and native-module knowledge applied by analogy to Theia (not Theia-specific documentation) — Flagged for Phase 1 validation on actual dev machine

---

**Research completed:** 2026-08-18  
**Ready for roadmap creation:** Yes

**Next step:** Roadmapper uses this SUMMARY.md to structure v3.0's phase breakdown and identify which phases need detailed planning (`/gsd-plan-phase`) vs. which can follow established patterns.
