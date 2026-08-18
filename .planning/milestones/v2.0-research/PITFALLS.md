# Pitfalls Research

**Domain:** Tauri 2 frameless desktop shell (Windows-first), custom docking UI ported from HTML/React prototype
**Researched:** 2026-07-06
**Confidence:** MEDIUM-HIGH (Tauri/WebView2 issues verified via official GitHub issue tracker and docs; React/drag pitfalls are well-established community knowledge; project-specific numbers cross-checked against the design handoff)

## Critical Pitfalls

### Pitfall 1: Drag region swallows clicks on interactive title-bar children

**What goes wrong:**
`data-tauri-drag-region` only affects the exact element it's applied to — not descendants. If it's applied broadly (e.g. to the whole title bar `<div>` to save effort) instead of only the empty flex spacer, window control buttons, the layouts menu, and the logo/click-to-toggle-Home area become unclickable or behave inconsistently (single click drags instead of firing onClick, especially with fast clicks which OS interprets as a drag-start).

**Why it happens:**
Porting an HTML prototype's title bar markup 1:1 without re-verifying attribute placement; the prototype may rely on browser-only behavior not present in the Tauri WebView2 drag-region implementation. Also a known rough edge: `[bug] Window without decorations has buggy window controls` (tauri-apps/tauri#7388) and general reports that buttons inside/adjacent to drag regions swallow clicks.

**How to avoid:**
Apply `data-tauri-drag-region` *only* to the literal spacer element (per handoff: "the empty flex spacer only"), never to a parent wrapping the logo, applet-name label, layouts menu, or the three window-control zones. Keep window-control buttons in a separate non-drag sibling with normal pointer-events. Test every title-bar interactive element (minimize/maximize/close, layouts menu, rail-toggle icons, logo click) manually after wiring, not just visually.

**Warning signs:** Buttons need a "double-click" or "click-and-hold" to register; window sometimes starts dragging when user meant to click a button; layout menu never opens.

**Phase to address:** Phase 1 (frameless window + custom title bar scaffold) — verify interactively before moving on to dock tree work.

---

### Pitfall 2: Maximize covers the taskbar / disables taskbar interaction on Windows

**What goes wrong:**
With `decorations: false`, clicking maximize on Windows can cause the window to cover the full screen including the taskbar, and until the window is un-maximized the taskbar becomes unresponsive to clicks. There's also no maximize/restore/minimize animation, and multi-monitor setups can get the wrong size (Tauri sizes to the monitor that had focus at launch, not the one the window is being maximized on).

**Why it happens:**
Windows' native maximize logic accounts for the taskbar based on window "chrome" flags that a fully frameless (`decorations:false`) window doesn't have; Tauri (WebView2/winit) doesn't automatically apply the "maximized but respects work area" adjustment that a decorated window gets for free. Documented: tauri-apps/tauri#7103, #14025, #6843.

**How to avoid:**
Handle the maximize toggle carefully: use Tauri's `toggleMaximize` API as specified in the handoff, but manually test against the actual taskbar auto-hide setting and multi-monitor setup on the target dev machine early. If the bug reproduces, apply the commonly-used workaround: track maximize state yourself and use `setSize`/`setPosition` against `availableMonitorSize()` (excludes taskbar) rather than relying solely on native maximize, or pin to a specific Tauri 2.x patch version known to have this fixed (check the changelog before locking `tauri` version).

**Warning signs:** Taskbar icons stop responding to clicks while app is maximized; app visually overlaps taskbar; restoring from maximize snaps to wrong monitor or size.

**Phase to address:** Phase 1 (window shell). Add an explicit manual QA step: maximize/restore/minimize on primary and secondary monitor, with taskbar auto-hide both on and off.

---

### Pitfall 3: WebView2 renders blurry/fuzzy at non-100% Windows display scaling (125%/150%)

**What goes wrong:**
On HiDPI or scaled displays, WebView2 can render at the wrong DPI awareness level, causing blurry text, misaligned 1px borders, and subpixel artifacts — directly undermining a "pixel-perfect, 1px borders, 0 border-radius" spec. Root cause: if the process isn't declared Per-Monitor-V2 DPI aware, Windows reports 96 DPI regardless of actual scaling, and WebView2 has to interpolate.

