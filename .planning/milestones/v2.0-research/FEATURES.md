# Feature Research

**Domain:** Dockable multi-pane desktop workbench shell (IDE/Obsidian/JupyterLab-class) + plugin/applet framework
**Researched:** 2026-07-06
**Confidence:** MEDIUM-HIGH (docking/IDE UX patterns are well-established and cross-verified across VS Code, JetBrains Rider, Obsidian, JupyterLab, golden-layout/dockview; project-specific stub/applet framing is unique to this handoff, so those calls are the project's own design intent rather than externally verified)

## Context

The design handoff (`design-sync-setup-guide/design_handoff_sourcerer_tauri/`) is an unusually complete behavioral spec — it already nails dock tree mechanics (5-zone drag docking, splits, resizers), a 3-mode left rail, named layouts, full workspace persistence, an applet framework contract (`manifest` + `App({React, host})`), a Home card dashboard, and an assistant panel. This research does **not** re-derive that — it audits it against what dockable-shell products in the wild are expected to have, flags what's present vs. silently assumed vs. genuinely absent, and sorts what's genuinely v1-required from what's safe to defer given the stated v1 scope: **pixel-perfect shell + applet framework + one real applet (Notes) + everything else as stubs.**

## Feature Landscape

### Table Stakes (Users Expect These)

Features users of VS Code / Rider / Obsidian / JupyterLab assume exist. Missing these makes the shell feel broken, not just "less featured."

| Feature | Why Expected | Complexity | Notes |
|---------|--------------|------------|-------|
| Drag-to-dock with 5-zone drop targets, splits, resizers | Baseline in every docking product (VS Code editor groups, Rider tool windows, golden-layout/dockview) | MEDIUM-HIGH | **Already fully specced in the handoff.** Port prototype algorithms near-1:1 (`hitTest`, `performDock`, `prune`). No new design work needed. |
| Tab bar per pane: reorder, close (✕), multi-instance | Universal in tabbed editors | LOW-MEDIUM | Already specced. |
| Workspace persistence across restarts (layout, open tabs, panel widths) | Users lose trust instantly if closing the app loses their workspace | MEDIUM | Already specced as a v1 requirement ("persist the whole workspace... restored on launch") — but see Gaps below for edge cases the handoff doesn't enumerate. |
| Named/saved layouts | Common in Rider, VS Code (workspace profiles), tiling WMs | LOW-MEDIUM | Already specced (◱ LAYOUTS menu). |
| Left rail / sidebar with collapse or resize | Every IDE-like shell has a collapsible primary nav | LOW-MEDIUM | Already specced (3 modes: expanded/compact/hidden). |
| Basic keyboard shortcuts for the most common actions (close tab, cycle rail, send message) | Power users of this product category expect *some* keyboard-first workflow, even if not exhaustive | LOW | Handoff specifies ⌘\ (cycle rail), ⌘↵ (send), y/d/n (proposal actions) — a reasonable v1 subset, not exhaustive. See Gaps: no tab-close shortcut, no pane-focus-cycle shortcut specified. |
| Window controls (min/max/close) wired correctly, incl. double-click-to-maximize on title bar | Any frameless custom-titlebar app that gets this wrong feels amateurish | LOW | Already specced; must wire real Tauri window API, not just visual buttons. |
| Empty-state / first-run experience (Home view when workspace is empty) | Users need to know it's not broken when no panes are open | LOW-MEDIUM | Already specced (Home metro dashboard). |
| Resizable panels with sane min/max width clamps | Prevents panels collapsing to 0 or exploding off-screen | LOW | Not explicitly stated in handoff (grip drag is specced, but min/max clamp values are not) — flag for requirements. |
| Focus indication (which pane/tab is "active") for keyboard input routing | Without this, keyboard shortcuts (⌘↵ send, y/d/n) are ambiguous when multiple panes are open | MEDIUM | **Gap.** Handoff defines *visual* active-tab styling but not an explicit focus-management model (which element receives keydown, how focus moves on click vs. drag, what happens to focus when a pane is closed/docked away). This is exactly the kind of thing VS Code invests heavily in (Focus Next/Previous Part, F6/Shift+F6) and it is common for clones of docking UIs to miss it until users complain. Recommend as an explicit v1 requirement even though it's invisible in static comps. |
| Multi-instance tab data isolation | Handoff explicitly allows multiple tabs of one applet (e.g., two Notes tabs) | LOW-MEDIUM | Already specced via `host.instanceId`; correctness (no shared mutable state leaking across instances) is an implementation risk in Notes, not a design gap. |
| Session/state restore for the *one real applet* (Notes) specifically — not just the shell | Users expect their actual notes content to survive restart, same as the shell layout | LOW | Implied via `host.storage`, but worth stating explicitly as its own acceptance criterion distinct from "shell restores." |
| Reasonable degrade path when persisted state is corrupt/missing/stale (e.g., a saved layout references an applet key that no longer exists, or JSON fails to parse) | Any product with local persisted state hits this in practice (schema changes during development) | LOW-MEDIUM | **Gap.** Not addressed in handoff. Cheap to build defensively (fallback to default layout) but easy to forget and causes a poor first-run-after-schema-change experience. Flag as a requirement, not a research topic — low complexity, high annoyance if skipped. |

### Differentiators (Competitive Advantage)

Not required for v1 functional completeness, but this is where the product's stated identity ("part demo, part working application," "believable stub ready to be replaced") pays off.

| Feature | Value Proposition | Complexity | Notes |
|---------|-------------------|------------|-------|
| High-fidelity demo stubs for every unbuilt applet (not blank placeholders) | Most nascent plugin platforms ship with ugly "coming soon" screens; a fully-styled, information-dense stub (glyph tile, code crumb, demo rows) sells the vision of the finished product before it's built — genuinely unusual and valuable for stakeholder demos / investor-style walkthroughs | LOW-MEDIUM | Already specced; this *is* the differentiator per PROJECT.md ("the 'part demo' half is a feature, not debt"). Keep pixel-perfect fidelity — sloppy stubs undercut the whole pitch. |
| Applet framework with a genuinely small, host-mediated contract (`manifest` + `App({React, host})`, 5-member `host` API) | Low ceremony to add an applet (copy template, fill manifest, register) — this is the real bet of the project; most Tauri/Electron shells hard-wire panels instead of building a plugin seam this early | MEDIUM | Already specced and proven in the HTML prototype. The differentiator is doing this *before* most of the real applets exist, which very few hobby/production shells attempt (usually plugin APIs get retrofitted after the app grows organically and becomes hard to decouple). |
| Persistent, always-visible AI assistant panel wired through one seam (`host.ai()`) rather than per-applet AI integrations | Every applet gets AI "for free" without needing its own model integration, and swapping the backend later touches one file | LOW (as a stub) / HIGH (once real backend lands, deferred) | v1 keeps this a stub — correctly deferred per PROJECT.md. The differentiator is the seam design itself, not the AI quality, in v1. |
| Assistant proposal review UX (approve/diff/reject via y/d/n) | Distinguishes "AI chat bolted on" from "AI as a reviewable collaborator" — a meaningfully different trust model than a bare chat box | MEDIUM | Already specced; keep even though backend is stubbed — proves the interaction pattern independent of model quality. |
| Metro-card Home dashboard with draggable sections + FLIP animation | Most shells default to a blank tab or a static "recent files" list; a living, reorganizable dashboard is a genuine point of delight | MEDIUM | Already specced; low functional necessity but cheap relative to its demo impact, keep in v1 scope as specced. |

### Anti-Features (Commonly Requested, Often Problematic — Correctly Out of Scope for v1)

| Feature | Why Requested | Why Problematic (for this v1) | Alternative |
|---------|---------------|------------------|-------------|
| Multi-window support (detach a pane into its own OS window) | Common ask in IDE-class products (VS Code, Rider both support it) and would feel "expected" eventually | Massively increases scope: needs cross-window state sync, a second Tauri window with its own dock tree, drag-between-windows hit-testing that the ported prototype doesn't have. The handoff explicitly specs a **single frameless main window**. Building this now would blow the "pixel-perfect shell + one applet" v1 budget. | Correctly deferred. Note it as an explicit non-goal in requirements so nobody accidentally half-builds toward it (e.g., don't architect dock-tree state as globally singleton in a way that makes multi-window impossible later — cheap insurance, not a v1 feature). |
| Full command palette (⌘K style fuzzy command search) | Table stakes in VS Code/Obsidian-class tools; feels like an obvious gap for power users | Handoff doesn't spec one at all — building it now means inventing UX not in the source-of-truth design, violating "port the handoff, don't add features." | Defer to a later milestone; if wanted, it should go through the same design/handoff process, not be improvised during shell-build. |
| Real AI backend / live model calls in `host.ai()` | Obviously "the interesting part" and tempting to wire up early since the seam already exists | PROJECT.md explicitly defers this — backend choice, cost, rate limits, and prompt/response shape are all undecided. Building it now risks having to redo the seam's contract once a real decision is made. | Keep `host.ai()` as an in-repo stub returning canned/deterministic responses; preserve the seam exactly as specced so swapping is a one-file change later. |
| Databasise engine integration (real Wiki/Library/Graph data behind those applet stubs) | The whole point of Sourcerer long-term, so there's pull to "just wire it in since Databasise already exists and is at v3.0" | PROJECT.md explicitly defers integration-mode decision (sidecar vs. external server vs. later) to its own milestone. Wiring it in ad hoc during shell-build would lock in an integration architecture decision by accident instead of on purpose. | Keep those applets as stubs; the shell's job in v1 is to prove the *framework* can host a real applet (Notes), not to prove Databasise integration. |
| Building out every other applet (Library, Wiki, Graph, Chat, Writing Studio, Power Browser, Kanban, News, KeyPass) even partially "for free" while building the framework | Temptation once the registry/host pattern exists — "it's just a few more files" | Directly contradicts the stated v1 scope (Notes is the *only* real applet) and dilutes the actual v1 goal (prove the loop end-to-end once, cleanly) into a shallow multi-applet sprawl that's harder to test and review | Ship every other applet as the pixel-perfect stub only; replace them one at a time in later milestones per PROJECT.md's own stated cadence. |
| Full accessibility/screen-reader parity (ARIA roles, full screen-reader navigation model) matching VS Code's accessibility investment | VS Code treats this as core and it's a legitimate long-term expectation for a serious desktop tool | This is a substantial, ongoing investment (VS Code has dedicated accessibility docs, F6-based part-focus navigation, screen-reader optimizations) that is disproportionate to a v1 whose explicit goal is pixel-perfect visual fidelity + one working applet | Do the cheap, structural version now (keyboard focus management as a table-stakes item above, semantic HTML where free) and treat full a11y compliance as an explicit later milestone, not a silent gap to discover at the end. |

