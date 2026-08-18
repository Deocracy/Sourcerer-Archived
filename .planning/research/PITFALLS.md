# Pitfalls Research

**Domain:** Forking and rebranding Eclipse Theia as a desktop-product foundation
**Researched:** 2026-08-18
**Confidence:** MEDIUM (official Theia/Eclipse docs + GitHub issues/discussions, cross-checked across multiple queries; no Theia-specific NixOS reports found — NixOS section is Electron/Nix general knowledge applied to Theia, flagged LOW where noted)

## Critical Pitfalls

### Pitfall 1: Forking the Theia monorepo instead of consuming it as a template + extensions

**What goes wrong:**
Teams `git clone` `eclipse-theia/theia` itself (the platform monorepo) and start editing it directly, intending to "fork Theia." Weeks later, pulling upstream fixes means resolving merge conflicts across hundreds of Lerna-managed packages, and the fork has silently diverged from what `yarn.lock` says is installed — local dev setups are documented to ignore lockfiles and regenerate a different `node_modules` tree than production would get.

**Why it happens:**
"Fork Theia" sounds like "fork the repo." But Theia's own community explicitly steers people away from this (GitHub Discussion #9862): npm/yarn cannot install packages from *within* a monorepo cleanly, so working against modified core packages either means `yarn link`/`npm link` (works for solo dev, brittle for a team) or standing up a local Verdaccio registry to publish and consume modified packages like real dependencies.

