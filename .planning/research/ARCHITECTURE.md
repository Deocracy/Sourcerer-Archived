# Architecture Research

**Domain:** Branded desktop IDE/workbench product built as an Eclipse Theia fork
**Researched:** 2026-08-18
**Confidence:** HIGH (official Theia docs + `theia-blueprint`/`theia-ide` reference repos, cross-checked across multiple sources)

## Standard Architecture

### System Overview

Theia is a **three-process** system, not two. This is the single most important thing to internalize coming from the Tauri shell (which was frontend + one Rust backend + one Pi sidecar): Theia's process split is frontend / backend / plugin-host, and Electron adds a fourth (main) that is thin and mostly about window/OS chrome.

```
┌───────────────────────────────────────────────────────────────────────┐
│  ELECTRON MAIN (target: electron only)                                │
│  Window creation, native menus, app lifecycle, spawns the backend     │
│  as a child process. Thin — not where product logic lives.            │
└───────────────────────────────┬─────────────────────────────────────-┘
                                 │ spawns
┌────────────────────────────────────────────────────────────────────────┐
│  FRONTEND  (Electron renderer, OR plain browser tab for target=browser) │
│  React-free — Theia's own widget/Lumino-based UI (Monaco editor,        │
│  dockable panels, tree views, status bar). Runs the Inversify DI        │
│  container that Theia extensions register into.                        │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────────────────┐       │
│  │ Monaco   │ │ Dock/    │ │ Custom   │ │ Webviews (VS Code    │       │
│  │ editor   │ │ Layout   │ │ Widgets  │ │ ext. UI renders here)│       │
│  └──────────┘ └──────────┘ └──────────┘ └──────────────────────┘       │
└──────────────────┬────────────────────────────────────┬────────────────┘
                    │ JSON-RPC over WebSocket             │ JSON-RPC (proxied)
                    ▼                                     ▼
┌────────────────────────────────────┐   ┌──────────────────────────────┐
│  BACKEND (Node.js process)          │──▶│  PLUGIN HOST (Node.js,        │
│  Filesystem, workspace, terminal,   │   │  separate process, spawned    │
│  Git, LSP servers, DAP, task        │   │  by the backend)               │
│  runner, backend halves of Theia    │   │  Runs VS Code extensions       │
│  extensions.                        │   │  (.vsix) via the VS Code API   │
└──────────────────────────────────────┘   │  translation/compat layer.    │
                                            └──────────────────────────────┘
```

Key facts that shape the fork:
- **Even in the Electron target, the backend is a separate Node.js process**, not in-process with the renderer — the frontend talks to it over a local WebSocket exactly as the browser target does. This is why Theia apps run near-identically as a desktop app or a browser-hosted server: the split is the same, only what hosts the frontend window changes.
- **VS Code extensions never run in the frontend or main backend process.** They run in a dedicated "plugin host" Node process the backend spawns, isolated via RPC — this is literally VS Code's own extension-host architecture, reused wholesale.
- Theia builds directly on VS Code/Code-OSS pieces: Monaco, the LSP and DAP clients, and (via the plugin host) the VS Code extension API itself. This is why "fork Theia" gets you a mature workbench for free rather than reinventing docking/editor/LSP plumbing — directly the value case in `.planning/PROJECT.md`'s pivot decision.

### Component Responsibilities

| Component | Responsibility | Typical Implementation |
|-----------|----------------|-------------------------|
| Electron main | Window lifecycle, native menu, spawns backend, packaging/update hooks | `@theia/electron`, `electron-builder` config in `electron-app/package.json` |
| Frontend | UI shell: dock layout, widgets, Monaco, command/menu/keybinding registries | Theia's own Lumino-based widget system + Inversify DI container (`*-frontend-module.ts` per extension) |
| Backend | Workspace/file access, LSP/DAP servers, terminals, Git, backend halves of contributions | Node.js, Express + `ws`, Inversify DI container (`*-backend-module.ts` per extension) |
| Plugin host | Runs VS Code (`.vsix`) extensions in isolation, exposes the VS Code extension API | Separate Node process; RPC-proxied to frontend/backend; webviews render back in the frontend iframe-like sandbox |
| Theia extension | First-class, compile-time npm package that extends *Theia's own* APIs — can reach the frontend widget tree, menus, DI bindings directly, not sandboxed | `@theia/core` `ContainerModule`, `bind()`/`rebind()` calls in a `*-frontend-module.ts`/`*-backend-module.ts` |
| VS Code plugin | Runtime-installed `.vsix` package using the VS Code extension API (`vscode.*` namespace) | Standard VS Code extension, works largely unmodified via Theia's compatibility/translation layer |

