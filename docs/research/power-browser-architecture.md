# Power Browser — Architecture Sketch (2026-08-19)

A Firefox-ESR fork with a fully custom GUI, hosting Theia IDE as a first-class
tab type in one unified tab strip. Replaces the earlier Electron plan.

Shell customization mechanics (URL bar, chrome rebind seams, build cost) live in
`tab-management-research/05-theia-shell-customization.md` (2026-08-19).

Extension model, registry strategy, and plugin-facing API decisions live in
`extension-model-decisions.md` (2026-08-19).

## Layer diagram

```
┌─────────────────────────────────────────────────────────────────┐
│ POWER BROWSER GUI  (ours — privileged XHTML/CSS/JS)             │
│  power.xhtml (replaces browser.xhtml visible UI)                │
│  ├─ Unified Tab Strip ── TabModel: [{kind: web|ide, ...}]       │
│  ├─ Toolbar / URL bar / extension buttons (ExtensionPopups)     │
│  └─ Panels: downloads, permissions, AI sidebar                  │
├─────────────────────────────────────────────────────────────────┤
│ FIREFOX FRONTEND MACHINERY  (kept, invisible)                   │
│  tabbrowser.js · SessionStore · PermissionUI hooks ·            │
│  Downloads · WebExtensions host (toolkit/) · Widevine CDM ·     │
│  Bergamot translation · Firefox AI Runtime (local inference)    │
├─────────────────────────────────────────────────────────────────┤
│ GECKO ENGINE  (untouched C++/Rust)                              │
│  content processes (one per site) · GPU proc · RDD ·            │
│  inference proc · socket proc                                   │
└─────────────────────────────────────────────────────────────────┘
        │ launches & supervises (child process)
        ▼
┌─────────────────────────────────────────────────────────────────┐
│ THEIA BACKEND  (bundled Node.js, 127.0.0.1:<port>)              │
│  browser-target Theia app · node-pty terminals · LSP servers ·  │
│  plugin host (VS Code extensions)                               │
└─────────────────────────────────────────────────────────────────┘
```

## Components (ours to build)

### 1. Unified Tab Strip
- Single ordered `TabModel`; each tab `{kind, title, icon, state}`.
- `kind: web` → backed by a `<browser remote="true">` via `gBrowser.addTab`.
- `kind: ide` → backed by a Theia widget inside ONE persistent, hidden-chrome
  `<browser>` loading `http://127.0.0.1:<port>` (mirror/proxy pattern).
- Activation: web tab → show its `<browser>`; ide tab → show the Theia
  `<browser>` and send `activateWidget(id)` to the bridge.
- Drag within strip: trivial (our DOM). Drag into Theia's split panes:
  synthesize Lumino dock-drop via bridge command (phase 2 polish).

### 2. Theia Bridge (two halves)
- **Theia extension** (`@power/browser-bridge`): subscribes to
  `ApplicationShell` events, exposes open/close/activate/list over
  `window.postMessage` (or WebSocket). Hides Theia's own tab bar via CSS.
  Uses only stable public Theia APIs → survives Theia upgrades.
- **Chrome-side module** (`PowerBridge.sys.mjs`): JSWindowActor pair relaying
  postMessage ↔ TabModel.
- **Mirror, don't query**: chrome keeps a local snapshot of Theia tab state,
  updated by push events. Strip renders from the mirror so IDE tabs repaint as
  fast as web tabs. Only mutations cross the bridge.
- **Focus policy is a milestone-3 decision**: chrome vs Theia ownership of
  Ctrl-W / Ctrl-Tab / Ctrl-P must be settled before the strip lands. Painful
  to retrofit.

### 3. Backend Supervisor (`TheiaService.sys.mjs`)
- Spawns bundled Node + Theia backend on a free localhost port at startup
  (`nsIProcess` / Subprocess.sys.mjs), health-checks, restarts on crash,
  kills on quit. Single-instance lock. Token-authenticated localhost only.

### 3b. Firefox API adapter (`PowerBrowserAPI.sys.mjs`)
Single anti-corruption layer — the ONLY module touching Firefox internals,
stable interface facing our code. One file to fix per ESR break instead of a
full audit. Also the chokepoint where plugin permission checks run, once a
plugin-facing browser API exists.

