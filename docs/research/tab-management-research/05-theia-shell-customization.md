# Theia Shell Customization — URL Bar, Chrome Rebuild, Build Cost (2026-08-19)

Session findings, verified against the local checkout at `sources/theia` (v1.74.1)
and `sources/theia-ide` (1.74.100). Companion to `power-browser-architecture.md`
(layer ownership) and `extension-model-decisions.md` (plugin API model).

---

## 1. The bar already in the IDE

The row with back/forward/split above the tab strip is `@theia/toolbar`, not
something custom. Defaults are exactly what's visible:
`packages/toolbar/src/browser/toolbar-defaults.ts` → `textEditor.commands.go.back`,
`.go.forward`, `workbench.action.splitEditorRight`, then command palette + terminal
on the right.

Those arrows are **not** browser history. They run `NavigationLocationService`
(`packages/editor/src/browser/navigation/`), a stack of **cursor positions inside
text editors**. Terminals, Settings, chat tabs, and webviews are invisible to it.
Command ids: `packages/editor/src/browser/editor-command.ts:90-102`.

`~/.theia-ide/toolbar.json` customization adds/removes **command buttons only** —
no text input, no per-tab state.

## 2. "Does every tab have a URL?" — no, but it can be made true

- Universal identity is `WidgetManager`'s `factoryId` + options JSON. That is what
  layout restore uses. It is **not** a URI.
- A URI exists only when the widget implements `Navigatable.getResourceUri(): URI | undefined`
  — `packages/core/src/browser/navigatable-types.ts:23-38`. Editors implement it;
  Settings, terminals, chat views do not.
- To make it universal: open everything through `OpenerService.open(uri)` with own
  schemes (`chat://`, `web://`, `settings://`), register a
  `NavigatableWidgetOpenHandler` per scheme
  (`packages/core/src/browser/navigatable.ts:22-37`), implement `getResourceUri()`
  on each widget type.
- Theia's shell makes no architectural distinction between "editor" and "view", so
  nothing blocks this. Matches the URI-as-universal-handle model documented in
  `tab-management-research/04-everything-is-a-url.md`.

**This is the reusable work.** It is identical whether the bar ends up in Theia or
in browser chrome, and it is the prerequisite for either.

## 3. Adding a bar — mechanics

Two levels, different cost:

- **Field in the existing toolbar** — implement `ToolbarContribution`;
  `AbstractToolbarContribution.render()` returns arbitrary React
  (`packages/toolbar/src/browser/abstract-toolbar-contribution.tsx`). ~150 lines.
  Read the active tab's URI via `ApplicationShell.onDidChangeCurrentWidget` +
  `NavigatableWidget.is(w) && w.getResourceUri()`.
- **A whole new row** — the shell is one vertical `BoxLayout`; `@theia/toolbar`
  injects itself into it at
  `packages/toolbar/src/browser/application-shell-with-toolbar-override.ts:85-86`
  (`createBoxLayout([topPanel, toolbar, panelForSideAreas, statusBar])`). Rebind
  `ApplicationShell` to a subclass, add a widget to that array.

Back/forward for non-editor tabs needs its own per-tab history stack.
`NavigationLocationService` does not generalize.

## 4. No VS Code extension can do this

VS Code's extension API exposes no insertion point for a workbench toolbar row —
see `code.visualstudio.com/api/extension-capabilities/extending-workbench`;
open request `microsoft/vscode#41309`. What exists on the marketplace:

- Toolbar (XiLaiTL) — button rows, but inside a **sidebar view**.
- Shortcut Menu Bar — buttons in the **editor title** area only.
- Action Buttons — **status bar** only.
- customize-ui — patches `workbench.html`; trips corruption warning, breaks on
  update. Not viable.
- Webview panel — a URL bar **inside a tab**, driving only its own iframe. Cannot
  see or drive other tabs.

Theia does expose the hooks VS Code refuses to. That asymmetry is a reason to be on
Theia, and the same reason the extension doesn't exist to be copied.

## 5. Rebuilding Theia's chrome — rebind, don't copy

Every shell surface is DI-bound and rebindable. `@theia/toolbar` is the only
`rebind(ApplicationShell)` in the whole framework repo — proof the seam works.

