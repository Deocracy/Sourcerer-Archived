# Architecture Research

**Domain:** Tauri 2 desktop shell with plugin applet framework (research workbench)
**Researched:** 2026-07-06
**Confidence:** HIGH (component boundaries — derived directly from fixed design handoff contract) / MEDIUM (Tauri-specific persistence/loader implementation choices — WebSearch-verified against official docs)

## Standard Architecture

### System Overview

```
┌───────────────────────────────────────────────────────────────────────┐
│                          Tauri Frontend (WebView)                      │
├───────────────────────────────────────────────────────────────────────┤
│  ┌───────────┐  ┌────────────────┐  ┌──────────────┐  ┌────────────┐  │
│  │ Title Bar │  │  Left Rail     │  │  Workspace   │  │ Assistant  │  │
│  │ (window   │  │  (nav, drag,   │  │  (dock tree, │  │ Panel      │  │
│  │  chrome)  │  │   reorder)     │  │   tabs,      │  │ (chat,     │  │
│  │           │  │                │  │   splits)    │  │  proposals)│  │
│  └─────┬─────┘  └───────┬────────┘  └──────┬───────┘  └─────┬──────┘  │
│        │                │                  │                 │        │
├────────┴────────────────┴──────────────────┴─────────────────┴────────┤
│                        Shell Store (single source of truth)            │
│   dockTree · activePaneId · railOrder/railBottom/railWidth ·           │
│   asstWidth · assistantOpen · savedLayouts · per-instance applet keys  │
├─────────────────────────────────────────────────────────────────────── │
│                     Applet Registry + Loader                           │
│   registry.ts (static import map: key → {manifest, App}) — demo stub   │
│   fallback for unregistered keys                                       │
├───────────────────────────────────────────────────────────────────────┤
│                          host API (injected per mount)                 │
│   host.storage (namespaced) · host.ai() · host.open() ·                │
│   host.instanceId · host.theme                                        │
├───────────────────────────────────────────────────────────────────────┤
│                     Tauri Command / Plugin Bridge (IPC)                │
│   window controls · store read/write · ai_complete (stub)              │
└───────────────────────────────────────────────┬─────────────────────────┘
                                                  │ IPC (invoke/events)
┌─────────────────────────────────────────────────┴─────────────────────┐
│                          Tauri Backend (Rust)                          │
│  ┌────────────┐  ┌──────────────────┐  ┌──────────────────────────┐   │
│  │ Window API │  │ tauri-plugin-    │  │ ai_complete command       │   │
│  │ (minimize/ │  │ store (JSON,     │  │ (stub in v1 — returns     │   │
│  │  maximize/ │  │  debounced,      │  │  canned/echo response;    │   │
│  │  close)    │  │  autosave)       │  │  swap point for real       │   │
│  │            │  │                  │  │  backend later)            │   │
│  └────────────┘  └──────────────────┘  └──────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────┘
```

### Component Responsibilities

| Component | Responsibility | Typical Implementation |
|-----------|----------------|------------------------|
| Title bar | Window chrome, drag region, applet-name display, layout menu | React component; `data-tauri-drag-region` on spacer only; calls `getCurrentWindow()` window API |
| Left rail | Applet navigation, reorder/pin/drag-out-to-dock | React component reading `railOrder`/`railBottom`/`railWidth` from shell store; emits dock-tree mutations via shell store actions |
| Workspace (dock tree) | Recursive layout of leaves/splits, tab bars, drag-to-dock, resizers | Pure-data recursive tree (`leaf`/`split`) in shell store + presentational React renderer walking the tree; algorithms ported near-1:1 from prototype (`hitTest`, `performDock`, `prune`) |
| Assistant panel | AI chat UI, sessions, proposals, resize/snap | React component; talks to `host.ai()`-equivalent seam at shell level (not applet-scoped) since it's a shell feature, not an applet |
| Shell store | Single source of truth for all shell-level (non-applet-private) state | One state container (Zustand/Redux-Toolkit/plain Context+useReducer); persisted wholesale via debounced writes to `tauri-plugin-store` |
| Applet registry | Maps applet `key` → `{manifest, App}`; unregistered keys render demo stub | Static import map (see Pattern 1 below) — no dynamic/runtime plugin loading in v1 |
| host API | The only surface applets may use to reach outside their own React tree | Factory function `makeHost(appletKey, instanceId)` invoked once per mounted tab; returns object closing over Tauri IPC calls |
| Tauri command surface | Window controls, store passthrough, `ai_complete` | Rust `#[tauri::command]` functions + `tauri-plugin-store` plugin registration |
| Persistence layer | Debounced write-through of shell store + `host.storage` calls to disk-backed JSON | `tauri-plugin-store`'s built-in `LazyStore` (100ms debounce) or hand-rolled debounce wrapper; versioned schema with migration function run on load |
| Demo stub renderer | Renders the standard stub UI for any rail item whose `key` is not in the registry | Single generic `AppletStub` component parameterized by manifest fields (glyph/code/title/desc) |