## Gaps: What the Handoff Doesn't Cover (Completeness Check)

The handoff is unusually thorough, so most classic docking-shell gaps are already closed. What remains, ranked by how likely it is to bite during implementation:

1. **Focus management model** (table stakes, flagged above) — visual "active tab" styling is specced; the underlying focus-routing model (click vs. keyboard, what happens on tab close, split, or applet swap) is not. This is the single most common silent gap in docking-UI clones because it doesn't show up in static comps or even in casual manual testing — it surfaces as "keyboard shortcuts feel randomly broken."
2. **Persisted-state corruption/staleness handling** (table stakes, flagged above) — no fallback behavior specified for malformed/outdated `tauri-plugin-store` JSON (e.g., after a schema change mid-development, or a saved layout referencing a since-renamed applet key). Cheap to add defensively; expensive to retrofit once users have "broken" saved layouts in the wild.
3. **Resize clamp values** (table stakes, flagged above) — rail width, assistant width, and split-pane min sizes need explicit min/max bounds; the handoff describes the drag *behavior* (grip, drag-resize, snap-to-close) but not numeric bounds. Small but must be decided somewhere (requirements or first implementation pass), not left ambiguous.
4. **Multi-window** — genuinely absent, and correctly so; call it an explicit non-goal (see Anti-Features) so architecture doesn't accidentally block it later without anyone deciding that on purpose.
5. **Keyboard shortcut completeness** — the specced set (⌘\, ⌘↵, y/d/n) is a deliberate minimal subset per the prototype, not an oversight; VS Code-style exhaustive keyboard nav (F6 part-cycling, arrow-key tab navigation) is reasonable to defer, but the *baseline* (Tab/Shift+Tab moving through interactive controls in a sane order, Escape closing menus/overlays) should not be assumed free — it needs deliberate DOM/tabindex structure decisions during build, not just CSS.
6. **What happens when the last tab in the last pane closes** — the handoff says "empty tree renders Home," which answers this at the whole-workspace level; not explicitly stated is behavior when a *split* pane empties (does it prune back to a single pane per the `prune` function mentioned, or leave an empty split visible?). The prototype's `prune` function is referenced by name in the handoff, implying this is already solved in the reference implementation — verify behavior when porting rather than re-deriving it.