| Surface | Rebind target | File |
|---|---|---|
| Menu bar (File/Edit/…) | `BrowserMenuBarContribution`, `BrowserMainMenuFactory` | `core/src/browser/menu/browser-menu-module.ts:26-31` |
| Left activity bar | `SidePanelHandler` (builds `SideTabBar`) | `core/src/browser/frontend-application-module.ts:179-180` |
| Sidebar bottom icons | `SidebarBottomMenuWidgetFactory` | same module |
| Tab appearance | `TabBarRendererFactory` / `TabBarRenderer`, `DockPanelRenderer` | `frontend-application-module.ts:202-208` |
| Whole layout / extra rows | `ApplicationShell` subclass | `toolbar/src/browser/application-shell-with-toolbar-override.ts` |
| Status bar | `bindStatusBar` | `frontend-application-module.ts:337` |
| Own React panels | `WidgetFactory` + `bindViewContribution` + `ReactWidget` | `examples/api-samples/src/browser/view/sample-unclosable-view.tsx` |

Note: the in-window menu bar visible in Theia IDE is the custom title-bar path
(browser menu factory), not Electron's native menu.

**Policy: subclass and rebind; copy a `render()` body only where no seam exists,
and catalogue those spots** — same anti-corruption discipline as
`PowerBrowserAPI.sys.mjs` and `power/INTERNAL-APIS.md`. A copied file is owned
forever; a subclass owns only the delta and takes Theia upgrades for free.

## 6. Build cost

`sources/theia-ide` has **never been built here** — no `node_modules`. The running
IDE is the read-only Nix-store 1.74.100 and cannot be patched; a local build is a
second, parallel install.

- Needs node ≥22, yarn 1.x: `yarn && yarn build:dev`, plus `yarn download:plugins`.
  ~3–5 GB, 15–40 min first pass.
- New extension goes in `theia-extensions/<name>/` (copy the shape of
  `theia-extensions/product/`), then added to `dependencies` of
  `applications/browser/package.json` and `applications/electron/package.json`.
- NixOS risk is native modules (`node-gyp`) and the Electron binary, not the
  TypeScript. **Build `applications/browser` first** — no Electron binary, iterate
  with `yarn watch` + reload. Fallback: `browser.Dockerfile` already exists.
- Estimate: ~half a day for a working toolbar field in the browser build, assuming
  `yarn install` cooperates. Making every tab URI-addressable is a separate,
  larger job.

## 7. Where this lands in the Power Browser design

In the Firefox-fork architecture the URL bar is **chrome-side** (`power.xhtml`
toolbar), because one bar serves both web and IDE tabs. Component split for it:

1. `power.xhtml` toolbar — the single bar, renders from the TabModel.
2. `gBrowser` — back/forward for `kind: web`, free.
3. `PowerBridge.sys.mjs` — relays active-widget URI into the chrome mirror.
4. `@power/browser-bridge` (Theia ext) — `onDidChangeCurrentWidget` →
   `getResourceUri()` push. ~30 lines, stable public API.
5. Per-widget `Navigatable` implementations — the real pile of work (§2).
6. Per-tab history stack for `kind: ide` — chrome-side, ours to write.

Consequence: **a Theia-side URL bar is throwaway.** So is any rebuild of Theia
chrome that the browser chrome will later own and hide (tab strip, and per the
bridge design Theia's own tab bar is hidden by CSS). Rebuild only Theia-owned
surfaces.

## Decisions taken this session

- D1. URL bar belongs to browser chrome, not a Theia toolbar contribution.
- D2. Shell customization is by DI rebind + subclass, never by copying files;
  seams that force a copy get catalogued.
- D3. First build target is `applications/browser`, not Electron.
- D4. Start work at per-widget `Navigatable`/URI schemes — the only part that is
  reusable under either layer split.

## Open questions

- **URL semantics** (new; decide alongside the milestone-3 items): what does a chat
  tab's URL *mean*, is it typeable, does typing one open the tab? Cheap now,
  miserable later — same class of decision as tab-state mirroring and
  focus/keyboard arbitration.
- Which surfaces are chrome-owned vs Theia-owned, enumerated surface by surface,
  before any chrome rebuild starts.