**Why it happens:**
Desktop shells assume the OS/webview handles DPI scaling correctly by default; Tauri/WebView2 needs explicit DPI-awareness configuration (manifest or `SetProcessDpiAwarenessContext`) which is easy to skip since it's invisible at 100% scaling (the default dev environment) and only surfaces later when a user or QA machine runs 125%/150% (the Windows default on most laptops).

**How to avoid:**
Verify Tauri 2's bundled manifest declares `PerMonitorV2` DPI awareness (check `tauri.conf.json` / the generated Windows manifest — Tauri 2 sets this by default in recent versions, but confirm the exact version in use). Test the actual pixel-fidelity spec (1px borders, 34px bars) at 100%, 125%, and 150% scaling on a real Windows 11 machine before declaring any UI phase "pixel-perfect done." Avoid relying on subpixel (non-integer) CSS values for borders/spacing — round to device pixels where scaling is a known factor.

**Warning signs:** Design review looks perfect on the dev's own monitor but "off" on a laptop with default scaling; 1px borders appear as 2px or invisible depending on zoom; screenshots at different scale settings show shifted alignment.

**Phase to address:** Phase 1 shell setup (verify DPI awareness config) and again explicitly in whichever phase claims pixel-fidelity sign-off (dock tree / title bar) — don't treat "looks right on my machine" as done.

---

### Pitfall 4: React 18 StrictMode double-invokes effects that wire up global pointer listeners for the drag system

**What goes wrong:**
The ported dock/rail drag algorithm likely attaches `pointermove`/`pointerup` listeners (or a rAF loop) in a `useEffect` during drag-start. In development, React 18 StrictMode mounts → unmounts → remounts every component once, running effects twice. If the drag-start effect doesn't have a correct cleanup function, this can double-attach global listeners, causing drag previews to render twice, ghost drag state after drop, or listeners that never get removed and leak across pane/tab lifecycles (especially with the prototype's mutable "dock tree" object model).

**Why it happens:**
The HTML prototype has no StrictMode (browser scripts run components once); porting the drag logic "near 1:1" without adding proper `useEffect` cleanup functions is an easy oversight, and the bug frequently doesn't show up until a user does rapid drag operations or reopens/re-docks panes many times (the leaked listener count grows), which QA in a short dev session may miss.

**How to avoid:**
Every effect that adds global listeners (`window.addEventListener('pointermove'/'pointerup', ...)`) or starts a rAF loop for drag tracking must return a cleanup function that removes the listener/cancels the frame. Prefer starting the actual listener attachment inside the `pointerdown` handler itself (imperative, not effect-driven) rather than in a `useEffect` — this sidesteps StrictMode remount concerns entirely and matches how the vanilla-JS prototype worked. Keep StrictMode enabled in dev specifically so these bugs surface immediately rather than shipping to test-in-production.

**Warning signs:** Dock/tab drag preview flickers or duplicates in dev but not "in production build"; listeners increment (checkable via a dev-only counter) each time a drag starts; occasional drag operations affect two panes at once.

**Phase to address:** Phase 2 (port dock tree + rail algorithms) — this is the single highest-risk phase for silent bugs since the algorithm is "proven sound" in the prototype but the React lifecycle context is entirely new.

---

### Pitfall 5: Pointer capture not used, causing drag to break when the cursor leaves the source element

**What goes wrong:**
Mouse/pointer-based drag implementations that rely on `mousemove`/`pointermove` bound only to the dragged element (rather than `window` or via `setPointerCapture`) lose tracking the instant the cursor moves fast enough to exit the element's bounding box mid-drag — very likely during tab drags across a 34px tab bar or rail reordering, given the small hit targets (34px bars, 36px rows) in this design.

**Why it happens:**
The HTML prototype may already bind listeners to `window`/`document` (a common vanilla-JS pattern), but "porting near-1:1" risks subtly changing the binding target (e.g., binding to the row/tab element instead) during the translation to React event handlers/refs, especially if a dev naively uses React's synthetic `onPointerMove` prop on the source element rather than an imperative `window` listener or `element.setPointerCapture(pointerId)`.

**How to avoid:**
On `pointerdown`, call `element.setPointerCapture(event.pointerId)` so all subsequent pointer events route to that element regardless of cursor position, OR explicitly attach `pointermove`/`pointerup` to `window`/`document` for the duration of the drag (matching whichever pattern the prototype's `support.js` actually uses — read it first, don't assume). Always pair with a `pointercancel` handler and release capture / remove listeners on drag end to avoid stuck-drag states.