## Extension Mechanisms — Theia Extensions vs. VS Code Plugins

This is the load-bearing distinction for the whole fork strategy, so state it plainly.

### Theia extensions (Inversify DI, compile-time)

**What:** An npm package (or packages) added as a dependency of the app's `package.json`, discovered at build time by `@theia/cli`/`@theia/application-manager`, and wired into Theia's Inversify IoC container via a `ContainerModule`. A Theia extension can `bind()` a brand-new service/widget or `rebind()` an existing core Theia binding (e.g. swap the default About dialog, override the getting-started/welcome widget, change the window title service).

**Capabilities (full access, no sandbox):**
- Contribute new dockable widgets/views directly into the Lumino layout
- Rebind/replace *any* existing Theia core service or component
- Add commands, menus, keybindings, preferences, status bar items
- Run backend-side code in the same trust domain as Theia core (file access, spawn processes, open network sockets)
- Cross-cut frontend + backend in the same conceptual extension (paired `*-frontend-module.ts` + `*-backend-module.ts`, connected via an RPC proxy Theia generates for you)

**Cost:** Compiled into the app at build time. Adding/changing one means a rebuild (`theia build`), not a runtime install. Requires learning Inversify's DI vocabulary (`bind`, `rebind`, `toDynamicValue`, `inSingletonScope`, `@injectable`, `@inject`).

**When to use for Sourcerer:** Anything that needs to reach *into* Theia's own chrome or sit as a first-class citizen of the shell — product branding (name/icon/title/about/welcome), the eventual Dashboard Assistant panel as a persistent, non-closable dockable widget, and any "applet" that needs host-API-style privileged access (storage, AI seam) analogous to what `host.ai()` gave applets in the old Tauri shell. This is the direct architectural descendant of that `host` API concept — a Theia extension *is* how you'd re-implement `host.storage`/`host.ai()`/`host.open` as real Theia services other extensions inject.

### VS Code extensions (plugin host, runtime-loaded)