**How to avoid:**
Fork/clone **`theia-ide`** (the product template, formerly "Theia Blueprint"), not the `theia` platform repo. Consume `@theia/*` packages as versioned npm dependencies in the product's `package.json`. Only touch the platform monorepo itself if contributing a fix upstream (via PR) — never as the long-term home for Sourcerer's own code. All Sourcerer-specific behavior belongs in a separate **product extension** (Theia's supported customization surface — see Pitfall 6), not as edits to core packages.

**Warning signs:** Any workflow that requires editing files inside `packages/core`, `packages/plugin-ext`, etc. to make Sourcerer work. A `git diff` against upstream Theia that touches core package source rather than only `applications/electron/*` and a dedicated product package.

**Phase to address:** Foundation phase 1 (repo setup / fork strategy) — this is the single most consequential decision in the whole milestone; get it wrong and every later phase inherits merge-conflict debt.

---

### Pitfall 2: Native module build failures (node-gyp) treated as one-off flukes instead of a standing build-environment requirement

**What goes wrong:**
`yarn`/`yarn build` fails compiling native addons (`nsfw` file watcher, `native-keymap`, `keytar`) with node-gyp/N-API errors (e.g. `'ThreadSafeFunction': is not a member of 'Napi'`), or fails silently behind a corporate/dev-host proxy. Separately — and specific to Theia's dual-target architecture — **Electron and browser (Node.js) targets link different native module ABIs**; rebuilding native modules for one target (`yarn electron rebuild` vs plain Node) breaks the other target if you switch between `yarn start` (browser/Node) and `yarn electron start` without re-rebuilding.

**Why it happens:**
node-gyp needs a full native toolchain (Python 3, a C/C++ compiler, on Windows the "Desktop development with C++" workload) that isn't guaranteed present, and Electron's V8/Node ABI differs from a plain Node.js install, so a native module compiled for one doesn't load in the other.

**How to avoid:**
Document the native-toolchain prerequisite explicitly in the dev setup (not just "run yarn"). Standardize on one target during a given work session (electron OR browser) and script the rebuild step (`electron-rebuild` / Theia's own rebuild scripts) into `yarn electron start` so it's not a manual, forgettable step. Pin `node-abi`/`electron` versions together — bumping Electron without checking native-module ABI compatibility is a recurring breakage source project-wide (not Theia-specific, but Theia inherits it via `nsfw`/`keytar`/`native-keymap`).

**Warning signs:** "Works in browser mode, breaks in Electron" (or vice versa) after a clean build. `node-gyp rebuild` errors mentioning N-API/ABI version mismatch immediately after bumping the Electron or Node version.

**Phase to address:** Foundation phase covering "local build reproducibility" — should ship a documented, scripted (not tribal-knowledge) build command per target, verified on the actual NixOS dev host before being called done.

---

### Pitfall 3: Assuming a single global Node/Yarn toolchain works across Theia upgrades

**What goes wrong:**
Theia pins Node.js and Yarn (classic Yarn 1.x, **not** Berry/2+) version ranges per release in `package.json` `engines`. A dev host with a floating "whatever's current" Node/Yarn, or a version pinned once and never revisited, silently breaks or half-works after a Theia version bump — sometimes with confusing downstream errors (native module ABI mismatch again, or Yarn resolution differences) rather than a clear "wrong Node version" message.

**Why it happens:**
The requirement isn't a single number ("Node 18") — it drifts per Theia release, and nothing in a typical local shell enforces it unless deliberately pinned (nvm/asdf/Nix devShell).

**How to avoid:**
Pin Node + Yarn via a Nix devShell (flake) checked into the repo, keyed to the exact Theia version in use, and re-verify/update the pin as part of any deliberate Theia version bump — never assume last version's toolchain still applies. Treat "what Node/Yarn does this Theia version want" as a question to re-ask at every upgrade, not a one-time setup task.

**Warning signs:** Build works for one dev, fails for another with no code diff. Build breaks immediately after bumping the `theia` dependency version with no other change.

**Phase to address:** Foundation phase 1 (repo + toolchain setup) for the initial pin; any later "upgrade Theia" phase must re-verify it as an explicit step, not an assumption.

---

### Pitfall 4: Treating experimental Theia APIs as stable, then getting broken by a routine minor upgrade

**What goes wrong:**
Sourcerer's product extension calls into a Theia extension point or API that isn't marked `@stable`. Six months later a routine `yarn upgrade` to the next Theia minor silently breaks the integration — because Theia's own policy explicitly permits breaking `@experimental` APIs in *any* minor release, with no deprecation cycle. Only `@stable` APIs get semver guarantees and a deprecation-then-removal cycle spanning a major version.

**Why it happens:**
Most of what's genuinely useful/new in an extensible framework starts life `@experimental` — and it's easy to not check the JSDoc stability tag before depending on something, especially mid-flow when following a tutorial or an existing extension's example code.

**How to avoid:**
Before depending on any Theia API/extension-point in Sourcerer's product code, check its `@experimental`/`@stable`/`@deprecated` JSDoc tag. Keep a short internal list of "APIs Sourcerer depends on + their stability tag" reviewed at every Theia version bump. Prefer `@stable` surfaces (VS Code extension API surface is a strong stable target — see Pitfall 6) over reaching into internal Theia extension points where a stable alternative exists.

**Warning signs:** A Theia minor bump (not major) breaks a build or runtime behavior with no corresponding CHANGELOG entry under "Breaking Changes" for a stable API — check whether the broken symbol was tagged `@experimental`.

**Phase to address:** Every phase that adds a Theia-API integration point should record which API/tag it used; a later "Theia upgrade" phase should audit against that list before bumping.

---

### Pitfall 5: EPL-2.0 obligations misunderstood as either "must open-source everything" or "no obligations at all"

**What goes wrong:**
Two failure modes, both wrong:
1. Over-caution: assuming EPL-2.0's copyleft means all of Sourcerer's own applet/product code must also be EPL-2.0-licensed and public.
2. Under-caution: stripping/altering Theia's existing copyright, patent, and trademark notices when rebranding, or using the "Eclipse Theia" name/logo as if it were available for Sourcerer's own branding.

**Why it happens:**
EPL-2.0 is a weak/file-level copyleft (similar in spirit to LGPL, distinct from GPL): it only requires that *modifications to EPL-covered files themselves* stay EPL-licensed if distributed, and does not extend to separately-distributed "Modules" that merely link/interoperate with the EPL code — which is exactly what Theia's product-extension pattern is designed to be. Separately, license terms and trademark rights are different things: EPL-2.0 grants copyright/patent permissions, but using the Eclipse Foundation's or contributors' trademarks/logos requires separate authorization the license does not grant.

**How to avoid:**
- Structure ALL Sourcerer-specific code as a product extension / separate package, per Theia's documented pattern — never edit EPL-covered core files in place with Sourcerer-only logic (also serves Pitfall 1's fork-avoidance goal).
- Keep Theia's existing copyright/attribution notices intact in distributed builds; add Sourcerer's own notices alongside, don't replace.
- Do not name the product "Theia" anything, or reuse the Eclipse Theia logo, without checking Eclipse Foundation trademark guidelines — treat the name/logo as off-limits by default.
- Get an actual license read (not just this research pass) before any public/commercial distribution — this document is a starting map, not legal sign-off.

**Warning signs:** Marketing copy or the About dialog implying Sourcerer *is* Eclipse Theia rather than *built on* it. Any commit that edits a core Theia file's license header.

**Phase to address:** Branding/customization phase — bake the "separate product extension, notices preserved, no Theia trademark reuse" rule into the branding phase's acceptance criteria directly.

---

### Pitfall 6: Stripping IDE features during "de-IDE-ify for a research app" branding, then needing them back later

**What goes wrong:**
Because Sourcerer is a research workbench, not a code editor, teams are tempted to rip out IDE-specific chrome early (extension marketplace, debug views, terminal, source-control view, VS Code extension host) to "clean up" the shell. Later, a planned applet (e.g. a code/notebook feature, or just wanting the VS Code extension ecosystem for something like Markdown/PDF viewers) needs one of those pieces back, and by then Sourcerer's product extension has grown assumptions that fight reintroducing it.

**Why it happens:**
Theia's biggest differentiator versus hand-rolling a shell (which is literally what the prior Sourcerer milestone was doing) is exactly this IDE machinery — the Monaco editor integration, full VS Code extension-API compatibility (achieved as of the widely-cited "full compatibility" milestone), LSP, debug adapter protocol. Removing it early forfeits the reason to be on Theia at all, and re-adding a *removed* Theia contribution point is harder than never removing it (contribution points are additive/opt-in by design — leaving something un-enabled costs nothing; ripping it out and re-wiring it later costs real effort).

**How to avoid:**
Default to **hiding** unused Theia contributions via configuration/preferences (view visibility, menu contributions, default layout) rather than deleting the packages that provide them. Only remove a package from the dependency tree if genuinely certain it will never be wanted (and even then, prefer "not installed yet" over "ripped out with product code now assuming its absence"). Arduino IDE 2.0 (a documented Theia adopter) took this approach: layered a simplified UI over the full Theia/Monaco/LSP shell for beginners rather than stripping the shell down.

**Warning signs:** Product-extension code that assumes a Theia core view/service doesn't exist (rather than just isn't shown by default). Deleting a `@theia/*` package dependency instead of disabling its contribution via preferences.