None of these are new features to design — they're implementation-completeness questions that a v1 acceptance pass should explicitly check, since they're the kind of thing that's invisible in a static high-fidelity comp but immediately obvious to a real user within the first five minutes of use.

## Feature Dependencies

```
Applet framework (registry, host API, manifest contract)
    └──requires──> host.storage (tauri-plugin-store binding)
    └──requires──> host.ai (stub implementation, seam preserved)
    └──requires──> Demo stub renderer (for all unregistered applet keys)

Notes applet (first real applet)
    └──requires──> Applet framework (registry loader, host API)
    └──requires──> host.storage (persisted note content)
    └──enhances──> Applet framework validation (proves registry-replaces-stub works end-to-end)

Workspace dock tree (tabs, splits, 5-zone docking)
    └──requires──> Focus management model (for keyboard shortcuts to target the right pane) [GAP — not explicit in handoff]
    └──enhances──> Named layouts (layouts are saved/restored dock-tree snapshots)

Full workspace persistence
    └──requires──> Dock tree serialization (tree, rail order/pins, widths, open tabs)
    └──requires──> Corrupt/stale-state fallback [GAP — not explicit in handoff]
    └──conflicts with──> Multi-window (single persisted tree assumes one window; would need reconciliation if multi-window is ever added)

Dashboard Assistant panel
    └──requires──> host.ai() seam (stubbed in v1)
    └──enhances──> Notes applet (AI summarize feature uses same seam)

Left rail (3 modes) + drag-out-to-dock
    └──requires──> Workspace dock tree (drag-out targets the same 5-zone dock logic)
```