## Recommended Project Structure

```
src-tauri/
├── src/
│   ├── main.rs              # Tauri app bootstrap, window config (frameless)
│   ├── commands/
│   │   ├── window.rs        # minimize/maximize/close/drag passthroughs (mostly handled client-side via window API, but explicit commands if needed)
│   │   └── ai.rs            # ai_complete command (stub in v1)
│   └── lib.rs                # plugin registration (tauri-plugin-store)
├── capabilities/             # Tauri 2 permission/capability JSON (window, store, custom commands)
└── tauri.conf.json           # decorations:false, window size, etc.

src/                           # React/Vite frontend
├── shell/
│   ├── store/
│   │   ├── shellStore.ts     # dockTree, activePaneId, railOrder, railWidth, asstWidth, assistantOpen, savedLayouts
│   │   ├── schema.ts         # versioned persisted-shape type + migration chain
│   │   └── persistence.ts    # debounced save, load+migrate on boot
│   ├── titlebar/
│   ├── rail/
│   ├── workspace/
│   │   ├── dockTree.ts       # pure tree ops: hitTest, performDock, prune, splitAt
│   │   └── WorkspaceView.tsx
│   ├── assistant/
│   └── home/
├── applets/
│   ├── host/
│   │   └── makeHost.ts       # host API factory (storage/ai/open/instanceId/theme)
│   ├── registry.ts           # static import map: key -> {manifest, App}
│   ├── AppletStub.tsx        # generic demo-stub renderer
│   ├── Notes/
│   │   ├── manifest.ts
│   │   └── Notes.tsx         # first real applet
│   └── _template/
│       └── TemplateApplet.tsx
├── theme/
│   └── tokens.ts             # design tokens shared via host.theme and shell components
└── main.tsx
```

### Structure Rationale

- **`shell/store/`:** Isolating persistence/schema/migration from the store shape itself lets the migration chain evolve independently of the live in-memory shape — critical once `savedLayouts` or `dockTree` shape changes across versions.
- **`applets/registry.ts` as a flat static map:** Matches the fixed contract (`manifest` + `App({React, host})` reduced to plain TSX import since no bundler indirection is needed) and keeps applet discovery a compile-time concern — no runtime plugin loading complexity for v1 (see Pattern 1).
- **`applets/host/makeHost.ts` separate from registry:** host construction is a shell concern (needs Tauri IPC, instanceId, storage namespace) — keeping it apart from the registry keeps the registry a pure lookup table.
- **`workspace/dockTree.ts` as pure functions:** the dock-tree algorithms (hitTest, performDock, prune) are pure tree transforms with no React dependency, ported near-1:1 from the prototype's plain-React functions — testable in isolation from rendering.

## Architectural Patterns

### Pattern 1: Static Registry Map (not dynamic import.meta.glob)