**Phase to address:** Branding/customization phase — the acceptance criterion should be "unwanted IDE chrome is hidden, not deleted" unless a specific package is confirmed unnecessary for the whole product's life.

---

## Technical Debt Patterns

Shortcuts that seem reasonable but create long-term problems.

| Shortcut | Immediate Benefit | Long-term Cost | When Acceptable |
|----------|-------------------|----------------|-----------------|
| Cherry-picking unreleased fixes from Theia `master` into the pinned release version (Theia's own documented "consume fixes without upgrading" pattern) | Unblocks a specific bug without waiting for the next monthly release | Each backport is manual, untested against the full codebase state it was written against, and accumulates as a growing, undocumented diff from upstream — official docs call this workable only when the fix is narrow and self-contained | Only for a single, isolated, well-understood fix with no interaction with other unreleased changes; never as a standing practice |
| Editing Theia core package source directly instead of using the product-extension pattern | Fastest way to get a UI tweak working right now | Converts "consume Theia as a dependency" into "maintain a real fork," reintroducing Pitfall 1's merge-conflict burden at every upgrade | Never — always look for the extension-point/contribution alternative first |
| Deleting unused `@theia/*` packages early to slim the build | Smaller bundle, less surface area to reason about now | Re-adding a removed IDE feature later (Pitfall 6) costs more than just leaving it disabled | Only once a package's total non-use is certain for the product's life, not just "not needed yet" |
| Skipping the Nix devShell toolchain pin and using whatever Node/Yarn is on the dev host | Faster to start building today | Silent breakage across dev machines and after Theia version bumps (Pitfall 3) | Never on a shared/long-lived project; acceptable only for a throwaway spike |

## Integration Gotchas

Common mistakes when connecting to external services/tooling in a Theia fork.

| Integration | Common Mistake | Correct Approach |
|-------------|----------------|-------------------|
| electron-builder packaging | Assuming Linux AppImage icon fallback behaves like other targets | Handle Linux icon/asset paths explicitly per Theia IDE's own `electron-builder.yml`; don't copy Windows/macOS config assumptions blindly |
| VS Code extension host / marketplace | Wiring a custom extension registry without checking Theia's existing VS Code-API-compatible plugin-loading path | Theia already has full VS Code extension API compatibility (as of the widely-documented 2023 milestone) — reuse the existing plugin host rather than building a parallel one |
| Yarn workspace + Theia's own dependency graph | Adding a Sourcerer dependency at the wrong workspace level, causing hoisting conflicts with a `@theia/*` package's own pinned sub-dependency | Add Sourcerer-only dependencies inside the product-extension package, not the repo root, unless truly shared infra |

## Performance Traps

| Trap | Symptoms | Prevention | When It Breaks |
|------|----------|------------|----------------|
| Cold `yarn`/`yarn build` with no cache in CI | Every CI run recompiles the full Theia monorepo (many packages) from scratch | Cache the Yarn cache directory itself (and `install-state.gz` if on Yarn Berry-adjacent tooling), not just `node_modules`; consider a build-cache tool (Turbo-style) for warm-build savings reported around 70% in comparable Node monorepos | As soon as CI is added — not a "later" concern, first CI run already pays the full cost |
| Theia's dev-mode filesystem watcher against a large repo/workspace tree on Linux | `ENOSPC` errors, watcher silently stops updating | Raise `fs.inotify.max_user_watches` on the Linux/NixOS dev host before it becomes a live blocker | Any workspace of nontrivial file count — hits early on Linux dev hosts specifically, not Windows/macOS |

## Security Mistakes

| Mistake | Risk | Prevention |
|---------|------|------------|
| Running Electron with sandbox disabled (`--no-sandbox`) to work around a NixOS SUID-sandbox-helper error rather than fixing the sandbox setup | Removes Chromium's process-level sandboxing for a real distributed product, not just a local dev convenience | Fix the underlying NixOS Electron sandbox binary (nixpkgs' Electron wrapper sets up the setuid `chrome-sandbox` helper correctly) rather than disabling the sandbox; reserve `--no-sandbox` for throwaway local dev only, never ship it |
| Reusing Theia's default preferences/plugin directories without scoping them to Sourcerer's own app identity | Cross-contamination with a real Theia IDE install on the same machine (shared config/plugin dirs), or a dev's personal Theia settings leaking into Sourcerer | Configure Sourcerer's own preferences-directory/app-id via the product extension (the `*-variables-server.ts` customization point Theia IDE itself uses) so it never shares state with vanilla Theia |