### 4. GUI shell (`power.xhtml` + modules)
- Replaces browser.xhtml's visible DOM; keeps its invisible init path where
  cheap (option-3 strategy: gut visible UI, keep machinery as SDK).
- Re-hosts required UI surfaces: permission prompts, extension
  browserAction buttons + popups (reuse `ExtensionPopups.sys.mjs`),
  context menus, downloads panel, find bar.

### 5. AI layer (later)
- Firefox AI Runtime callable from privileged JS → local summarization of
  web tabs; provider-agnostic sidebar (Claude/etc.) wired to both web and
  IDE context.

## Repo & build strategy
- Structure = Zen model: `upstream/` (firefox ESR tag) + `patches/` +
  `power/` (our frontend tree) + CI that rebases on each ESR point release
  and fails loudly on conflicts.
- **Track ESR**: mechanical security rebases every ~4 weeks, one big rebase
  per year at ESR major. Catalogue every internal-API touchpoint in
  `power/INTERNAL-APIS.md`; re-test that list each update.
- Own update channel (Zen/LibreWolf pattern). Own branding (Mozilla
  trademark rules require full rename — done: Power Browser).
- Theia app is a separate npm workspace; ships as versioned sidecar, updates
  independently of the browser build.

## What we get for free (why Firefox, not Electron)
Widevine DRM (streaming works) · full WebExtensions (uBlock, LastPass —
popups re-hosted by us) · Mozilla security patch stream · local translation ·
open-source local AI runtime · per-site process isolation.

## Cost accepted
Perpetual ESR rebase discipline · we own updater infra · extension UI
surfaces and permission prompts must be re-hosted · no Chrome-only features
(Google sync etc.) · any plugin-facing browser API needs its own manifest
permission model built first (see extension-model-decisions.md, D6).

## Build order
1. **Boot**: Firefox ESR builds under our branding, stock UI. (infra)
2. **Shell**: power.xhtml with web-only tab strip + toolbar via gBrowser.
3. **Theia**: supervisor + one "IDE" tab loading Theia; bridge activate/list.
4. **Unify**: editor tabs appear as first-class tabs; open/close/reorder.
5. **Re-host**: extension buttons/popups, permission prompts, downloads.
6. **Polish**: cross-boundary drag, session restore of mixed tabs, AI sidebar.

Milestone 3 is the earliest daily-drivable build; 5 is the earliest
shareable one.

Milestone 3 also settles two architectural GUI decisions that are cheap now
and miserable later: **tab-state mirroring** and **focus/keyboard
arbitration**. A plugin-facing browser API is milestone 6+ — real
WebExtensions and Open VSX plugins already cover their halves.

## OS support & distribution (added same day)

**Desktop: Windows / macOS / Linux all first-class** (inherited Firefox build
targets; Theia sidecar supports all three). Mobile out of scope — Android
would be a separate GeckoView/Kotlin product; iOS has no shippable Gecko.

- Windows: `mach package` emits the standard NSIS one-click setup.exe.
  Authenticode cert (~$100–400/yr) only to avoid SmartScreen warnings.
- macOS: .dmg drag-install; sidecar lives in .app/Contents/Resources.
  Apple Developer $99/yr for signing+notarization — effectively mandatory
  at public distribution, not before.
- Linux: Flatpak/Flathub is the one-click path (copy Zen/LibreWolf/Floorp
  manifests). Gotcha: sandbox confines the Theia backend — needs filesystem
  permissions + `flatpak-spawn --host` for terminals (VS Code Flatpak
  pattern). $0 to sign.
- All signing costs = distribution-time only; personal/dev builds cost $0.
  Widevine needs NO extra certification on the Firefox route (unlike
  Electron/Castlabs VMP) — forks play DRM content today.

**Bundling Theia into the installers is minor**: per-platform Node binary +
built Theia app (~150–250MB) added to the package layout; native-module
prebuilds (node-pty) per OS/arch — the VS Code/Theia distribution pattern.
Policy: browser + sidecar always ship/update in lockstep through our update
channel so the bridge never meets a mismatched Theia.

**Effort**: ~1 week/platform first time (crib from Zen/Floorp CI), then
automated. Packaging is beaten path; the project's real costs remain the GUI
rebuild and ESR rebase discipline. Plan: develop on Linux, add Win+mac CI at
milestone 2, defer signing until milestone 5.