**What:** `registry.ts` is a hand-written array/object of `import * as X from './applets/X'` entries, each contributing `{manifest, App}`. Unregistered rail slots fall back to `AppletStub`.
**When to use:** Always, for this project. The applet set is fixed at build time (Notes in v1, others added by editing this file as real modules are built) — there is no requirement for runtime-loaded third-party plugins in v1.
**Trade-offs:** Simpler, fully type-checked, tree-shakeable, and matches the "registry key replaces its demo stub" contract exactly. Loses the ability to add applets without a rebuild — acceptable since "Applet Builder" (a future applet for authoring new applets) is explicitly out of scope for v1 and even it would most plausibly generate source to be compiled in, not loaded at runtime in a Tauri desktop app (no safe remote-code-eval story without embedding a JS sandbox).
**Note on `import.meta.glob`:** Vite does support glob-based dynamic imports (`import.meta.glob('/src/applets/*/index.tsx')`) which would let the registry auto-discover applet folders without a manual import list, trading a small amount of magic for less registry-file churn. Either is valid; the static array is recommended for v1 for its directness and because the registry file doubles as the authoritative rail-order-independent list of what exists — pick this over glob-based auto-discovery unless the applet count grows large enough that editing the array becomes a chore.

**Example:**
```typescript
// applets/registry.ts
import * as Notes from './Notes/Notes';
// import * as Wiki from './Wiki/Wiki';  // uncomment when built — replaces the "Wiki" stub

export const applets: Record<string, { manifest: Manifest; App: AppletComponent }> = {
  [Notes.manifest.key]: Notes,
  // [Wiki.manifest.key]: Wiki,
};

export function resolveApplet(key: string) {
  return applets[key] ?? null; // null → caller renders <AppletStub manifest={demoManifestFor(key)} />
}
```

### Pattern 2: Host API as a Per-Mount Factory Closure