## UX Pitfalls

| Pitfall | User Impact | Better Approach |
|---------|-------------|-------------------|
| Shipping the full default Theia/VS Code-style IDE chrome (debug panel, source control, terminal, extensions marketplace icon) unchanged to non-technical scholar users | Confusing, code-editor-flavored UI for users who aren't writing code | Hide (not delete — see Pitfall 6) unused default views/menu contributions via preferences before any user-facing milestone; keep them available for later applets that do need them |
| About dialog / welcome page still reading generic "Theia IDE" boilerplate post-fork | Undermines the rebrand, looks unfinished/unbranded | Product-extension branding (about dialog, getting-started widget) is a first-class supported customization point — use it, don't leave defaults in place past the branding phase |

## "Looks Done But Isn't" Checklist

- [ ] **"Fork builds locally":** Often means "built once on the author's machine" — verify a clean-checkout build succeeds on a second machine/profile with only the documented Nix devShell, no ambient global Node/Yarn.
- [ ] **"Electron packaging works":** Often verified only via `yarn electron start` (dev mode) — verify the actual packaged installer (`yarn electron package`) launches and its native modules load (Electron-target rebuild, not browser-target).
- [ ] **"Branding applied":** Often covers the window title and app icon but misses the About dialog, welcome/getting-started page, and preferences directory scoping — verify all four, not just the visible chrome.
- [ ] **"Rebrand is EPL-compliant":** Often means "we changed the name" — verify original copyright/attribution notices are still intact in distributed files and that no Eclipse Theia trademark/logo asset is still bundled or referenced.
- [ ] **"Theia version pinned":** Often means "package.json says a version" — verify the Nix devShell's Node/Yarn versions actually match what that specific Theia version's `engines` field requires, not last version's pin.

## Recovery Strategies

| Pitfall | Recovery Cost | Recovery Steps |
|---------|---------------|-----------------|
| Accidentally forked/diverged the `theia` platform monorepo instead of using the product-extension pattern | HIGH | Diff the fork against the matching upstream tag to isolate real Sourcerer-only changes; extract those into a product-extension package; re-point the product to consume upstream `theia-ide`/`@theia/*` as versioned dependencies; treat the old fork as archived reference only |
| Native module ABI mismatch after an Electron/Node version bump | LOW | Re-run the target-specific rebuild (`electron-rebuild` for Electron target, plain `yarn` for Node/browser target); pin `node-abi`-relevant versions together going forward |
| Depended on an `@experimental` API that broke in a minor bump | MEDIUM | Check the CHANGELOG for the replacement API/pattern; if none exists yet, pin the Theia version until a migration path lands rather than patching around the break |
| Stripped an IDE feature (Pitfall 6) now needed by a later applet | MEDIUM–HIGH | Depends on how deep the removal went: if only hidden/disabled, re-enable via preferences (cheap); if the package was actually removed and product code assumes its absence, re-add the dependency and audit for code that assumed it wasn't there |