**Warning signs:** Dragging a tab or rail item quickly "drops" it mid-air, cursor detaches from the drag preview; drag works fine with slow mouse movement in manual testing but breaks under normal fast usage.

**Phase to address:** Phase 2 (dock tree + rail port).

---

### Pitfall 6: 5px drag threshold implemented with a blocking preventDefault / non-passive listener, breaking scroll/click feel

**What goes wrong:**
The spec calls for "all drags use a 5px movement threshold before engaging; plain click = activate." If this threshold check is implemented by calling `preventDefault()` inside a `pointermove`/`touchmove` listener registered without `{ passive: false }` explicitly set (or worse, relies on default passive listeners), the browser may throw a console warning and ignore the `preventDefault()`, or (if made non-passive) introduce jank on every pointer move even outside a drag, because the browser can no longer optimistically scroll/composite.

**Why it happens:**
Modern browsers/WebView2 default `touchmove`/`wheel` listeners to passive for performance; devs copy a plain HTML prototype's addEventListener calls without re-checking passive/active semantics, or apply `touch-action: none` globally instead of scoping it to draggable handles, killing native scroll/pinch behavior elsewhere in the shell (e.g., session list, message thread scrolling in the Assistant panel).

**How to avoid:**
Use `touch-action: none` (CSS) scoped only to drag handles/grips (resize grips, tab drag sources, rail item drag handles) rather than `preventDefault()` in JS — this is the documented best-practice replacement and avoids passive-listener conflicts entirely. Reserve any non-passive listener usage for the rare case CSS `touch-action` can't express the constraint, and register those explicitly with `{ passive: false }`.