**What:** `makeHost(appletKey, instanceId)` is called once when a tab/pane mounts an applet, producing a fresh `host` object closing over that applet's storage namespace and instance id. `host.ai` and `host.open` close over shell-level singletons (Tauri invoke wrapper, a `openInActivePane` shell-store action).
**When to use:** Every applet mount — this is the sole sanctioned way applets touch anything outside their own component tree, per the "applets never bypass the host API" constraint.
**Trade-offs:** Slight indirection overhead vs. applets importing Tauri APIs directly, but this is exactly the seam the design mandates (it's what makes `host.ai()` swappable and keeps applet code portable/testable without Tauri running).

**Example:**
```typescript
// applets/host/makeHost.ts
export function makeHost(appletKey: string, instanceId: string): Host {
  const ns = (k: string) => `sourcerer:${appletKey}:${k}`;
  return {
    storage: {
      get: (k, fallback) => storeGet(ns(k)) ?? fallback,
      set: (k, v) => storeSetDebounced(ns(k), v),
      remove: (k) => storeRemove(ns(k)),
    },
    ai: (promptOrBody) => invoke('ai_complete', { payload: promptOrBody }),
    open: (key) => shellStore.getState().openAppletInActivePane(key),
    instanceId,
    theme: THEME_TOKENS,
  };
}
```

### Pattern 3: Debounced Whole-Store Persistence with Versioned Migration

**What:** The entire shell store (dockTree, rail state, savedLayouts, etc.) is serialized and written to a single `tauri-plugin-store` JSON file on every meaningful mutation, debounced (~150–300ms) to coalesce rapid drag/resize events. On load, a `schemaVersion` field gates a migration chain (`migrations[v] = (old) => new`) before hydrating the live store.
**When to use:** Always — this is the only persistence path for shell state per the constraint "whole workspace state persists and restores."
**Trade-offs:** `tauri-plugin-store`'s `LazyStore` already debounces (default 100ms) and autosaves, which covers most of this for free; a hand-rolled debounce on top is only needed if you want a single atomic write of the *entire* shell state rather than per-key writes, which is preferable here since dockTree/rail/assistant fields are logically one document, not independent keys. Known risk (see Pitfalls): abrupt process termination during a pending debounced write can corrupt the store file — mitigate with an explicit flush-on-close hook via the window close-requested event.

**Example:**
```typescript
// shell/store/persistence.ts
const CURRENT_SCHEMA_VERSION = 2;
const migrations: Record<number, (old: any) => any> = {
  1: (old) => ({ ...old, assistantOpen: old.assistantOpen ?? true, schemaVersion: 2 }),
};

export async function loadShellState(): Promise<ShellState> {
  const raw = await store.get<PersistedShape>('shell');
  if (!raw) return DEFAULT_STATE;
  let state = raw;
  while (state.schemaVersion < CURRENT_SCHEMA_VERSION) {
    state = migrations[state.schemaVersion](state);
  }
  return state;
}

export const saveShellState = debounce(async (state: ShellState) => {
  await store.set('shell', { ...state, schemaVersion: CURRENT_SCHEMA_VERSION });
  await store.save(); // force flush; do not rely solely on internal autosave timer
}, 200);
```

## Data Flow

### Request Flow (applet reading/writing its own data)

```
[Applet UI event (e.g. Notes textarea onChange)]
    ↓
[host.storage.set(key, value)]  — applet-private, never touches shell store
    ↓
[makeHost closure → storeSetDebounced]
    ↓ (debounced)
[Tauri invoke → tauri-plugin-store write] → [disk JSON file]

[On applet mount]
[host.storage.get(key, fallback)] → [Tauri invoke → plugin-store read] → [applet local React state]
```

### Request Flow (AI seam)

```
[Applet calls host.ai(promptOrMessagesBody)]
    ↓
[Tauri invoke('ai_complete', payload)]
    ↓
[Rust command ai_complete — v1: stub/echo response, backend TBD]
    ↓
[Promise<string> resolves back to applet]
```

### State Management (shell-level)

```
[Shell Store (dockTree, rail*, asst*, savedLayouts)]
    ↓ (subscribe/select)
[Title bar / Rail / Workspace / Assistant components] ←→ [Actions: moveTab, splitPane, resizeRail, saveLayout, ...]
    ↓ (on every mutation)
[debounced persistence.ts] → [tauri-plugin-store JSON] → (restored on next launch via loadShellState())
```

### Key Data Flows

1. **Dock-tree mutation loop:** User drags a tab or resizes a split → workspace component computes new tree via pure functions in `dockTree.ts` → shell store updated → debounced persist → all consuming components re-render off the new tree (no separate "dock service" — the tree itself is the state).
2. **Applet isolation:** Applet-private state (Notes content, per-instance UI state) never enters the shell store; it lives in the applet's own React state hydrated from/written to `host.storage`, which is namespaced (`sourcerer:<key>:<k>`) so applets cannot collide with each other or with shell keys.
3. **Registry replacement:** Rail rendering always asks the registry to `resolveApplet(key)` for every configured rail item; if found, render `<App React={ReactNS} host={makeHost(key, instanceId)} />` (or plain TSX `<App host={...} />` if the React-via-props indirection is dropped as the handoff suggests for a bundled world); if not found, render `<AppletStub manifest={demoManifestFor(key)} />`. This is the single decision point that lets replacing a stub with a real applet be a one-line registry edit.
4. **Layout save/apply:** "◱ LAYOUTS" menu reads/writes `savedLayouts` (array of `{name, tree, railWidth, asstWidth, assistantOpen}`) inside the same shell store/persistence path — not a separate storage mechanism, just a named snapshot slice of shell state.

## Scaling Considerations

This is a single-user desktop app, not a multi-tenant service — "scale" here means applet count and workspace complexity, not concurrent users.

| Scale | Architecture Adjustments |
|-------|--------------------------|
| 1 applet (Notes), few tabs | Current design as specified is sufficient; single JSON store file, static registry array |
| 5–10 applets, many tabs/instances | Still fine with static registry; consider splitting `host.storage` reads into per-namespace lazy loads if any applet's persisted data grows large (e.g. Library/Wiki caches), rather than loading everything at once |
| Real backend data (Databasise integration, later milestone) | Graduate persistence from `tauri-plugin-store` JSON to `tauri-plugin-sql` (SQLite) as the handoff already flags — likely only for applets with real structured data, not the shell store itself, which stays small and JSON-shaped indefinitely |

### Scaling Priorities

1. **First bottleneck:** Debounced whole-shell-store writes growing large if `savedLayouts` accumulates many named layouts with full dock trees — mitigate by capping/pruning stale layouts or storing layouts as a separate store key from live workspace state (already implied: prototype separates `divi-dock-layouts` from live state).
2. **Second bottleneck:** Applet-private storage growing unbounded inside one shared plugin-store JSON file (all keys in one file has a practical size ceiling before every debounced write serializes an increasingly large document) — when an applet's data materially exceeds "notes-sized" (e.g. Library with many documents), that applet should graduate to its own SQLite-backed store rather than `host.storage`, while keeping the `host.storage` contract's *shape* (get/set/remove) so applet code doesn't change, per the handoff's explicit forward-compat note.

## Anti-Patterns

### Anti-Pattern 1: Applets Reaching Tauri APIs Directly

**What people do:** Import `@tauri-apps/api` directly inside an applet module to call `invoke()` or window APIs "just this once."
**Why it's wrong:** Breaks the one enforced seam (`host`) that makes `host.ai()` backend-swappable and keeps applets portable/testable outside Tauri; also breaks the "applet = one JS/TSX module, no bundling own deps beyond React" contract from the framework README.
**Instead:** Extend `host` (and `makeHost`) with a new capability if an applet genuinely needs something not yet exposed — never reach around it.

### Anti-Pattern 2: Storing Applet-Private State in the Shell Store

**What people do:** Add Notes' draft content or another applet's UI state fields directly onto the shell store because "it's just easier to persist everything in one place."
**Why it's wrong:** Violates the explicit boundary ("shell state" vs "applet-private state stays inside each applet behind `host.storage`"); couples shell-store schema migrations to every applet's internal shape, defeating the purpose of namespacing.
**Instead:** Shell store only ever holds: dockTree, activePaneId, railOrder/railBottom/railWidth, asstWidth, assistantOpen, savedLayouts, and the *existence* of per-instance tabs (id → appletKey mapping) — never the applet's internal data, which the applet manages itself via `host.storage` keyed by its own `instanceId`.

### Anti-Pattern 3: Runtime/Dynamic Plugin Loading for v1

**What people do:** Reach for `import.meta.glob` with `eager: false`, or worse, `eval`/dynamic script injection, to support "hot-loadable" applets or an Applet Builder that writes and loads code at runtime.
**Why it's wrong:** Adds meaningful complexity (dynamic chunk loading, sandboxing untrusted generated code, CSP concerns in a Tauri WebView) for a v1 that only needs one real applet (Notes) and a fixed set of demo stubs; Applet Builder and true runtime plugin loading are explicitly out of scope.
**Instead:** Static registry array (Pattern 1). Revisit dynamic loading only if/when an "Applet Builder" milestone requires generating and loading applets without a rebuild.

### Anti-Pattern 4: Skipping the Debounce Flush on Window Close

**What people do:** Rely solely on the store plugin's internal debounce timer and let the window close; assume "it'll have saved by then."
**Why it's wrong:** A known Tauri store-plugin issue documents `LazyStore` corruption risk when the process terminates mid-write or before a pending debounced write fires (power loss/crash is the extreme case, but even a normal window-close race is possible if the debounce window hasn't elapsed).
**Instead:** Hook the Tauri window `close-requested` event to force an immediate flush/save of the shell store (and any pending applet storage writes) before allowing the window to actually close.