## Pitfall-to-Phase Mapping

| Pitfall | Prevention Phase | Verification |
|---------|-------------------|----------------|
| Forking the monorepo vs consuming as template+extensions | Foundation phase 1 (repo/fork strategy) | Repo structure review: Sourcerer-specific code lives only in a product-extension package + `applications/electron/*`, never inside a `@theia/*` core package |
| Native module build failures / dual-target ABI mismatch | Foundation phase (build reproducibility) | Clean-checkout build succeeds for both `yarn start` and `yarn electron start` on the NixOS dev host without manual intervention |
| Node/Yarn version drift across Theia upgrades | Foundation phase 1 (toolchain pin) + any future Theia-upgrade phase | Nix devShell pin matches the in-use Theia version's `engines` field; documented re-check step exists for future upgrades |
| Experimental API breakage on minor bumps | Every phase adding a Theia API integration; audited at upgrade phases | A tracked list of "Theia APIs Sourcerer depends on + stability tag" exists and is checked before any Theia version bump |
| EPL-2.0 / trademark rebranding mistakes | Branding/customization phase | Distributed build retains original copyright notices; no "Theia" name/logo reuse in Sourcerer's own branding; product-specific code is a separate module |
| Stripping IDE features prematurely | Branding/customization phase | Unused views/contributions are hidden via preferences, not removed as dependencies, unless explicitly confirmed permanent |

## Sources

- [Developing an application against a fork of Theia — GitHub Discussion #9862](https://github.com/eclipse-theia/theia/discussions/9862) — MEDIUM confidence
- [Theia API Management policy — `doc/api-management.md`](https://github.com/eclipse-theia/theia/blob/master/doc/api-management.md) — LOW-tier per seam classification (single-source docs fetch), but primary official source
- [Extending/Adopting the Theia IDE (Theia Blueprint docs)](https://theia-ide.org/docs/blueprint_documentation/) — primary official source
- [Consuming Theia fixes without upgrading](https://theia-ide.org/docs/consume_theia_fixes_master/) — primary official source
- [Eclipse Public License 2.0 — full text](https://www.eclipse.org/org/documents/epl-2.0/EPL-2.0.html) and [SPDX EPL-2.0](https://spdx.org/licenses/EPL-2.0.html) — official license text
- [Eclipse Theia FAQ for Adopters and Users — EclipseSource](https://eclipsesource.com/blogs/2019/12/24/eclipse-theia-ide-faq/) — MEDIUM confidence
- [Theia electron/browser targets must be rebuilt separately — Issue #5676](https://github.com/eclipse-theia/theia/issues/5676) — MEDIUM confidence
- [nsfw N-API build error — Issue #7510](https://github.com/eclipse-theia/theia/issues/7510); [documentation prerequisite gap — Issue #7255](https://github.com/eclipse-theia/theia/issues/7255) — MEDIUM confidence
- [Theia Adopter Story: Arduino IDE 2.0 — Eclipse Foundation Blog](https://blogs.eclipse.org/post/john-kellerman/theia-adopter-story-new-arduino-ide-20) — MEDIUM confidence
- [Celebrating Eclipse Theia's Milestone: Full VS Code Extension API Compatibility](https://eclipse-foundation.blog/2023/12/18/celebrating-eclipse-theias-milestone-full-compatibility-with-vs-code-extension-api/) — MEDIUM confidence
- General Electron-on-NixOS sandbox/native-module issues (NixOS Discourse, electron/electron GitHub issues, Mic92/nix-ld) — LOW confidence, not Theia-specific, applied by analogy; verify directly against the actual NixOS host during the foundation build phase
- General Yarn/monorepo CI caching guidance (yarnpkg/berry discussions, DEV Community write-ups) — MEDIUM confidence, not Theia-specific

---
*Pitfalls research for: forking/branding Eclipse Theia (Sourcerer v3.0 Theia Foundation milestone)*
*Researched: 2026-08-18*
