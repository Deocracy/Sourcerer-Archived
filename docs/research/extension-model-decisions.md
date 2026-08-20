# Extension Model — Decisions (2026-08-19)

Companion to `power-browser-architecture.md`. Covers how extensions work in
Power Browser: which systems exist, what a registry can offer, and what was
decided vs deferred.

## The terminology trap

Theia says "extension" for two unrelated things. Everything below depends on
keeping them apart.

| | Theia extension | Plugin (`.vsix`) |
|---|---|---|
| Package | npm `@scope/name` | `.vsix` from a registry |
| Loaded | compile-time, into the binary | runtime |
| Runs in | frontend/backend directly, full DI | sandboxed plugin host |
| Can | rebind core services, change shell, unbind features | VS Code API surface only |
| Install/uninstall UI | none, ever | yes — the Extensions panel |
| Registry | npm (already exists) | Open VSX (self-hostable) |

The Extensions panel lists **only** plugins. The file explorer, git view,
terminal, and AI chat are Theia extensions and will never appear there. This
is architectural, not a tooling gap.

Upstream = Firefox and Theia. Downstream = Power Browser.

## Decisions

### D1 — No custom registry for Theia extensions
npm is already the registry. Publish `@power/*` packages; consumers add them
to `package.json` and rebuild. A private npm scope covers anything closed.

### D2 — Self-hosted Open VSX for plugins
The only registry we actually build/operate. Gives a curated `.vsix` list in
the Extensions panel. Che and other vendors run private instances; copy that.

### D3 — Custom plugin API namespace, when needed
Theia supports injecting an extra namespace into the plugin host via
`ExtPluginApiProvider` (plugin side) + `MainPluginApiProvider` (privileged
side). Precedent: Eclipse Che ships `@eclipse-che/plugin` alongside `vscode`.

**Shape decision:** mirror the WebExtensions `browser.*` API rather than
inventing one. Semantics, permission model, and developer familiarity come
free; porting a WebExtension becomes mechanical.

### D4 — AI chat variants ship as agents, not extensions
`@theia/ai-core` already has an agent registry with per-agent enable/disable
surfaced in the AI Configuration view. Five chat variants = five registered
agents. No packaging work. Only escalate to separate extensions if variants
differ structurally (different widget / shell placement), not just by prompt,
model, or tools.

### D5 — One anti-corruption layer for Firefox internals
A single module (`PowerBrowserAPI.sys.mjs`) is the ONLY thing in our tree that
touches Firefox internals, exposing a stable interface inward. Formalizes what
`power/INTERNAL-APIS.md` currently only catalogues.

Buys: one file to fix per ESR break instead of a full audit; third-party
plugins never see the churn. Does NOT reduce how often upstream breaks — only
how wide each break is.

Asymmetry: Theia has stable public APIs, so discipline alone suffices there
(already the bridge's stated policy). Firefox promises nothing for chrome JS,
so the adapter is mandatory on that side.

### D6 — Permission model is a hard gate on D3
Exposing browser control to third-party plugins without a manifest permission
model + install-time prompts turns our registry into a spyware channel. The
adapter (D5) is the chokepoint where every call gets checked. **No third-party
browser API ships before this exists.**

### D7 — Plugin browser API is milestone 6+
Real WebExtensions already work (AMO, uBlock, LastPass) and Theia plugins
already cover IDE work (Open VSX). The bridge only earns its cost for
extensions needing *both halves at once* — the unified thesis. Do not design
the API before the milestone-3 bridge exists and its real latency is measured.

## Constraints and what actually mitigates them

Four constraints on plugin-facing browser APIs. Two are fixed by D5/D6; two
are design constraints, not defects.

1. **Async everywhere** — not fixable (process boundary). Also not a problem:
   VS Code's API and `browser.*` are both already promise-based. Only bites if
   you design a sync API. Don't.
2. **Content-process hop** — not fixable (Gecko isolation). Mitigated by
   design, below.
3. **Permission model** — fixed by D6.
4. **Rebase surface** — fixed by D5.

**Principle for 1 and 2: move the code to the data, not the data to the code.**

- Content scripts — plugin registers a script running *in* the content
  process, returns only the result. Why WebExtensions have them.
- Mirror state locally — tab-model snapshot on the Theia side, updated by push
  events. Reads (query, titles, URLs, favicons) become local and instant; only
  mutations cross the bridge. Removes latency from ~90% of calls.
- Pass handles, not payloads — tab ids, never DOM trees.
- Batch bursts at the adapter.
- Keep interactive paths (keystroke, scroll, hover) off the bridge entirely.

Hop count is structural, not a performance verdict — JSWindowActor IPC is
sub-millisecond and cost is dominated by payload size. Build the bridge, then
measure; add mirroring where it actually shows up.

## GUI consequences (ranked)

1. **Split-personality tab strip** — web tabs repaint in-process instantly,
   IDE tabs a round-trip later, side by side in one strip where it's most
   visible. Fix: render from the local mirror.
2. **Focus / keyboard arbitration** — Ctrl-W, Ctrl-Tab, Ctrl-P: chrome and
   Theia both want them. Wrong policy = shortcuts work depending on last
   click. Painful to retrofit.
3. **Optimistic close vs dirty editors** — tab must close on click, but Theia
   can refuse (unsaved changes). Needs an explicit un-close reconcile path.
4. **Cross-boundary drag** — must track cursor at display refresh; synthesizing
   Lumino drop targets live is the hardest GUI problem here. Already deferred
   to phase 2 in the architecture note. Correctly deferred.
5. **Startup pop-in** — backend takes seconds; web tabs restore instantly, IDE
   tabs don't. Needs a real per-tab skeleton state.
6. **Two context-menu systems** — Theia's DOM menu vs chrome menu. Needs
   explicit per-region suppression.
7. **Theme / zoom seams** — separate theming systems; toggles must propagate
   across the bridge or the boundary is visible.
8. **Backend crash** — supervisor restarts Theia; GUI must degrade IDE tabs,
   never delete them. Losing someone's open files is unrecoverable.
9. **Resize relayout** — Lumino relayouts a round trip after chrome resizes.

**Decide early (architectural):** #1 and #2. Both cheap now, miserable to
retrofit. Everything else is milestone 5–6 polish.

**Not a problem:** scroll never crosses the bridge. Page scroll is Gecko APZ
on the compositor thread at display refresh; editor scroll is Monaco in its
content process; tab-strip scroll is chrome DOM. All full refresh rate.
Only cross-boundary drag is latency-sensitive.