**What:** A standard `.vsix` VS Code extension, installed at runtime (from Open VSX or a private registry, or sideloaded), running in the isolated plugin-host Node process. Theia ships a compatibility/translation layer that adapts the `vscode.*` API surface (including index-base differences — VS Code positions are 1-indexed in some spots vs Theia's 0-indexed internals) so most unmodified VS Code extensions just work.

**Capabilities (sandboxed, API-surface-limited):**
- Whatever the standard VS Code extension API exposes: commands, language features, tree views, custom editors, **webviews** for custom UI (rendered back in the frontend inside an isolated iframe-like boundary)
- No direct access to Theia's internal DI container or core services outside the documented `vscode.*` API

**Cost:** Cannot rebind or directly manipulate Theia's own UI shell/services — a VS Code extension's UI contributions render inside VS Code-API-shaped containers (tree view, webview panel), not as arbitrary first-class Theia widgets.

**When to use for Sourcerer:** Anything you want to source from the existing VS Code/Open VSX ecosystem for free (language servers, linters, formatters, Git tooling, Markdown preview) rather than write yourself, or any Sourcerer-authored feature that's naturally expressible as "a VS Code extension" and where you want it distributable/testable independent of the Theia shell (portable to real VS Code too). Not the right model for chrome-level branding or for anything that needs to be a permanent, privileged part of the shell (the assistant panel, the future applet host contract) — those want Theia extensions.

**Rule of thumb:** *Branding and shell-privileged infrastructure → Theia extension. Reusable/ecosystem-shaped features → VS Code extension, only reach for it when the feature is genuinely extension-shaped and doesn't need shell-level privilege.*

## Recommended Project Structure

Modeled directly on `eclipse-theia/theia-blueprint` (the Eclipse Foundation's own reference template for exactly this "fork Theia into a branded desktop product" scenario) plus `theia-ide` (the flagship, further-along example), both Yarn-workspace/Lerna monorepos:

```
sourcerer-theia/                        # the fork's monorepo root
├── package.json                        # workspaces root, shared devDeps, lerna/yarn config
├── lerna.json                          # (or plain yarn workspaces — theia-blueprint uses lerna)
├── applications/                       # (theia-blueprint naming; theia-ide calls this apps/)
│   ├── browser-app/
│   │   ├── package.json                # "theia": { "target": "browser", "frontend": {...} }
│   │   └── src-gen/, lib/              # generated by @theia/cli, gitignored
│   └── electron-app/
│       ├── package.json                # "theia": { "target": "electron", ... } + electron-builder config
│       └── src-gen/, lib/
├── theia-extensions/                   # Sourcerer's own Theia extensions (compile-time, DI-bound)
│   ├── sourcerer-product/              # branding: name/icon/title/about-dialog/welcome-widget
│   │   ├── src/browser/
│   │   │   ├── sourcerer-frontend-module.ts   # ContainerModule: rebind AboutDialog, GettingStartedWidget
│   │   │   └── sourcerer-about-dialog.tsx
│   │   └── package.json
│   ├── sourcerer-launcher/             # (optional, mirrors theia-blueprint's AppImage/launcher ext)
│   ├── sourcerer-updater/              # (optional, later milestone — electron-updater wiring)
│   └── sourcerer-assistant/            # future: Dashboard Assistant as a privileged dockable widget
├── patches/                            # any upstream @theia/* patch-package overrides, kept minimal
├── configs/                            # shared tsconfig/eslint referenced by all packages
├── scripts/                            # repo-level build/release scripts
└── doc/                                # dev workflow docs (this milestone's deliverable)
```

### Structure Rationale

- **`applications/`, not a single app root:** Theia's own tooling (`@theia/cli`) expects one `package.json` per *target* (browser vs electron) because the generated bundle, `src-gen/`, and dependency set differ per target. Keeping both under one workspace lets them share the same `theia-extensions/*` packages without duplication.
- **`theia-extensions/`, separate from `applications/`:** extensions are the actual product code; applications are thin assembly points that just list which extensions + which `@theia/*` core packages to pull in via `dependencies` and the `theia.target` field. This mirrors npm's own separation of "library" vs "app that depends on the library" and is exactly how `theia-blueprint` and `theia-ide` are laid out.
- **One extension package per concern (`sourcerer-product`, `sourcerer-assistant`, ...), not one giant extension:** matches Theia's own convention (`@theia/git`, `@theia/debug`, `@theia/terminal` are all separate packages) and keeps the DI `ContainerModule` per package small and reviewable.
- **`patches/`:** flagged explicitly because forking a large upstream project tempts editing `node_modules`/vendored core directly — don't. Any unavoidable upstream tweak goes through `patch-package` (or a proper upstream PR) so `theia` version bumps stay tractable. This is the fork-maintenance discipline the "incrementally extend, don't diverge" goal in `.planning/PROJECT.md` depends on.

## Architectural Patterns

### Pattern 1: Branding via `rebind()` in a product extension

**What:** Theia core binds default implementations (About dialog, Getting Started/welcome widget, window title service, product icons) behind DI tokens. A product extension's `ContainerModule` calls `rebind(AboutDialog).to(SourcererAboutDialog)` (etc.) instead of adding new bindings — you're *replacing* what's already there, not adding alongside it.
**When to use:** All Sourcerer branding work this milestone (name, window title, theme direction, about/welcome content).
**Trade-offs:** Requires knowing which core token to target (found by reading `@theia/core`'s own module files) — not always obvious from docs alone; DeepWiki/source-reading is often the fastest path.

**Example** (mirrors `theia-blueprint-product`'s `theia-blueprint-frontend-module.ts`):
```typescript
import { ContainerModule } from '@theia/core/shared/inversify';
import { AboutDialog } from '@theia/core/lib/browser/about-dialog';
import { SourcererAboutDialog } from './sourcerer-about-dialog';
import { GettingStartedWidget } from '@theia/getting-started/lib/browser/getting-started-widget';
import { SourcererGettingStartedWidget } from './sourcerer-getting-started-widget';

export default new ContainerModule(bind => {
    bind(SourcererAboutDialog).toSelf().inSingletonScope();
    bind(AboutDialog).toService(SourcererAboutDialog); // or rebind() if AboutDialog is already bound elsewhere

    bind(SourcererGettingStartedWidget).toSelf();
    bind(GettingStartedWidget).toService(SourcererGettingStartedWidget);
});
```

### Pattern 2: Contributing a new dockable widget (`AbstractViewContribution` + `WidgetFactory`)

**What:** To add a brand-new panel (not replace an existing one) — the mechanism later milestones will use for applets and the Dashboard Assistant — bind a `WidgetFactory` that knows how to construct the widget, then bind a `WidgetContribution`/`AbstractViewContribution` subclass that registers a command + menu entry to open it and tells Theia which shell area (`left`, `right`, `main`, `bottom`) it docks to by default.
**When to use:** Any new first-class panel that should behave like Theia's own Explorer/Outline/Problems views — draggable, dockable, persisted in the layout, reachable from the View menu.
**Trade-offs:** More ceremony than a VS Code webview, but the payoff is a panel that's a true peer of Theia's own views (participates in layout persistence, keyboard nav, focus tracking) rather than an iframe-sandboxed guest.

**Example (shape only):**
```typescript
@injectable()
export class SourcererAssistantWidget extends ReactWidget {
    static readonly ID = 'sourcerer.assistant';
    static readonly LABEL = 'Assistant';
}

@injectable()
export class SourcererAssistantContribution extends AbstractViewContribution<SourcererAssistantWidget> {
    constructor() {
        super({
            widgetId: SourcererAssistantWidget.ID,
            widgetName: SourcererAssistantWidget.LABEL,
            defaultWidgetOptions: { area: 'right' },
            toggleCommandId: 'sourcerer.assistant.toggle',
        });
    }
}

export default new ContainerModule(bind => {
    bind(SourcererAssistantWidget).toSelf();
    bind(WidgetFactory).toDynamicValue(ctx => ({
        id: SourcererAssistantWidget.ID,
        createWidget: () => ctx.container.get(SourcererAssistantWidget),
    })).inSingletonScope();
    bind(SourcererAssistantContribution).toSelf().inSingletonScope();
    bind(FrontendApplicationContribution).toService(SourcererAssistantContribution);
    bind(CommandContribution).toService(SourcererAssistantContribution);
    bind(MenuContribution).toService(SourcererAssistantContribution);
});
```

### Pattern 3: `package.json`-driven application composition

**What:** An `applications/*/package.json` doesn't contain product *code* — it's a manifest. `dependencies` lists which `@theia/*` core packages and which `theia-extensions/*` packages to include; a `theia` config block sets `target` (`browser` | `electron`), `applicationName`, `defaultTheme`, `defaultIconTheme`, and other frontend config. `@theia/cli`'s `theia build`/`theia start` reads this to generate the webpack bundle (`src-gen/`) and Electron main entrypoint.
**When to use:** This is *the* mechanism for "composing a product from extensions" — it's how the whole fork stays declarative instead of forking Theia's own source tree.
**Trade-offs:** None really — this is the intended, supported way to build a Theia product; the alternative (forking `@theia/core` itself) is explicitly the anti-pattern below.

**Example** (shape, electron target):
```jsonc
{
  "name": "sourcerer-electron-app",
  "private": true,
  "dependencies": {
    "@theia/core": "latest",
    "@theia/editor": "latest",
    "@theia/filesystem": "latest",
    "@theia/workspace": "latest",
    "@theia/monaco": "latest",
    "@theia/electron": "latest",
    "@theia/plugin-ext-vscode": "latest",
    "sourcerer-product": "0.0.0"
  },
  "theia": {
    "target": "electron",
    "frontend": {
      "config": {
        "applicationName": "Sourcerer",
        "defaultTheme": "dark",
        "defaultIconTheme": "vs-seti"
      }
    }
  }
}
```

## Data Flow

### Request Flow (frontend → backend service call)

```
[User action in a Theia widget]
    ↓
[Frontend service (injected via DI)] → JSON-RPC proxy (auto-generated by Theia's `connectionHandler`)
    ↓  (WebSocket)
[Backend service implementation] → [Node.js: fs / child_process / LSP server / etc.]
    ↓
[Response] ← proxy ← [Frontend service resolves promise, updates widget/state]
```

### VS Code extension flow (plugin host)

```
[VS Code extension activates in Plugin Host process]
    ↓ vscode.* API call
[Theia's VS Code-API compatibility layer] → translates to Theia's internal services (over RPC)
    ↓
[Theia core service executes] (file read, diagnostics, etc.)
    ↓
[Result translated back to VS Code API shape] → returned to the extension
```
Webview-based UI from a VS Code extension renders in the **frontend** process (sandboxed), not the plugin host — the plugin host only runs the extension's logic.

### Key Data Flows

1. **Branding/config:** `applications/*/package.json`'s `theia` block is read at *build* time by `@theia/cli`, baked into the generated frontend bundle — not runtime-configurable without a rebuild.
2. **Extension registration:** every `theia-extensions/*` package's `frontendModule`/`backendModule` entry point (declared in that package's own `package.json` under a `theiaExtensions` field) gets discovered and merged into the app's single Inversify container at bundle-generation time.
3. **Persisted layout/state:** Theia persists workspace layout (which widgets are open/where) itself, analogous to the old shell's `workspace.json` — this is a carry-over-review topic for a later milestone (does Sourcerer's existing layout-persistence design map onto Theia's, or does Theia's own mechanism simply replace it).

## Scaling Considerations

Not a user-count scaling story (single-user desktop app) — the axis that matters here is **fork divergence over time**, i.e. how hard it stays to pull upstream Theia updates as Sourcerer's own extensions grow.

| Stage | Architecture Adjustments |
|-------|--------------------------|
| Foundation (this milestone) | 1 product extension (branding only), fork builds and runs locally, no custom widgets yet |
| Early applet/assistant work | Add `theia-extensions/sourcerer-assistant` etc. as new sibling packages — never touch `@theia/*` source |
| Many custom applets | Consider whether some applets are better shaped as VS Code extensions (sandboxed, independently distributable) vs Theia extensions (privileged) on a case-by-case basis, per the rule of thumb above |

### Scaling Priorities

1. **First risk: upstream drift.** Every custom rebind/override in `theia-extensions/*` is a point that can break on a Theia version bump if the core token it targets gets renamed/restructured. Keep the `sourcerer-product` extension's surface area small (branding only) and avoid deep `rebind()` chains into internals not documented as extension points.
2. **Second risk: patch-package accumulation.** If any upstream Theia behavior can't be reached through the documented DI/contribution-point surface and requires a source patch, track it explicitly (a `patches/` dir with named, dated patch files) rather than letting ad hoc edits creep into `node_modules`.

## Anti-Patterns

### Anti-Pattern 1: Editing `@theia/*` package source directly instead of extending it

**What people do:** Clone `eclipse-theia/theia` itself and hand-edit core packages to get custom behavior, because it's the fastest way to see a change.
**Why it's wrong:** Every future `theia` version bump now requires manually re-applying every hand-edit; this is exactly the maintenance trap "fork Theia" is supposed to avoid compared to "hand-roll a shell." The whole value proposition in the milestone's Key Decision (mature workbench platform, maintained upstream) evaporates if the fork forks Theia's *source*, not just *composes* it.
**Do this instead:** Depend on published `@theia/*` npm packages as ordinary dependencies; extend via `theia-extensions/*` (rebind/bind through Inversify) or `patches/` (tracked, minimal, named patch-package diffs) only when there's truly no extension point.

### Anti-Pattern 2: Building the Dashboard Assistant / future applets as VS Code webviews when they need shell privilege

**What people do:** Default to "VS Code extension + webview" for every custom panel because it's the more familiar/portable model from prior VS Code extension experience.
**Why it's wrong:** Webviews are intentionally sandboxed — no direct DI container access, no rebinding core services, no privileged host-API-equivalent surface. The old Tauri shell's `host.ai()`/`host.storage` contract needs a privileged seam; a VS Code webview extension can't be that seam without punching a large, ad hoc RPC hole through the plugin-host boundary.
**Do this instead:** Model the Dashboard Assistant (and any future host-API-style applet contract) as a Theia extension contributing a real dockable widget (Pattern 2), with its own injectable service(s) other Theia extensions can depend on — the direct architectural equivalent of the old `host` object.

## Integration Points

### External Services

| Service | Integration Pattern | Notes |
|---------|---------------------|-------|
| Pi assistant sidecar (carried over from v1.0, review deferred) | Would become a **backend-side** Theia extension service (Node.js, same trust domain as filesystem/LSP access), exposed to the frontend via Theia's RPC proxy generation — analogous to how the old Rust Tauri command exposed `host.ai()` | Not decided this milestone; flagged for the deferred carry-over review |
| Databasise engine (Cozo+LightRAG, REST + MCP) | Backend-side extension calling out over HTTP/MCP, same shape as any other backend service | Integration mode still explicitly undecided per `PROJECT.md` |
| Open VSX / VS Code extension registry | Standard Theia `@theia/plugin-ext-vscode` + a configured registry URL | Only relevant once Sourcerer wants to let users install third-party VS Code extensions; not required for the foundation milestone |

### Internal Boundaries

| Boundary | Communication | Notes |
|----------|---------------|-------|
| Frontend ↔ Backend | JSON-RPC over WebSocket, auto-proxied by Theia's DI container (declare an interface, Theia generates the proxy) | Same pattern for every Theia/backend service pair — no manual IPC wiring needed, unlike the old Tauri `invoke()` command-by-command surface |
| Backend ↔ Plugin host | RPC, managed entirely by Theia/VS Code's extension-host protocol | Not something product code touches directly unless writing Theia's own plugin-API compatibility shims (out of scope) |
| Theia extension (frontend) ↔ Theia extension (frontend) | Direct Inversify injection (`@inject(SomeService)`) within the same container | This is the applet-to-applet / assistant-to-host equivalent boundary going forward |
| Fork's own `theia-extensions/*` ↔ upstream `@theia/*` | `bind()`/`rebind()` only, package dependency, never source edits | See Anti-Pattern 1 |

## Suggested Build Order for the Foundation Milestone

Ordered by hard dependency, not by importance — each step needs the previous one buildable/running before it's testable.

1. **Repo reorg first, isolated from the fork.** Freeze `Sourcerer-v0/` (Tauri shell + Container Platform work) exactly as `PROJECT.md` already scopes it, *before* introducing the Theia fork's own tree — avoids the two efforts' build tooling (Rust/Cargo/Vite vs Yarn/Lerna/webpack) fighting over root-level config files.
2. **Scaffold from `theia-blueprint`, not from a blank Theia example.** It's the Eclipse Foundation's own reference for "this exact scenario" (branded desktop product fork) — cloning it and stripping down is far less work than assembling `applications/`+`theia-extensions/` from scratch, and its structure is the one this document's "Recommended Project Structure" mirrors.
3. **Get the unmodified fork building and launching locally first** (`electron-app` target) — prove the Yarn workspace/Lerna build, `@theia/cli` generation step, and `electron-builder` packaging work on this machine *before* any Sourcerer-specific code exists. This is the actual "kill question" of the milestone: does the fork build and run at all here.
4. **Add `theia-extensions/sourcerer-product`** and wire branding: `applicationName`/window title via `applications/*/package.json`'s `theia.frontend.config`, then rebind the About dialog and Getting Started/welcome widget (Pattern 1). Verify visually — this is the smallest possible "our fork, not vanilla Theia" proof.
5. **Document the dev workflow** (equivalent of the old shell's `tauri dev` vs `cargo run` landmine notes) — which commands actually work reliably here for `theia build` / `theia start` / electron packaging, captured while doing steps 3–4, not reconstructed later.
6. **Stop here for this milestone.** Custom widgets (assistant panel, applet host contract), Databasise integration, and VS Code-extension-vs-Theia-extension decisions per feature are explicitly out of scope — they're the next milestone(s), per `PROJECT.md`'s deferred carry-over review.

## Sources

- [Extensions and Plugins — theia-ide.org](https://theia-ide.org/docs/extensions/) — HIGH confidence, official docs: Theia extension vs. VS Code plugin conceptual split
- [Authoring Theia Extensions — theia-ide.org](https://theia-ide.org/docs/authoring_extensions/) — HIGH confidence, official docs
- [Services and Contributions — theia-ide.org](https://theia-ide.org/docs/services_and_contributions/) — HIGH confidence, official docs: DI/contribution-point model
- [Widgets — theia-ide.org](https://theia-ide.org/docs/widgets/) — HIGH confidence, official docs: `AbstractViewContribution`, `WidgetFactory`
- [Extending/Adopting the Theia IDE (Blueprint documentation) — theia-ide.org](https://theia-ide.org/docs/blueprint_documentation/) — HIGH confidence, official docs: `theia-blueprint` structure, product/launcher/updater extensions, branding customization points (About dialog, Getting Started widget)
- [eclipse-theia/theia-blueprint (GitHub)](https://github.com/eclipse-theia/theia-blueprint) — HIGH confidence, primary reference repo for this exact fork scenario
- [eclipse-theia/theia-ide (GitHub)](https://github.com/eclipse-theia/theia-ide) — HIGH confidence, flagship further-along example of the same pattern
- [Repository Structure — eclipse-theia/theia (DeepWiki)](https://deepwiki.com/eclipse-theia/theia/1.1-repository-structure) — MEDIUM confidence (third-party-generated docs, cross-checked against official docs above)
- [Extension System — eclipse-theia/theia-ide (DeepWiki)](https://deepwiki.com/eclipse-theia/theia-ide/2.2-extension-system) — MEDIUM confidence, corroborates official Inversify/DI docs
- [Product Extension — eclipse-theia/theia-ide (DeepWiki)](https://deepwiki.com/eclipse-theia/theia-ide/6.1-product-extension) — MEDIUM confidence: confirms `TheiaBlueprintGettingStartedWidget`/About dialog override location
- [VS Code Extension Support — eclipse-theia/theia (DeepWiki)](https://deepwiki.com/eclipse-theia/theia/2.4-vs-code-extension-support) — MEDIUM confidence: plugin-host process isolation, index-base translation layer
- [The Eclipse Theia IDE vs. VS Code — eclipsesource.com](https://eclipsesource.com/blogs/2019/12/06/the-eclipse-theia-ide-vs-vs-code/) — MEDIUM confidence, vendor blog but technically detailed and consistent with official docs
- [How to inversify() in Eclipse Theia — eclipsesource.com](https://eclipsesource.com/blogs/2018/11/28/how-to-inversify-in-eclipse-theia/) — MEDIUM confidence, widely-cited practical DI walkthrough
- `@theia/cli` README (npm/GitHub `dev-packages/cli`) — HIGH confidence, official: `theia.target`, `theia.frontend.config.applicationName`/`defaultTheme` package.json schema
- Project's own `.planning/PROJECT.md` — HIGH confidence, primary source for milestone scope, frozen `Sourcerer-v0/` decision, and the deferred carry-over review

---
*Architecture research for: Eclipse Theia fork (v3.0 Theia Foundation milestone), Sourcerer*
*Researched: 2026-08-18*