**Warning signs:** Console warnings about "Ignored attempt to cancel a touchmove event"; scrolling in the Assistant message thread or rail feels janky after adding drag; pinch-zoom or scroll gestures stop working on Windows touchscreens (relevant since decorations:false + touch is also flagged as buggy upstream — tauri-apps/tauri#4746).

**Phase to address:** Phase 2 (dock/rail drag port), verified again in Phase 3 (Assistant panel resize grip) since it introduces its own drag handle.

---

### Pitfall 7: tauri-plugin-store writes are debounced and not crash-safe — losing recent workspace state

**What goes wrong:**
`tauri-plugin-store`'s default `autoSave` debounces writes (100ms default) and, per an open upstream issue, `LazyStore` files have a real chance of corruption (truncated to null bytes, or fully emptied) if the app doesn't terminate gracefully shortly after a `set()` call — e.g., OS force-close, crash, or power loss during a maximize/resize burst that's triggering rapid workspace-state writes. Given this project's requirement to "persist the whole workspace on change" (dock tree, rail state, widths, tabs), the write frequency is high, increasing exposure.

**Why it happens:**
Store plugin is a thin JSON-file-per-key wrapper without atomic write-then-rename semantics guaranteed across all paths; combined with "persist on every change" (vs. debounced/batched), a user resizing a dock split rapidly can trigger many disk writes racing an unexpected termination.

**How to avoid:**
Debounce workspace-state persistence at the application level (e.g., 250–500ms after the last change, not on every pixel of a resize drag) rather than relying only on the plugin's own debounce. On load, validate the loaded JSON against a schema/shape check (see Pitfall 8) and fall back to a last-known-good snapshot or default empty workspace rather than crashing if the file is corrupt/truncated. Consider writing a `.bak` copy of the last successfully-parsed store state before overwriting, so a corrupted write doesn't destroy the only copy. Track the plugins-workspace issue (tauri-apps/tauri#3085 area) for a fix landing in a future plugin version and pin/upgrade deliberately rather than silently floating on `^` version ranges.

**Warning signs:** After a crash/force-quit during testing, workspace reopens to default/empty state or throws a JSON parse error; store file on disk is 0 bytes or full of null bytes after an abrupt shutdown.

**Phase to address:** Phase 4/5 (workspace persistence) — build the load-time validation and defensive fallback into the very first persistence implementation, not as a later hardening pass.

---

### Pitfall 8: Schema drift of the persisted dock tree breaks restore-on-launch

**What goes wrong:**
The dock tree is a recursive structure (`leaf = {tabs[], active}` / `split = {dir, sizes[], children[]}`) with named layouts also persisted. As the applet framework grows (new applet keys, registry changes, Notes → other applets), a previously-saved dock tree or named layout can reference an applet `key` that no longer exists, or a tree shape from an earlier schema version, causing the restore step to either crash on load or silently render a broken/empty pane where a real tab used to be.

**Why it happens:**
No versioning or key-existence validation was designed into "restore tree on launch" in the handoff — it's described as pure round-trip persistence. Any future applet rename, stub-to-real-implementation swap, or intentional dock-tree shape change (e.g., adding a new zone/property) invalidates old saved state with no migration path.

**How to avoid:**
Store a schema/version number alongside the persisted workspace state and named layouts. At load time: (1) validate the tree recursively (every leaf's tab `key` exists in the current registry; every split's `children` array is non-empty and `sizes` sums sanely); (2) if a referenced applet key is missing, replace that tab with the standard stub or a "removed applet" placeholder rather than crashing; (3) if the version doesn't match, either run a migration function or discard to a safe default (empty workspace / single Home pane) with a one-time warning rather than a hard failure. Add this validation the same phase persistence is introduced, not deferred to "later hardening."

**Warning signs:** App fails to launch (blank white screen) after an applet is renamed/removed during development; renaming a registry key during Notes-applet work breaks previously-saved test layouts; QA has to manually delete the store file to unblock testing after schema changes.

**Phase to address:** Phase 4/5 (workspace persistence) and again whenever the applet registry changes shape (ongoing, but flag explicitly in the Notes-applet phase since it's the first registry mutation after initial stub setup).

---

### Pitfall 9: Font bundling done wrong — CORS/404 fallback to system fonts or licensing omission

**What goes wrong:**
Two distinct failure modes: (1) fonts referenced via a CDN/Google Fonts URL (even accidentally left over from prototype CSS) silently fail in a Tauri WebView with no network, or worse, work in dev when online but break for offline/restricted users, falling back to a system font and quietly breaking the "pixel-perfect" type scale; (2) IBM Plex is OFL-licensed and permissive, but bundling still requires including the license file — omitting it is a compliance/attribution miss even though it doesn't block functionality.

**Why it happens:**
The prototype (a browser HTML file) may have loaded IBM Plex from Google Fonts or a CDN `<link>` for convenience during design; porting this reference file "near 1:1" without auditing every `@font-face`/`<link>` risks carrying that network dependency into the shipped app, which the handoff explicitly forbids ("No Google Fonts at runtime — bundle fonts locally").

**How to avoid:**
Grep the prototype HTML/CSS for any `fonts.googleapis.com`, `fonts.gstatic.com`, or other remote font URLs before porting; replace every one with local `@font-face` rules pointing at bundled `.woff2` files shipped in the app's asset directory (loaded via Vite's static asset pipeline, not a CDN). Include the IBM Plex OFL license file in the repo/bundle. Test the app with network disabled to confirm no remote font requests occur (check WebView2 devtools network tab).

**Warning signs:** Text renders differently (or as a generic sans-serif) when running with no internet connection; devtools network tab shows requests to fonts.googleapis.com/gstatic.com at runtime.

**Phase to address:** Phase 1 (shell scaffold, since fonts underlie every screen) — verify with network disabled before any pixel-fidelity sign-off.

---

### Pitfall 10: Applet framework contract drift — bypassing `host` API or `React`-via-props indirection mismatch

**What goes wrong:**
The prototype's applet contract passes `React` as a prop (`App({React, host})`) because it's a vanilla script without a bundler; the handoff explicitly says to "drop the React-via-props indirection if using a bundler" when porting to real TSX modules. If this isn't done consistently, or if some applets end up importing `React` normally while the registry loader still expects the old signature, applets silently fail to render or double-import React (React 18 hook errors, "Invalid hook call" from two React copies). Separately, if any future applet reaches past `host.storage`/`host.ai()` directly (e.g., calls `localStorage` or a raw HTTP fetch), it breaks the single-seam guarantee the whole framework is built on and creates untestable, unstubbable behavior that can't be swapped later when the AI backend or Databasise integration is decided.

**Why it happens:**
Two different module worlds (vanilla-script prototype vs. bundled TSX) get merged carelessly during "port near-1:1," and there's no compiler-enforced boundary preventing an applet module from importing outside `host` — nothing stops a rushed implementation of a later applet from taking a shortcut once the "it's just a demo stub anyway" mentality sets in.

**How to avoid:**
Decide the final applet contract signature once at the very start (drop `React` prop, use standard ESM imports; keep `manifest` + `App({host})` only) and update `_TemplateApplet` equivalent + registry loader to match before building Notes. Add a lightweight lint rule or code-review checklist item: applet modules may only import from `../host` (or equivalent) plus React/UI primitives — no `fetch`, no `localStorage`, no direct Tauri `invoke` calls. Treat this as an architectural constraint enforced at Notes-applet time, since Notes sets the pattern every future applet copies.

**Warning signs:** "Invalid hook call" errors when Notes or a second applet is added; an applet's data doesn't survive between sessions because it wrote to `localStorage` instead of `host.storage`; AI seam ends up hardcoded to a specific backend inside an applet instead of routed through `host.ai()`.

**Phase to address:** Phase 3 (applet framework: registry loader + host API) — lock the contract shape here; Phase 4 (Notes applet) is the first real test of the constraint holding.

---

### Pitfall 11: `cargo tauri dev` vs `cargo run` from `src-tauri` diverge in dev-loop behavior

**What goes wrong:**
`cargo run` from `src-tauri` bypasses the Tauri CLI's config merging, `beforeDevCommand` (frontend dev server auto-start), and file-watching/auto-rebuild wiring that `cargo tauri dev` provides. A dev workflow that mixes the two inconsistently (e.g., docs/scripts say one thing, a teammate or future-you runs the other) causes confusing symptoms: frontend changes not hot-reloading, stale `tauri.conf.json` values, or the frontend dev server never starting because `beforeDevCommand` was skipped.

**Why it happens:**
This project's own prior-phase notes (Databasise/Sourcerer packaging work) already flagged detached-launch and `cargo run`-vs-`cargo tauri dev` friction on Windows for a sibling project; the same two-command ambiguity applies here and is worth deciding explicitly rather than rediscovering per-session.

**How to avoid:**
Pick one canonical dev command for this repo's `README`/scripts (likely `cargo tauri dev` for the standard hot-reload loop, since it owns `beforeDevCommand`/frontend proxy wiring — reserve raw `cargo run` for narrow Rust-only debugging where the frontend is already built/static). Document the choice explicitly in project setup so it isn't rediscovered ad hoc each session.

**Warning signs:** "It worked in the other terminal but not this one"; frontend changes don't appear without a manual restart; window opens with a blank page because the Vite dev server never started.

**Phase to address:** Phase 1 (initial scaffold) — decide and document immediately.

---

### Pitfall 12: Windows paths with spaces break Tauri shell/command invocations and dev tooling

**What goes wrong:**
Tauri has open, confirmed issues where paths containing spaces are mishandled: `shell.open()` fails on Windows paths with spaces, and `new Command()` invocations get the path silently truncated at the first space. The repo itself already lives under `D:\Vibe Coding\Sourcerer` (a space in the parent path) and the design handoff sits under `design-sync-setup-guide\...` (also spaced) — any future feature that shells out (opening a file, invoking an external process, referencing the reference HTML/JS assets programmatically) is at risk if not tested against this exact path.

**Why it happens:**
The repo/workspace path was already chosen with spaces (consistent with the rest of the user's Windows filesystem layout); this isn't something to "fix" by renaming, but every use of Tauri's `Command`/`shell` APIs (or any Rust `std::process::Command` calls) needs explicit quoting/escaping awareness rather than assuming it "just works" like it does on paths without spaces.

**How to avoid:**
When adding any Tauri shell/command invocation (e.g., "open file in default app," future external tool integration), test it specifically against the actual spaced repo path, not just a `C:\dev\project` no-space path. Prefer Rust's own `std::process::Command` (which handles argument arrays natively, avoiding shell string-concatenation entirely) over Tauri's JS-side `Command`/`shell.open` API where the known bugs live, when a choice exists.

**Warning signs:** A file-open or external-process feature works on a CI/clean machine path but fails for the actual dev on their own machine; path gets silently truncated (only `D:\Vibe` gets used, `Coding\Sourcerer` dropped).

**Phase to address:** Whichever phase first introduces any shell-out / external-process / file-system-path Tauri command (likely later, e.g., an applet that opens files) — flag it now so it isn't a surprise.

---

## Technical Debt Patterns

| Shortcut | Immediate Benefit | Long-term Cost | When Acceptable |
|----------|-------------------|----------------|-----------------|
| Persist workspace state on every single state change with no debounce | Simple to implement first | Excess disk I/O during drag/resize, higher corruption exposure (Pitfall 7) | Never past initial prototype spike — debounce before Phase 4/5 sign-off |
| Keep `React`-via-props applet signature "for now" instead of migrating to standard ESM | Saves an hour of contract redesign | Every applet built after Notes inherits the wrong pattern; painful global rename later | Never — fix before Notes applet (Phase 3/4) |
| Skip dock-tree schema versioning "since it's greenfield, no real data yet" | Faster persistence implementation | First registry rename (near-certain once applets other than Notes are built) breaks all saved test layouts | Acceptable only for the very first internal spike; must be added before any milestone claims persistence "done" |
| Test pixel-fidelity only at 100% Windows scaling (dev default) | Faster QA loop | Ships blurry/misaligned UI to the (likely majority of) users on 125%/150% laptops | Never for a "pixel-perfect" spec — always test at least one scaled config per UI phase |
| Leave `data-tauri-drag-region` on a coarse wrapper instead of the precise spacer | Faster initial title bar wiring | Buttons/menu become flaky-clickable, hard-to-reproduce bug reports later | Never — costs almost nothing to do correctly the first time |

## Integration Gotchas

| Integration | Common Mistake | Correct Approach |
|-------------|----------------|-------------------|
| `tauri-plugin-store` | Treating `.set()` as durable/atomic; relying solely on plugin's internal debounce | App-level debounce + load-time schema/shape validation + defensive fallback on parse failure |
| WebView2 (Windows) | Assuming default DPI awareness "just works" like a native Win32 app | Explicitly verify/declare Per-Monitor-V2 DPI awareness before any pixel-fidelity claim |
| Tauri window API (`minimize`/`toggleMaximize`/`close`) | Wiring buttons without testing maximize against real taskbar auto-hide + multi-monitor | Manual QA matrix: primary/secondary monitor × taskbar auto-hide on/off × maximize/restore/minimize |
| Applet registry / `host` API | Allowing an applet to import outside `host` "just this once" | Enforce via code review checklist (and ideally a lint rule) that applet modules only touch `host` + UI primitives |
| Prototype `support.js` reference | Assuming its event-binding pattern (window vs. element-level listeners) without reading it | Read `support.js`'s actual drag implementation before writing the React port — copy its listener-target choice deliberately |

## Performance Traps

| Trap | Symptoms | Prevention | When It Breaks |
|------|----------|------------|----------------|
| Per-pixel workspace-state persistence during resize drags | Disk thrashing, UI stutter during resize, corruption risk climbs | Debounce persistence writes (250–500ms after drag ends, not per-frame) | Immediately noticeable once resize/drag is implemented, worsens with more panes/tabs open |
| rAF loop for drag tracking left running after drag ends (leak from StrictMode double-effect, Pitfall 4) | Idle CPU usage creeps up the longer a session runs, especially after many drag operations | Always cancel rAF / remove listeners in effect cleanup or on pointerup/pointercancel | Noticeable after tens of drag operations in a long-running session; easy to miss in a short QA pass |
| Deeply nested dock-tree re-render on every drag-hover frame (recomputing hit-test/prune across the whole tree) | Dock preview UI feels laggy once more than a few splits/panes exist | Memoize hit-test computation to the relevant subtree; avoid full-tree re-render per pointermove | Scales with number of open splits/tabs; fine at 2-3 panes, degrades by ~6-8 nested splits |

## Security Mistakes

| Mistake | Risk | Prevention |
|---------|------|------------|
| Applet bypasses `host.ai()` and calls an AI backend directly with an embedded key | Credential/key leakage into an individual applet module, no central control point when backend changes | Enforce single-seam architecture (Pitfall 10); never let an applet hold its own API key |
| Tauri `shell`/`Command` API used with unsanitized user-provided paths (e.g., future "open file" applet feature) | Command injection or path-traversal via unescaped arguments, compounded by the spaces-in-path bug (Pitfall 12) | Prefer Rust-side `std::process::Command` with argument arrays; validate/sandbox any user-facing path input |

## UX Pitfalls

| Pitfall | User Impact | Better Approach |
|---------|-------------|-------------------|
| Maximize/restore has no animation and can visually "jump" over the taskbar (Pitfall 2) | Feels broken/unpolished compared to native Windows apps, undermines "pixel-perfect" positioning | Test and, if needed, implement a lightweight custom fade/resize transition; at minimum ensure correct final sizing against the work area |
| 5px drag threshold not felt correctly (fires as a click or as a drag when it shouldn't) | Users misdrag tabs/rail items when trying to click, or fail to initiate a drag when intended | Implement and manually test threshold logic exactly as the prototype does (read `support.js`), don't approximate with a different pixel value or timing-based heuristic |
| Restoring an invalid/corrupted persisted workspace silently drops the user's layout with no explanation | User loses named layouts/session state with no feedback, appears as random data loss | Surface a one-time, non-blocking notice ("workspace couldn't be restored, starting fresh") rather than silently discarding on validation failure |

## "Looks Done But Isn't" Checklist

- [ ] **Title bar wired to window API:** Often missing multi-monitor + taskbar-auto-hide testing — verify maximize/restore on a secondary monitor and with "Automatically hide the taskbar" both on and off (Pitfall 2)
- [ ] **Pixel-perfect fidelity sign-off:** Often only checked at 100% Windows scaling — verify 1px borders and 34px bar heights render correctly at 125% and 150% scaling on a real machine (Pitfall 3)
- [ ] **Drag/dock system "ported 1:1":** Often missing StrictMode-safe cleanup and pointer-capture — verify no duplicate/leaked listeners after 20+ repeated drag operations in dev mode (Pitfalls 4, 5)
- [ ] **Workspace persistence "working":** Often untested against corruption/crash — verify app recovers gracefully (not a crash or blank screen) after force-killing the process mid-write, and after manually editing the store JSON to reference a nonexistent applet key (Pitfalls 7, 8)
- [ ] **Fonts "bundled locally":** Often still has a leftover CDN `<link>` from the prototype — verify with network disabled that no font requests hit Google Fonts/gstatic (Pitfall 9)
- [ ] **Applet framework "host API enforced":** Often has at least one shortcut import bypassing `host` — grep every applet module for `fetch(`, `localStorage`, or direct `invoke(` calls outside the `host` implementation (Pitfall 10)

## Recovery Strategies

| Pitfall | Recovery Cost | Recovery Steps |
|---------|---------------|-----------------|
| Drag region swallows clicks (Pitfall 1) | LOW | Re-scope `data-tauri-drag-region` to the exact spacer element; re-test each title-bar control |
| Maximize covers taskbar (Pitfall 2) | MEDIUM | Add manual maximize handling against `availableMonitorSize()`; verify across Tauri version bump/patch notes |
| Blurry UI at scaled DPI (Pitfall 3) | MEDIUM | Add/confirm PerMonitorV2 manifest declaration; re-run pixel-fidelity QA pass at 125%/150% |
| Leaked drag listeners (Pitfall 4/5) | LOW-MEDIUM | Add cleanup functions / move listener attachment to `pointerdown` handler instead of `useEffect`; add a dev-only listener-count assertion |
| Corrupted/lost persisted store (Pitfall 7) | MEDIUM | Add app-level debounce + `.bak` snapshot + load-time try/catch with default-workspace fallback; cannot recover already-lost user data, only prevent recurrence |
| Broken restore from schema drift (Pitfall 8) | MEDIUM | Add version tag + migration/validation function; for already-broken saved layouts, ship a one-time "reset workspace" affordance |
| Leftover remote font dependency (Pitfall 9) | LOW | Replace `<link>`/CDN `@font-face` with bundled local `.woff2` files; re-test offline |
| Applet bypassing `host` API (Pitfall 10) | LOW-MEDIUM (per applet) | Refactor the offending applet to route through `host`; add lint/review gate to prevent recurrence |

## Pitfall-to-Phase Mapping

| Pitfall | Prevention Phase | Verification |
|---------|------------------|---------------|
| 1. Drag region swallows clicks | Phase 1 (frameless window + title bar) | Manually click every title-bar interactive element after wiring |
| 2. Maximize covers taskbar | Phase 1 (frameless window + title bar) | Manual QA matrix: monitor × taskbar-autohide × maximize/restore/minimize |
| 3. Blurry UI at Windows scaling | Phase 1 (shell scaffold, DPI config) + re-verified at any pixel-fidelity sign-off | Visual check at 100/125/150% scaling on real hardware |
| 4. StrictMode double-effect listener leaks | Phase 2 (dock tree + rail port) | Dev-mode listener-count check after repeated drags; StrictMode left ON |
| 5. Missing pointer capture breaks fast drags | Phase 2 (dock tree + rail port) | Manual fast-drag test across tab bars/rail |
| 6. Non-passive/preventDefault drag threshold breaks scroll | Phase 2 (dock/rail) + Phase 3 (Assistant resize grip) | Check devtools console for passive-listener warnings; verify unrelated scroll areas still scroll |
| 7. Store writes not crash-safe | Phase 4/5 (workspace persistence) | Force-kill process mid-write in QA; confirm graceful fallback, not crash |
| 8. Dock-tree schema drift breaks restore | Phase 4/5 (workspace persistence) + re-checked whenever registry changes | Manually corrupt/rename a saved layout's applet key; confirm graceful placeholder, not crash |
| 9. Font bundling/CDN leftover | Phase 1 (shell scaffold) | Network-disabled test, check devtools network tab |
| 10. Applet `host` API bypass | Phase 3 (applet framework) + Phase 4 (Notes applet as first real test) | Grep all applet modules for disallowed imports each time an applet is added |
| 11. `cargo tauri dev` vs `cargo run` ambiguity | Phase 1 (scaffold) | Document canonical command in repo setup instructions |
| 12. Windows spaced-path breaks shell/Command calls | Whichever phase first shells out / opens files | Test against the actual `D:\Vibe Coding\Sourcerer` path, not a clean no-space path |

## Sources

- [Window Customization | Tauri v2 docs](https://v2.tauri.app/learn/window-customization/) — drag region attribute semantics
- [tauri-apps/tauri#7388 — Window without decorations has buggy window controls](https://github.com/tauri-apps/tauri/issues/7388)
- [tauri-apps/tauri#11945 — Unmaximize by double-clicking drag region resizes incorrectly](https://github.com/tauri-apps/tauri/issues/11945)
- [tauri-apps/tauri#7103 — Windows taskbar disabled when maximizing decorations:false window](https://github.com/tauri-apps/tauri/issues/7103)
- [tauri-apps/tauri#14025 — Maximized window doesn't take up entire screen space](https://github.com/tauri-apps/tauri/issues/14025)
- [tauri-apps/tauri#6843 — window size incorrect under multiple monitors](https://github.com/tauri-apps/tauri/issues/6843)
- [tauri-apps/tauri#8383 — window doesn't always restore previous dimensions](https://github.com/tauri-apps/tauri/issues/8383)
- [tauri-apps/tauri#4746 — drag-region doesn't work correctly on Windows touchscreen](https://github.com/tauri-apps/tauri/issues/4746)
- [tauri-apps/tauri#1074 — High DPI scaling: WebView2 on Windows blurry](https://github.com/tauri-apps/tauri/issues/1074)
- [MicrosoftEdge/WebView2Feedback#1700 — WebView2 not scaling on high-DPI monitor](https://github.com/MicrosoftEdge/WebView2Feedback/issues/1700)
- [tauri-apps/plugins-workspace#3085 — LazyStore chance of corrupting persisted file on power loss/crash](https://github.com/tauri-apps/plugins-workspace/issues/3085)
- [Store | Tauri v2 plugin docs](https://v2.tauri.app/plugin/store/) — autoSave debounce default
- [tauri-apps/tauri#6431 — shell.open() doesn't work with spaced paths on Windows](https://github.com/tauri-apps/tauri/issues/6431)
- [tauri-apps/tauri#7914 — new Command() truncates path with spaces](https://github.com/tauri-apps/tauri/issues/7914)
- [Tauri CLI reference / Develop docs](https://v2.tauri.app/develop/) — `cargo tauri dev` vs `cargo run` behavior
- [facebook/react#24502 — useEffect double invocation in StrictMode](https://github.com/facebook/react/issues/24502)
- [Creating drag interactions with setPointerCapture — r0b blog](https://blog.r0b.io/post/creating-drag-interactions-with-set-pointer-capture-in-java-script/)
- [pmndrs/use-gesture#264 — disabling setPointerCapture on drag events](https://github.com/pmndrs/use-gesture/issues/264)
- Project design handoff: `design-sync-setup-guide/design_handoff_sourcerer_tauri/README.md` (this repo) — authoritative spec for metrics, tokens, drag behaviors referenced throughout
- Project memory: prior Sourcerer/Databasise sibling-project notes on `cargo run` vs `cargo tauri dev` and detached-launch Windows gotchas (informs Pitfall 11, MEDIUM confidence — carried from adjacent project experience, not independently re-verified for this repo)

---
*Pitfalls research for: Tauri 2 frameless desktop shell with custom docking UI (Windows-first)*
*Researched: 2026-07-06*