## Integration Points

### External Services

| Service | Integration Pattern | Notes |
|---------|---------------------|-------|
| AI backend (behind `host.ai()`) | Single Tauri command `ai_complete`, proxying to a not-yet-chosen agent/backend | v1: stubbed response (canned/echo); keep the command's request/response shape (accepts prompt string OR Messages-API-style body) stable so swapping backends later doesn't touch applet code |
| Databasise engine (Cozo+LightRAG, REST+MCP) | Deferred — integration mode (sidecar process vs external server vs later milestone) explicitly undecided | Out of scope for v1; when it lands, it will most likely become either a new Tauri sidecar or a `host`-mediated fetch to a local REST endpoint — do not pre-build either path speculatively |

### Internal Boundaries

| Boundary | Communication | Notes |
|----------|---------------|-------|
| Shell store ↔ Workspace/Rail/Titlebar/Assistant components | Direct subscribe/select (e.g. Zustand hooks) + dispatched actions | All are "shell" — no host API indirection needed, they share the same store instance |
| Shell ↔ Applets | `host` API only (storage/ai/open/instanceId/theme) | Enforced one-way seam; applets never import shell internals or Tauri APIs directly |
| Applet ↔ Applet | `host.open(appletKey)` only (opens another applet in the active pane) | No direct applet-to-applet data channel in v1; if cross-applet data sharing is ever needed, it should go through a shell-mediated API, not a shared global |
| Frontend ↔ Tauri backend (Rust) | `invoke()` for commands (`ai_complete`, plugin-store calls), Tauri window API for chrome (minimize/maximize/close), `data-tauri-drag-region` for the drag area | Capabilities/permissions (Tauri 2's capability JSON) must explicitly allow the window and store plugin APIs used, or IPC calls silently fail at runtime |

## Suggested Build Order

This mirrors and slightly expands the handoff's own suggested order, sequenced by hard dependency:

1. **Scaffold Tauri 2 + Vite/React/TS.** Frameless window (`decorations:false`), custom title bar wired to window API, capabilities/permissions configured. *Nothing else can be validated in a real window without this.*
2. **Shell store + persistence layer skeleton.** Define the store shape (dockTree, activePaneId, rail*, asst*, savedLayouts), wire debounced save/load via `tauri-plugin-store`, including `schemaVersion` field from day one even if there's only one version so far (migration scaffolding is cheap now, expensive to retrofit). *Blocks:* workspace/rail need somewhere to read/write state; persistence needs to exist before "restore on launch" can be tested at all.
3. **Port dock tree + rail rendering/algorithms** (pure functions in `dockTree.ts`, then React views) from the prototype. *Depends on (2)* for state shape; independent of applets (renders against a hardcoded/demo tab set first).
4. **Implement `host` factory + applet registry + demo-stub renderer.** `makeHost()` (storage backed by plugin-store namespacing, `ai` stubbed via `ai_complete` command, `open` wired to workspace actions), `registry.ts` static map (empty except stub fallback), generic `AppletStub`. *Depends on (2) and (3)* — needs the shell store's `openAppletInActivePane` action and a place (dock tree leaf) to render into.
5. **Port Notes as the first real applet**, registered into the registry, replacing its stub. *Depends on (4)* entirely — this is the loop-proving step (registry → host → storage → AI seam) and validates the whole framework end-to-end before any other applet is attempted.
6. **Assistant panel UI** against the stubbed AI seam (can be built in parallel with step 5 once (1)-(2) exist, since it's a shell feature, not an applet — but sequenced after Notes here because Notes is the higher-priority "does the framework work" proof point).
7. **Home view** (metro card dashboard) — lowest dependency risk, can slot in anytime after (2); depends only on shell store for empty-workspace detection and card section state.
8. **Remaining applets, one at a time**, each replacing its demo stub via the registry — strictly additive from this point, no shared-infrastructure changes expected per applet.

## Sources

- [Tauri Store Plugin docs](https://tauri.app/plugin/store/) — MEDIUM confidence, official docs, confirms JSON-backed store plugin API shape
- [tauri-apps/plugins-workspace issue #3085 — LazyStore corruption risk](https://github.com/tauri-apps/plugins-workspace/issues/3085) — MEDIUM confidence, verified GitHub issue, informs the flush-on-close anti-pattern
- [Tauri 2 Plugin Development docs](https://v2.tauri.app/develop/plugins/) — HIGH confidence, official Tauri 2 docs, confirms capability/permission model for commands
- [Vite `import.meta.glob` discussion (vitejs/vite #14161)](https://github.com/vitejs/vite/discussions/14161) — MEDIUM confidence, informs the static-vs-glob registry trade-off in Pattern 1
- Design handoff: [d:\Vibe Coding\Sourcerer\design-sync-setup-guide\design_handoff_sourcerer_tauri\README.md](../../design-sync-setup-guide/design_handoff_sourcerer_tauri/README.md) — HIGH confidence, authoritative fixed contract for this project
- Design handoff: [d:\Vibe Coding\Sourcerer\design-sync-setup-guide\design_handoff_sourcerer_tauri\reference\applets\README.md](../../design-sync-setup-guide/design_handoff_sourcerer_tauri/reference/applets/README.md) — HIGH confidence, authoritative applet framework contract
- [d:\Vibe Coding\Sourcerer\.planning\PROJECT.md](../PROJECT.md) — HIGH confidence, project scope/constraints

---
*Architecture research for: Tauri 2 desktop shell + applet plugin framework*
*Researched: 2026-07-06*
