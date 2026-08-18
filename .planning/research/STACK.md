# Stack Research

**Domain:** Desktop IDE/workbench framework — Eclipse Theia fork foundation

**Confidence:** MEDIUM-HIGH — exact version numbers (Node engine floor, Yarn range, @theia/core version) confirmed via direct npm registry queries (raw JSON, not LLM-summarized). Fork repo structure/package.json content (theia-ide's engines/workspaces/scripts) was read via WebFetch summarization, not raw bytes — re-verify by cloning the fork directly before relying on exact script names at implementation time.
**Researched:** 2026-08-18

## Recommended Stack

### Core Technologies

| Technology | Version | Purpose | Why Recommended |
|------------|---------|---------|-----------------|
| `eclipse-theia/theia-ide` (fork target) | pinned to Theia **1.74.x** (repo's own `package.json` version `1.74.100`, `@theia/*` deps at `1.74.1`) | The repo to fork as Sourcerer's base | This is the repo formerly named `theia-blueprint` — GitHub confirms the rename; old `theia-blueprint` URLs 302 to `theia-ide` and remain readable. It is simultaneously (a) the actual downloadable Theia IDE product and (b) the Eclipse Foundation's own documented **template for building a custom desktop product** on Theia. Forking this — not the framework repo `eclipse-theia/theia` itself — is the standard, Eclipse-endorsed path (see Fork Strategy below). |
| `@theia/core` + `@theia/*` extension packages | **1.74.1** (current `npm view @theia/core version`, verified against registry directly) | The Theia platform packages Sourcerer's product depends on (not vendors/forks) | `theia-ide`/`theia-blueprint` itself is built the "consume as npm dependency" way — its `applications/browser` and `applications/electron` workspaces declare `@theia/core`, `@theia/editor`, etc. as regular npm deps, not submodules of a forked framework. Forking `theia-ide` inherits this correctly out of the box. |
| Node.js | **>=22** (engine-pinned in `theia-ide`'s `package.json`; upstream `theia` core docs recommend Node 24 as the "default" dev version) | JS/TS runtime for build + Electron main process | Confirmed directly from `theia-ide`'s raw `package.json` (`"engines": {"node": ">=22"}`) and from `eclipse-theia/theia`'s own `doc/Developing.md` and root `package.json` (also `">=22"`). This is a hard floor — Theia's native module rebuild step (`theia rebuild`) is ABI-sensitive to the Node/Electron version pairing; older Node will not build. |
| Yarn Classic | **>=1.7.0 <2** (engine-pinned in `theia-ide`'s `package.json`) | Package manager for the Theia monorepo/workspaces | This is Yarn 1.x ("Classic"), explicitly capped below 2.0 — **not** Yarn Berry (2/3/4). Theia's build tooling (`@theia/cli`, native rebuild scripts, workspace resolution) is authored against Classic's `yarn workspaces` semantics; do not substitute pnpm, npm workspaces, or Yarn Berry without independently verifying — this is not documented as supported. |
| Electron | version pinned by `theia-ide`'s `applications/electron` workspace (tracks whatever Electron major ships with the 1.74.x product release; not independently pinned here — read it off the forked repo at scaffold time) | Desktop application shell (replaces Tauri) | `theia-ide` builds from one codebase to two targets — `browser` (served over HTTP, e.g. for Docker) and `electron` (native desktop app) — selected via the `theia.target` field in each application's `package.json` plus a matching `theia rebuild:electron` / `theia rebuild:browser` native-module rebuild step. For Sourcerer (a desktop scholar workbench, replacing the Tauri shell), **target Electron**, not browser. |

### Supporting Libraries / Tooling

| Library | Version | Purpose | When to Use |
|---------|---------|---------|-------------|
| `@theia/cli` | 1.74.1 (tracks core) | Build orchestration (`theia build`, `theia rebuild`, `theia start`, `theia download:plugins`) | Already a dev-dependency of the forked `theia-ide` — don't hand-roll a build pipeline around it. |
| Lerna | version pinned in `theia-ide`'s devDependencies | Monorepo task orchestration across `applications/*` and `theia-extensions/*` workspaces | Comes with the fork; the repo is structured as a Lerna + Yarn-workspaces monorepo. Don't introduce Nx/Turborepo on top — redundant with what's already wired. |
| `generator-theia-extension` (Yeoman) | **0.1.49** (verified via `npm view`) | Scaffolding a *new* Theia extension package (not the app shell itself) | Use once Sourcerer needs a custom extension (e.g. an applet-equivalent contribution point) rather than editing the product/branding extensions that ship with `theia-ide`. Requires Yeoman (`yo`) installed globally. Not needed for the initial fork-and-brand milestone — defer until the first real extension is built. |

## Fork Strategy: Fork `theia-ide`, Not `eclipse-theia/theia`

This is the head-on answer to "fork the framework repo vs. build a product on npm packages":

**Do not fork `eclipse-theia/theia`.** That repo is the framework itself (`packages/*`, `dev-packages/*`) — Eclipse's own composing-applications documentation is explicit that Theia's "modular architecture" and "custom extension mechanism" are the intended integration point, and a discussion thread on the core repo states plainly that the framework "is not designed to be forked." Forking the framework repo means carrying a permanent merge burden against upstream `packages/*` internals for zero product benefit.

**Do fork `eclipse-theia/theia-ide`** (the renamed `theia-blueprint`). This is the pattern Eclipse itself documents and that real adopters use (multiple public forks exist: `alvsan09/theia-blueprint`, `evolvedbinary/theia-blueprint`, etc.). It already:
- Depends on `@theia/*` as versioned npm packages (the "build a product on Theia npm packages" approach) rather than vendoring framework source.
- Ships both `applications/browser` and `applications/electron` targets, pre-wired.
- Has a dedicated `theia-extensions/product` workspace whose explicit job is branding (about dialog, welcome page) — this is where Sourcerer's name/icon/theme customization goes, not scattered edits across the tree.
- Has `theia-extensions/updater` and `theia-extensions/launcher` already solving update-mechanism and AppImage-launcher problems Sourcerer would otherwise have to build from scratch.

Net effect: "fork theia-ide" and "build a product on Theia npm packages" are **the same recommendation**, not competing options — `theia-ide` *is* the reference implementation of the npm-package-consumption approach, pre-assembled and pre-branded (as "Theia IDE"), ready to be re-branded as "Sourcerer" instead.

## Licensing: EPL-2.0 (dual EPL-2.0 / GPL-2.0-with-classpath-exception)

Verified directly against the published `@theia/core` package: `"license": "EPL-2.0 OR GPL-2.0-only WITH Classpath-exception-2.0"`.

- **EPL-2.0 is weak copyleft**, not strong copyleft (unlike GPL/AGPL). The copyleft obligation attaches to *modifications of EPL-licensed files themselves* — if you modify a file inside `@theia/core` or another `@theia/*` package, that modified file must stay EPL-2.0-licensed (or be contributed back). Code that merely *depends on* or *extends* Theia through its extension/contribution-point mechanism, kept in separate files, is not a derivative work and carries no EPL obligation.
- **Practical implication for the fork:** as long as Sourcerer's product-specific code lives in `theia-extensions/product` (or new sibling extension packages) rather than patching files inside `node_modules/@theia/*` or a vendored copy of the framework, Sourcerer's own code does not have to be EPL-licensed.
- **Open question this milestone does not resolve:** Sourcerer's `Key Decisions` table already commits to **AGPL** for its own repo (D-P1, decided 2026-08-03, "before any public repo / Store submission"). AGPL (strong copyleft, network-use clause) applied to a *repository whose base layer is EPL-2.0/GPL-2.0-classpath code* needs its own compatibility check before any public release — EPL-2.0 and GPL/AGPL have known one-directional combination rules. Flag this for a licensing-focused review pass before Sourcerer ships publicly; it does not block local dev-only forking now.

## Release Cadence and Current Line

- Theia ships **monthly minor releases** (verified: 1.70 → Apr 16 2026, 1.73 → Jul 7 2026, 1.74 → Aug 11 2026 — all via eclipsesource.com's official "News and Noteworthy" series, roughly one release every 4-5 weeks).
- On top of monthly releases, the Eclipse Foundation ships a **quarterly "Community Release"** (introduced 2022, reaffirmed on Theia's 5th anniversary) aimed at adopters who want a slower-moving, more stability-tested train instead of tracking every monthly drop.
- As of this research date (2026-08-18), **1.74 is the latest monthly release** (Aug 11 2026) and is what `theia-ide`'s current `package.json` (`1.74.100`) is pinned to.
- **Recommendation:** fork at the current `theia-ide` `master` (tracking 1.74.x) rather than pinning to a Community Release branch — Sourcerer is doing active foundational work, not running a stable production deployment yet; take the monthly-cadence source. Revisit the Community Release track once Sourcerer has shipped and wants a lower-churn update cadence.

## Installation

```bash
# Fork eclipse-theia/theia-ide on GitHub, then clone the fork
git clone <your-fork-url> sourcerer-theia
cd sourcerer-theia

# Node >=22 required (24 recommended) — verify with:
node --version

# Yarn Classic 1.7+ (NOT Yarn 2/3/4/Berry) — verify with:
yarn --version   # must report 1.x

# Install workspace deps (Lerna + Yarn workspaces, pre-wired)
yarn

# Development build (faster, unminified)
yarn build:dev

# Pull in the default plugin set the product ships with
yarn download:plugins

# Rebuild native modules for the target you're running —
# switching targets requires re-running this; browser and electron
# native builds are NOT interchangeable
yarn rebuild:browser    # for applications/browser
yarn rebuild:electron   # for applications/electron

# Run the Electron desktop app (the target Sourcerer wants)
cd applications/electron
yarn start
```

## Alternatives Considered

| Recommended | Alternative | When to Use Alternative |
|-------------|-------------|-------------------------|
| Fork `eclipse-theia/theia-ide` | Fork `eclipse-theia/theia` (the framework repo) | Only if Sourcerer needed to modify Theia's own core packages directly and planned to upstream those changes as a framework contribution — not the case here. The framework repo is explicitly not designed to be forked as a product base. |
| Fork `eclipse-theia/theia-ide` | Start from scratch via `theia-ide.org/docs/composing_applications/` (empty app, hand-add `@theia/*` deps one at a time) | If Sourcerer wanted a minimal app with only hand-picked extensions and zero pre-existing branding/updater/launcher scaffolding. Slower start, but avoids inheriting any of `theia-ide`'s existing product decisions (its default plugin set, its updater mechanism). Not recommended here — the milestone goal is "working, rebranded fork... as fast as possible," and `theia-ide` gets there in one clone + rebrand pass. |
| Electron target | Browser target (served via Docker/HTTP, accessed through a browser tab) | If Sourcerer's long-term "one server, two clients" vision (noted in PROJECT.md) pivots to a browser-first or hosted-multi-user delivery model. Out of scope for this milestone — v3.0's explicit goal is a local desktop app replacing the Tauri shell. |
| Yarn Classic (as pinned) | pnpm / npm workspaces / Yarn Berry | Only after independently verifying Theia's build tooling (native rebuild scripts, `@theia/cli` internals) works under the alternative — this is undocumented territory upstream and each community example targets Yarn Classic. Do not switch package managers speculatively. |

## What NOT to Use

| Avoid | Why | Use Instead |
|-------|-----|-------------|
| Forking `eclipse-theia/theia` directly | Framework repo, not a product template; permanent upstream-merge burden; a maintainer discussion on the repo itself states it "is not designed to be forked." | Fork `eclipse-theia/theia-ide`. |
| Node <22 | Hard engine floor in both `eclipse-theia/theia` and `theia-ide`'s `package.json` (`>=22`); native module ABI mismatches will break the build below this. | Node 22 LTS or newer; Node 24 if you want the version upstream devs currently default to. |
| Yarn Berry (2/3/4) or npm as the workspace manager, without verification | Not what the fork's `engines`/tooling is authored against (`yarn: ">=1.7.0 <2"`); untested combination. | Yarn Classic 1.x, matching the pinned range. |
| Patching `@theia/*` files directly inside `node_modules` (or vendoring a modified copy of the framework into the fork) | Triggers EPL-2.0's copyleft obligation on those specific modified files, and creates silent drift from upstream that breaks the monthly-update path. | Extend via Theia's contribution-point/extension mechanism in `theia-extensions/*` — the same mechanism the product's own branding/updater/launcher extensions already use. |
| Treating `theia-blueprint` and `theia-ide` as two different repos to choose between | They are the same repo — GitHub confirms the rename with a working redirect; `theia-blueprint` references in older blog posts/forks point at what is now `eclipse-theia/theia-ide`. | Use `eclipse-theia/theia-ide` as the canonical name going forward; note the old name only when it appears in dated documentation or fork names you find in the wild. |

## Stack Patterns by Variant

**If Sourcerer needs a genuinely new extension (an applet-equivalent contribution) rather than just rebranding:**
- Scaffold it with `generator-theia-extension` (`yo theia-extension`) into its own `theia-extensions/<name>` workspace inside the fork.
- Because this keeps the new code in a separate, non-EPL-obligated file tree and matches how the fork's own `product`/`updater`/`launcher` extensions are already structured.

**If Sourcerer later needs a browser-accessible surface alongside the desktop app (matches the "one server, two clients" long-term note in PROJECT.md):**
- Reuse the same fork's `applications/browser` workspace (already present, already dependency-compatible with `applications/electron`) rather than standing up a second unrelated web app.
- Because Theia's whole design point is one core + multiple frontend targets from the same extension set — this is what the framework is for.

## Version Compatibility

| Package A | Compatible With | Notes |
|-----------|-----------------|-------|
| `theia-ide` fork at `1.74.100` | `@theia/*` packages at `1.74.1` | Verified directly against the fork's `package.json` and the live npm registry entry for `@theia/core@latest` (`1.74.1`) — these track together; do not bump individual `@theia/*` deps ahead of the fork's pinned line without testing, Theia's packages are released in lockstep as a monorepo. |
| Node `>=22` | Electron's bundled Node/V8 (whatever major ships with the fork's pinned Electron version) | Theia's native-module rebuild step (`theia rebuild:electron`) exists specifically to reconcile the dev-machine Node ABI with Electron's bundled Node ABI — read the exact Electron version off `applications/electron/package.json` in the fork before assuming a specific pairing. |
| Yarn `>=1.7.0 <2` | Lerna (version pinned in the fork) | Both are inherited together from the fork; don't upgrade Lerna independently of what `theia-ide`'s own maintainers ship, since Lerna's yarn-workspaces integration is version-sensitive. |

## Sources

- [github.com/eclipse-theia/theia-ide](https://github.com/eclipse-theia/theia-ide) — HIGH confidence, primary source: repo identity/rename confirmation, "template for building your own product" framing, live at research date
- Raw `package.json` fetched directly from `eclipse-theia/theia-blueprint` (redirects to `theia-ide`) master branch — HIGH confidence: version `1.74.100`, `engines: {node: ">=22", yarn: ">=1.7.0 <2"}`, workspace layout (`applications/*`, `theia-extensions/*`)
- npm registry direct query (`registry.npmjs.org/@theia/core/latest`, `/@theia/cli/latest`, `/generator-theia-extension/latest`) — HIGH confidence, most authoritative version source: `@theia/core@1.74.1`, license `EPL-2.0 OR GPL-2.0-only WITH Classpath-exception-2.0`, `generator-theia-extension@0.1.49`
- Raw `doc/Developing.md` and root `package.json` fetched from `eclipse-theia/theia` master branch — HIGH confidence: Node `>=22` (24 recommended default), Git 2.11+, Python3/node-gyp, VS2022 Build Tools on Windows
- [theia-ide.org/docs/composing_applications/](https://theia-ide.org/docs/composing_applications/) — MEDIUM confidence (WebFetch-summarized, not independently re-verified line-by-line): explicit "NOT a fork of VS Code" framing, npm-dependency-based app composition pattern, sample `browser-app`/`electron-app` package.json shape (some version numbers in this doc, e.g. an older "Node 18+" line, are stale relative to the fork's actual current `>=22` pin — trust the fork's own `package.json` over this doc's prose)
- [github.com/eclipse-theia/theia/discussions/14502](https://github.com/eclipse-theia/theia/discussions/14502) ("Theia license not really attractive for contributors") — MEDIUM confidence: community discussion confirming EPL-2.0's weak-copyleft framing and the "not designed to be forked" characterization of the core framework
- [eclipsesource.com Eclipse Theia release blog series](https://eclipsesource.com/blogs/2026/08/11/eclipse-theia-1-74-release-news-and-noteworthy/) (1.70 Apr 2026, 1.73 Jul 2026, 1.74 Aug 2026 posts) — HIGH confidence: monthly release cadence and dates
- [newsroom.eclipse.org — 5th Anniversary / Community Release announcement](https://newsroom.eclipse.org/news/announcements/5th-anniversary-eclipse-theia-eclipse-foundation-introduces-new-software-release) — HIGH confidence: quarterly Community Release track, introduced 2022
- [eclipse.org/legal/epl-2.0/](https://www.eclipse.org/legal/epl-2.0/) and [FOSSA EPL blog](https://fossa.com/blog/open-source-software-licenses-101-eclipse-public-license/) — HIGH confidence (official license text + reputable third-party legal summary): weak-copyleft mechanics
- `github.com/eclipse-theia/generator-theia-extension` — HIGH confidence: Yeoman generator purpose and usage (`yo theia-extension`), confirmed against npm registry version
- Project's own `.planning/PROJECT.md` — HIGH confidence, primary source: v3.0 milestone goal, AGPL decision (D-P1) whose interaction with EPL-2.0 is flagged above as unresolved

---
*Stack research for: Eclipse Theia fork foundation (Sourcerer v3.0 milestone)*
*Researched: 2026-08-18*
