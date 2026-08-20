# GUI Customizability — Verified Model (2026-08-19)

**Intent:** Power Browser's GUI must be fully customizable — every visible
element addable, changeable, removable — ideally at runtime, in JS/HTML/CSS,
without a rebuild.

This note records what is actually achievable, verified against the local
Theia v1.74.1 source by three multi-agent passes (43 agents, 72 UI components
mapped, adversarial verification on every load-bearing claim).

Companions: `power-browser-architecture.md` (the plan),
`extension-model-decisions.md` (extension/plugin split, D1–D7),
`tab-management-research/05-theia-shell-customization.md` (URL bar, rebind
seams, build cost).

---

## 0. Specimen vs subject — read this first

The investigation was done by dissecting the **Electron** Theia IDE running on
this machine. Power Browser runs Theia in **browser target**, inside a
`<browser>` element pointed at `http://127.0.0.1:<port>`.

**This does not discount the findings.** Only 4 of the 72 components mapped
were Electron-specific. Everything in §2 (Lumino boundary), §3 (cost tiers),
§5 (`.vsix` limits), §6 (the bridge design), §7 (rebind seams) and §8 (build
chain) is target-independent — those are properties of Theia's architecture,
not of its host. The specimen is how we learned *what* needs editing, *how
hard* each piece is, and *how the machinery works*.

What the target difference does change is that it deletes a category of
problem outright:

| Constraint found in Electron | Status in Power Browser |
|---|---|
| Native OS window frame, `⌄ ◇ ✕` not in the DOM | **Gone.** Firefox chrome (`power.xhtml`) draws the window. |
| Native Chromium menu bar via `BrowserWindow.setMenu()` | **Gone.** Browser target always builds the DOM menu bar. |
| `window.titleBarStyle` gates whether chrome is DOM | **Does not exist.** The preference is declared in `packages/core/src/electron-common/electron-window-preferences.ts` — Electron-only. |
| `theia-top-panel` hidden unless `titleBarStyle: 'custom'` | **Always shown.** `BrowserMenuBarContribution.onStart` → `appendMenu(shell)` adds `theia:icon` + `theia:menubar` to area `'top'` unconditionally. |
| Webview origin override is unavailable | **Available.** `THEIA_WEBVIEW_EXTERNAL_ENDPOINT` is browser-target only. |

**Consequence:** in Power Browser, 100% of Theia's chrome is DOM from the
start. There is no native-OS tier. The Electron-only advice from this session
(`titleBarStyle: custom`, `WebContentsView` for web content) is **moot** —
web content is Gecko `<browser remote="true">` via `gBrowser.addTab`, which
the architecture already specifies.

---

## 1. Three editability layers

```
┌──────────────────────────────────────────────────────────┐
│ 1. FIREFOX CHROME — power.xhtml                          │
│    Tab strip, URL bar, window controls, panels           │
│    → privileged XHTML/CSS/JS. Fully ours, by definition. │
├──────────────────────────────────────────────────────────┤
│ 2. THEIA SHELL — everything in #theia-app-shell          │
│    Lumino widget tree. All DOM in browser target.        │
│    → editable, but see §3 for what it costs.             │
├──────────────────────────────────────────────────────────┤
│ 3. WEBVIEW INTERIORS — plugin-contributed UI             │
│    Cross-origin iframes. NOT reachable. See §4.          │
└──────────────────────────────────────────────────────────┘
```

Layer 1 is free. Layer 2 is the subject of this note. Layer 3 is the only
genuine wall.

---

## 2. The Lumino boundary

Verified: `ReactWidget extends BaseWidget extends Widget` (`@lumino/widgets`),
and `ApplicationShell extends Widget`, attached to `document.body` as
`#theia-app-shell`.

**Everything visible in the shell is inside a Lumino widget.** Status bar,
activity bar, tab bars, side panels, toolbar row, dialogs — all of it.

The useful rule: **Lumino owns every container; it owns no leaf contents.**

- A status bar entry is *data* inside `StatusBarImpl` (a Lumino widget).
- A toolbar button is a React element inside `ToolbarImpl` (a Lumino widget).
- A tab label is painted by `TabBarRenderer` inside a Lumino `TabBar`.

So "is it Lumino?" is the wrong question for anything smaller than a panel.

**Lumino widgets that live outside the shell tree** (matters if you ever walk
`shell.children()` expecting completeness): dialogs attach to `document.body`
as `theia-dialog-shell`; context menu popups attach to body; secondary windows
get their own `document`; the debug hover attaches into a Monaco-owned node.

**Body-level non-widget overlays** (~12): quick input container (Monaco's
`QuickInputController`), notifications overlay (React root, not a widget),
hover popover, tooltip host, select portal, breadcrumb popups, maximize
overlay, splash, plus hidden measure/download/upload elements.

---

## 3. Cost tiers — what changing a thing actually takes

72 components of a running Theia IDE window were classified. Counts:

| Tier | Count | Meaning |
|---|---|---|
| `settings` | 24 | a preference, at runtime |
| `rebuild-only` | 26 | build-time Theia extension |
| `vsix-runtime` | 16 | a `.vsix` can do it |
| `native-os` | 4 | **Electron only — N/A in Power Browser** |
| `untouchable` | 2 | see §4 |

**Key finding: most visible chrome is already runtime-changeable.** Nearly an
entire status bar is plugin-contributed (`remoteStatus`,
`scm.change.repository`, `scm.status.*`, `problem-marker-status`,
`statusBar.collaboration`, anonymous `plugin-status-bar-item:<uuid>` entries).
Activity-bar icons are `PluginViewWidget`s. All strip colors are
`activityBar.*` theme tokens.

`rebuild-only` is concentrated in **structure**: `TheiaDockPanel`,
`ToolbarAwareTabBar`, `SideTabBar` ×2, `SidePanelHandler`, `TabBarRenderer`,
`SidebarBottomMenuWidget`, `StatusBarImpl`'s layout, `TabBarToolbar`.

That is exactly the set the unified tab strip touches.

---

## 4. The two real walls

### 4a. Webview interiors

Plugin UI (e.g. Claude Code's chat panel) renders in **two nested iframes**:
an outer host frame served from `{{uuid}}.webview.{{hostname}}`, and an inner
`#active-frame` whose HTML arrives over RPC as a *string*, gets `DOMParser`'d,
and is written in with `document.open()/write()/close()`.

Cross-origin ⇒ no `contentDocument`, no `querySelector`, no CSS injection, no
listeners, no DI. Only `postMessage`. Theme integration works because
`WebviewThemeDataProvider` pushes `--vscode-*` CSS variables through that
channel — cooperation, not reach-in.

**Browser-target escape hatch exists:** setting
`THEIA_WEBVIEW_EXTERNAL_ENDPOINT` to `{{hostname}}` serves webviews
same-origin (`serveSameOrigin = true`). Theia emits deliberate frontend and
backend security warnings when the pattern is non-default.

**Recommendation: do not take it.** In a *browser*, collapsing the boundary
between arbitrary plugin/web content and the IDE origin is the wrong trade.
Keep webviews cross-origin; treat their interiors as permanently opaque.

### 4b. Built-in codicon glyphs

Theia implements file icon themes but **not** `productIconTheme` (grep finds
nothing but an nls metadata blob). Every built-in glyph is hard-coded at its
widget's `title.iconClass`. Changing the gear or the beaker means patching the
owning package. Plugin-contributed container icons are exempt — their SVG is
masked in from the `.vsix`.

---

## 5. `.vsix` limits — structural, not configurable

Verified and worth writing down because it constrains any "let users customize
it with an extension" plan:

- Plugin code runs in a **forked Node child process** (binary pipe on stdio
  fd 4) or a Web Worker. Never the frontend page.
- Only `$`-prefixed methods on registered proxies cross. 44 `*_MAIN` proxies
  total, enumerated in `main-context.ts`.
- **No `.vsix` can touch the DOM, Lumino, the DI container, or
  `ApplicationShell` — in any configuration.** No proxy accepts markup, a DOM
  node, or a React element; a live object cannot survive byte-buffer
  serialization.
- Cannot reach `MAIN_MENU_BAR` at all — 34 implemented contribution points,
  none maps to the menu bar. `file/newFile` looks like a hook but is
  `['file','newFile']`, feeding the New File quick-pick.
- **Can** add activity-bar icons (`contributes.viewsContainers`), status bar
  entries, tree views, webview views.
- **Sharp edge:** a plugin can *hijack* any built-in status bar entry by
  reusing its id — plugin ids are forwarded with no namespacing, so
  `createStatusBarItem('problem-marker-status')` overwrites core's. It cannot
  delete one.

---

## 6. The design: build the capability once

Since no `.vsix` will ever reach the DOM, runtime customizability is not
something a setting grants. **Build one Theia extension that is a bridge, then
everything after it is runtime.**

### 6a. CSS layer — the userChrome.css equivalent (~20 lines)

Core already exposes the hook:

```ts
DecorationStyle.createStyleElement(styleId, document.head)   // decoration-style.ts:19
DecorationStyle.createStyleSheet(styleId)                    // → CSSStyleSheet
```

Wrap it in a `FrontendApplicationContribution` that reads a `userChrome.css`
from disk and re-reads on file change. Full restyle of every widget, hot
reload, no rebuild.

In-tree precedents doing exactly this: `PluginSharedStyle` (plugin icons,
rebuilt on every theme change), `styling-service` (contributed color themes),
`plugin-icon-theme-service`.

**Start here.** It covers *change* and *remove-by-hiding* with no security
cost and almost no code.

### 6b. JS layer — add/remove structure

Same contribution also loads a user JS file and hands it the frontend Inversify
container. Runtime code can then `shell.addWidget`, `removeBottomMenu`,
rebind — everything a build-time extension can do.

**Security: this is arbitrary code with full IDE privileges and no sandbox.**
Correct for a personal build; if Power Browser ships it, that is the entire
security model gone. Gate it behind a dev flag, or fold it into the D3/D6
permission model rather than shipping it raw.

---

## 7. Rebind seams — verified

- **`ApplicationShell` is rebindable, with a live in-tree example.**
  `@theia/toolbar` ships `ApplicationShellWithToolbarOverride`, which
  overrides `createLayout()` — the seam where topPanel / sidebars / bottom
  panel / statusBar compose into the `BoxLayout`. **This is already running in
  the Theia IDE on this machine** (`"toolbar.showToolbar": true` in
  `~/.theia-ide/settings.json`), shipping a custom row above the tab bar. It
  is the reference implementation for the unified tab strip's shell surgery.
- **Class tokens force subclassing.** `ApplicationShell`, `StatusBarImpl`,
  `BrowserMainMenuFactory` are classes used as DI tokens — you must `extend`.
  Symbol tokens (e.g. `CustomTitleWidgetFactory`) accept from-scratch
  replacement.
- **Some things have no seam.** `DynamicMenuBarWidget` is constructed with a
  bare `new` — unreachable by rebind. Replace `BrowserMainMenuFactory`
  instead.
- **`.toService()` not `.to()`.** `.toService` is a late-resolved alias that
  follows later rebinds; `.to(X)` snapshots and does not.
- **Load order is topological by dependency.** Core loads first, always.
  Declare a dependency on the package whose binding you rebind and your module
  is guaranteed to load later and win. Sibling extensions rebinding the same
  token is a race decided by key order in the app's package.json.
- **Easiest widget override:** bind another `WidgetFactory` with the same `id`
  from a later module — `WidgetManager` keys by id, last wins. The Theia IDE
  product uses this to replace the Getting Started widget.
- **Removal APIs:** `MenuModelRegistry.unregisterMenuAction(id, menuPath)`
  removes *any* node by id, submenus included (despite the name).
  `FilterContribution` / `ContributionFilterRegistry` drops an entire upstream
  contribution before it runs. `shell.leftPanelHandler.removeBottomMenu(id)`
  for the sidebar strips — no fork of core needed, just run after
  `CommonFrontendContribution`.

---

## 8. Build chain — npm is not the registry

Relevant to D1. Verified:

- Extensions are discovered by walking the app's `dependencies` for packages
  whose package.json has a `theiaExtensions` key. Resolution is pure on-disk
  `resolve-package-path`. **Zero registry calls in the entire generate/build
  path.**
- `"private": true` packages qualify — the Theia IDE's own branding extension
  proves it.
- Local `file:` / `link:` / workspace globs / symlinks all work. What fails is
  a bare semver range pointing at nothing — that dies at install, not build.
- `theia build` regenerates `src-gen/frontend/index.js` with one literal
  `require('<pkg>/<module>')` per extension, then bundles with esbuild. That
  literal list is why adding an extension needs a regenerate, not a restart.
- yarn 1 and npm workspaces are both proven in-tree. **pnpm is untested and
  likely breaks** — `theia check:hoisted` treats non-hoisted layouts as errors
  and `theia-patch` hard-codes a flat path.
- `theia build --watch` keeps esbuild in watch mode; day-to-day iteration is
  not a full rebuild.

---

## 9. Decisions taken

- **G1 — Theia chrome is DOM by default.** Browser target removes the entire
  native-OS tier. No `titleBarStyle` work is needed or possible.
- **G2 — Window chrome is Firefox's, not Theia's.** Tab strip, URL bar and
  window controls live in `power.xhtml`. Theia's own top panel and tab bar get
  hidden by the bridge extension's CSS (already in the architecture).
- **G3 — Ship the CSS customization layer; gate the JS one.** §6a has no
  security cost and covers most of the intent. §6b is arbitrary privileged
  code — dev-flag it, or fold into the D6 permission model.
- **G4 — Webviews stay cross-origin.** The same-origin override exists in
  browser target and is refused on security grounds. Webview interiors are
  permanently opaque; that is accepted, not worked around.
- **G5 — Structural shell work goes through `ApplicationShell` subclassing,**
  following `@theia/toolbar`'s `ApplicationShellWithToolbarOverride` pattern,
  not by copying or forking core.

## 10. Open questions

- Does hiding Theia's tab bar via CSS (bridge design) interact badly with
  `ToolbarAwareTabBar`'s overflow/scroll logic, which measures DOM?
- Cross-boundary drag (Firefox strip ↔ Lumino dock) needs `TheiaDockPanel`
  subclassing — Lumino's `DockPanel` owns tab drag and will not accept tabs
  from a bar it did not create. Milestone 6 polish; cost unestimated.
- Whether the userChrome.css layer should also apply to `power.xhtml`
  (one customization file for both layers) or stay Theia-scoped.