### Dependency Notes

- **Notes applet requires the applet framework, not the reverse:** the framework (registry, host API, stub renderer) must be functionally complete *before* Notes can be ported as the reference real applet — this fixes phase ordering (framework phase before Notes phase).
- **Focus management is an invisible dependency of the dock tree:** because it's not explicit in the handoff, it's easy to schedule the dock tree as "done" once drag/drop/resize work visually, then discover keyboard shortcuts (⌘\, ⌘↵, y/d/n) don't reliably target the right pane. Recommend making focus routing an explicit acceptance criterion of the dock-tree phase, not a separate later phase.
- **Persistence corruption handling enhances (doesn't block) full workspace persistence:** it can ship as a fast-follow within the same phase without blocking the core persist/restore happy path, but should not be pushed to "later" indefinitely since every developer will hit a stale-schema state at least once during iteration.
- **Multi-window conflicts with the single-persisted-tree assumption:** not a v1 concern, but worth a one-line architectural note (don't hard-code "exactly one window" assumptions somewhere that would require a rewrite, even though multi-window itself is correctly out of scope).

## MVP Definition

### Launch With (v1) — matches PROJECT.md's stated Active requirements

- [ ] Frameless Tauri window + custom title bar wired to real window API — table stakes, zero differentiation risk
- [ ] Left rail (3 modes, resize, reorder/pin/drag-out-to-dock) — table stakes, already fully specced
- [ ] Workspace dock tree (tabs, 5-zone docking, splits, resizers, multi-instance tabs) — table stakes, already fully specced; **add explicit focus-management acceptance criteria**
- [ ] Dashboard Assistant panel UI against stubbed AI seam — differentiator (proposal review UX), correctly stubbed per PROJECT.md
- [ ] Home metro dashboard with draggable sections + FLIP — differentiator, cheap relative to impact
- [ ] Named layouts + full workspace persistence — table stakes; **add explicit corrupt/stale-state fallback and resize-clamp bounds**
- [ ] Applet framework (registry loader, host API, demo stubs for all unbuilt applets) — the core differentiator/bet of the whole project
- [ ] Notes as first real applet — table stakes for validating the framework loop end-to-end

### Add After Validation (v1.x)

- [ ] Fuller keyboard navigation (Tab-order completeness, Escape-to-close-overlay conventions, F6-style part cycling) — trigger: once shell is stable, before broader user testing
- [ ] Second real applet beyond Notes — trigger: once Notes has proven the loop is genuinely reusable, not just correct for one case
- [ ] Command palette — trigger: if user testing surfaces demand; not in current design handoff, needs its own design pass first

### Future Consideration (v2+)

- [ ] Real AI backend behind `host.ai()` — deferred by explicit user decision (own milestone)
- [ ] Databasise engine integration (live Wiki/Library/Graph data) — deferred by explicit user decision (own milestone)
- [ ] Multi-window support — defer indefinitely unless a concrete user need emerges; correctly treated as an anti-feature for now
- [ ] Full accessibility/screen-reader compliance — defer to a dedicated milestone once the visual/functional shell is validated

## Feature Prioritization Matrix

| Feature | User Value | Implementation Cost | Priority |
|---------|------------|---------------------|----------|
| Dock tree (drag/split/resize) | HIGH | HIGH | P1 |
| Applet framework (registry/host/stubs) | HIGH | MEDIUM | P1 |
| Notes real applet | HIGH | LOW-MEDIUM | P1 |
| Full workspace persistence | HIGH | MEDIUM | P1 |
| Left rail 3 modes | MEDIUM-HIGH | LOW-MEDIUM | P1 |
| Focus management (explicit) | MEDIUM (invisible until missing) | LOW-MEDIUM | P1 |
| Persisted-state fallback handling | LOW-MEDIUM (invisible until it happens) | LOW | P1 |
| Assistant panel UI (stub backend) | MEDIUM | MEDIUM | P1 |
| Home dashboard + FLIP | MEDIUM | MEDIUM | P1 |
| Named layouts | MEDIUM | LOW-MEDIUM | P1 |
| Fuller keyboard nav / command palette | LOW-MEDIUM | MEDIUM | P2 |
| Real AI backend | HIGH (long-term) | HIGH | P3 (own milestone) |
| Databasise integration | HIGH (long-term) | HIGH | P3 (own milestone) |
| Multi-window | LOW (for this product's users) | HIGH | P3 / defer indefinitely |
| Full a11y compliance | MEDIUM (long-term) | HIGH | P3 |

**Priority key:**
- P1: Must have for launch (matches PROJECT.md Active requirements, plus the completeness-check additions this research surfaced)
- P2: Should have, add when possible
- P3: Nice to have / explicitly deferred by prior user decision

## Competitor Feature Analysis

| Feature | VS Code | JetBrains Rider | Obsidian | Sourcerer's Approach |
|---------|---------|------------------|----------|----------------------|
| Docking model | Editor groups + fixed side/bottom panels, drag-to-split | Tool windows dockable to any edge, pinned/floating/docked modes | Panes + tabs, drag-to-split, mobile-adapted | 5-zone recursive split tree (center/L/R/top/bottom), closer to Rider's flexibility than VS Code's more rigid group model |
| Plugin/extension model | Full extension API (huge surface, versioned, marketplace) | Plugin SDK (Java/Kotlin, IDE-wide) | Community plugin API (JS, sandboxed-ish) | Deliberately minimal: 5-member `host` object + manifest, no marketplace, no versioning concerns yet — appropriate for v1 scope, would need to grow if a marketplace/ecosystem is ever a goal |
| Keyboard-first navigation | Extensive (F6 part-cycling, full command palette, chorded shortcuts) | Extensive (double-shift search-everywhere, IDE-wide shortcuts) | Moderate (hotkeys are user-configurable but not exhaustive by default) | Minimal specced set (⌘\, ⌘↵, y/d/n) — reasonable v1 subset; VS Code-level exhaustiveness correctly out of scope for now |
| Empty-state / dashboard | "Get Started" walkthrough page | Welcome screen with recent projects | Empty vault prompts, graph view as a pseudo-dashboard | Metro-card Home dashboard with living sections — more ambitious/opinionated than any of the three, genuine differentiator |
| Session persistence | Full workspace state restore (open editors, layout, terminal state) | Full project window-state restore | Full vault/workspace state restore | Matches the category standard; explicitly scoped to restore everything (ahead of the HTML prototype, which only persisted layouts + Notes) |
| Stub/preview of unbuilt features | N/A (extensions are either installed or not — no "coming soon" placeholder convention) | N/A | N/A (plugins are installed or not) | **Unique to this project** — no direct competitor precedent for shipping a fully-styled demo stub per unbuilt feature; this is a genuine point of differentiation worth protecting fidelity on |

## Sources

- VS Code Accessibility docs — [Accessibility](https://code.visualstudio.com/docs/editor/accessibility), [User Interface](https://code.visualstudio.com/docs/getstarted/userinterface) — MEDIUM confidence (official docs, general knowledge cross-check, not deeply verified against current version but focus/keyboard model has been stable for years)
- `design-sync-setup-guide/design_handoff_sourcerer_tauri/README.md` — HIGH confidence (authoritative project source-of-truth, read directly)
- `design-sync-setup-guide/design_handoff_sourcerer_tauri/reference/applets/README.md` — HIGH confidence (authoritative applet framework contract, read directly)
- `.planning/PROJECT.md` — HIGH confidence (authoritative project scope/decisions, read directly)
- General domain knowledge of JetBrains Rider tool-window docking, Obsidian pane/plugin model, JupyterLab docking (golden-layout-derived), golden-layout/dockview libraries — MEDIUM confidence (training-data based, cross-checked for internal consistency across products, not independently re-verified this session; flag for validation if precise behavioral claims about these specific products become load-bearing for a design decision)

---
*Feature research for: Dockable multi-pane desktop workbench + applet framework (Sourcerer)*
*Researched: 2026-07-06*
