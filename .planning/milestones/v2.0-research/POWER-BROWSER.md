# Power Browser — Component & Architecture Research

**Date:** 2026-07-09 · **Method:** deep-research workflow (5 search angles, 23 sources fetched, 114 claims extracted, 25 adversarially verified: 24 confirmed / 1 refuted) · **Consumed by:** future Power Browser phase (and KeyPass phase for §5)

**Requirement set** (from user + `04-DISCUSS-CHECKPOINT.json` items 31, 33–37): dual engine per-tab switchable (lightweight headless + full Chromium with extensions) · each browser tab is a dockview tab, one underlying browser runtime · tab hibernation/sleep · assistant pre-loads/observes/acts on tabs · least-effort assistant read/write surface · KeyPass boundary (assistant triggers autofill, never sees secret).

> **DECISION (user, 2026-07-09): Path A — WebView2, the engine Tauri already borrows.** No bundled second engine in v1. Extensions via a CRX side-load bridge (store-install limitation accepted as "not a big deal"). cef-rs and the ungoogled-chromium sidecar are the recorded fallbacks if the WebView2 spikes fail. Details in §1a; spikes reordered accordingly.

---

## 1. Engine options — verdict

### Full-rendering leg: **cef-rs (embedded Chromium) is the strongest verified path** — HIGH confidence, 3-0 verified

- [tauri-apps/cef-rs](https://github.com/tauri-apps/cef-rs) is an **official Tauri-organization project**: Windows x86_64 + ARM64 supported, latest release `cef-v149.3.0+149.0.6` (2026-06-28) tracking CEF 149, 312 releases, pushes as recent as 2026-07-10, not archived.
- Caveat (verifier-flagged): platform checkmarks assert *coverage*, not production maturity; many releases are automated CEF version bumps. A spike must prove it renders inside our window before committing.
- Closest prior art found in search (unverified, blog): Atrium — a shipping Tauri app embedding a CEF surface positioned behind/inside the app's own webview UI by punching a transparent visual hole through the React layer (`getatrium.dev/blog/embedding-real-browser-tauri`). This is exactly the "pane geometry/visibility host capability" seam Phase 4 reserved.
- WebView2 (Tauri's own Windows webview) can host arbitrary sites as tabs, but see §2 — extensions are side-load-only, and it's Edge-Chromium, not ungoogled. Verso/Servo and wry multi-webview: **no verified claims** — not disproven, just unassessed.

### Lightweight leg: **Lightpanda cannot be the v1 headless engine on Windows** — HIGH confidence, 3-0 verified

- **Headless-only by architecture**: no graphical rendering engine at all (no layout/rasterization/compositing) — it can never be a user-visible tab surface, and even `Page.captureScreenshot` is impossible. No GUI mode on any roadmap (checked April 2026).
- **Still Beta** (project's own warning: errors/crashes, incomplete Web API coverage).
- **No native Windows binary** — WSL2 only; nightlies cover Linux/macOS only. Native Windows "in development", not shipped. For a Windows-first app this means a WSL-hosted sidecar — a material complication.
- It **does** expose a CDP WebSocket (`lightpanda serve --host 127.0.0.1 --port 9222`), drivable by Puppeteer (`browserWSEndpoint`) and Playwright (`connectOverCDP`) — same transport as Chromium. But **coverage is partial (~17 domains)**; the vendor-page claim that its CDP "covers most commonly needed features" was **REFUTED 0-3**. 2026 sources recommend Puppeteer over Playwright against it.

### 1a. DECIDED — Path A: WebView2 as the v1 engine (user decision 2026-07-09)

Tauri bundles **no** engine; on Windows it borrows the OS-installed WebView2 (Edge-Chromium). Power Browser tabs = additional WebView2 webviews (wry multi-webview) positioned over dockview panes. Why this holds up against every requirement:

- **Assistant reads/acts:** WebView2 speaks full CDP (remote-debugging port on the shared browser process; puppeteer-core usable as client). A11y tree + DOM for cheap reads, screenshots, `Runtime.evaluate`. Note: all WebView2s in the app share one browser process *including the shell UI webview* — the assistant's CDP session must filter targets to Power Browser tabs only.
- **Assistant pops tabs:** host API creates a webview over a new pane (the Phase-4 pane-geometry capability), or hidden for background loads.
- **Background scraping:** no true headless mode — a background tab is a hidden/offscreen webview. Fine at assistant scale (pre-load N sources per session), and it has one property a separate puppeteer browser can't match: **hidden tabs share the user's profile/logins** (paywalled sources work with zero credential juggling). Ceiling: heavy parallel crawling wants a real headless backend — that's the abstract engine seam where Lightpanda slots in later.
- **Hibernation:** `TrySuspendAsync` (freeze) + discard, per §3.
- **Extensions:** side-load bridge, per §2 — accepted trade-off.
- **Bundle cost:** zero (engine ships with Windows).

**Fallback ladder if spikes fail:** (1) **cef-rs** embedded Chromium — the verified-healthy true-embedding path, ~150–200 MB bundle, extension status unproven; (2) **ungoogled-chromium as CDP sidecar** — see 1b.

### 1b. ungoogled-chromium assessed (repo inspected 2026-07-09) — ingredient, not component

[ungoogled-software/ungoogled-chromium](https://github.com/ungoogled-software/ungoogled-chromium) is **4.1 MB of patches, not a browser**: 108 `.patch` files + 36 Python build scripts + strip-lists (`flags.gn`: no Google API keys, safe-browsing off, reporting/remoting off), applied to the real ~40 GB Chromium tree at build time. Tracks Chromium 150.0.7871.114 (current). Consequences:

- **Not embeddable** — output is a standalone `chrome.exe`. Using it = separate process + CDP + window reparenting (the risky mechanic).
- **Never build it ourselves** — consume prebuilt binaries from `ungoogled-chromium-windows`; we ride their security-update cadence.
- **Extensions friction** — ungoogling *removes* Web Store integration; installs need the `chromium-web-store` companion or manual CRX. "Ungoogled + easy Google plugins" pulls against itself.
- **Cannot combine with CEF** — ungoogled-patches-on-CEF = maintaining a custom Chromium build; out of the question.

### Implication for the per-tab engine toggle

The toggle is **not** "same page, different renderer" — Lightpanda renders no pixels, so "send this tab to the zig browser" means *move the URL into a headless scraping context* (assistant/MCP territory), not an alternate view. The two engines share the CDP transport, which is what makes a common assistant surface feasible.

**Recommendation (superseded by the §1a decision, same logic):** single-engine v1 — WebView2 for everything, with the freeze/discard tiers of §3 delivering most of the memory win Lightpanda was wanted for (a discarded tab costs ~nothing). Keep the engine seam abstract (tab = URL + state routed to an engine backend) so Lightpanda can slot in as the scraping backend when it ships native Windows. Lightpanda-in-WSL2 is available sooner for pure assistant scraping jobs if wanted, but don't put it on the user-facing critical path.

## 2. Chrome extensions — no route gives Web-Store installs out of the box

| Route | Verified status |
| --- | --- |
| **WebView2** | Opt-in side-load of **unpacked** extensions only (`AreBrowserExtensionsEnabled=true` + `AddBrowserExtensionAsync`, folder with `manifest.json`; no CRX, no store). Microsoft stated (Aug 2023, unreversed in 2026 docs) store install **will not be supported** — licensing. An app *may* download+unpack CRX files itself and side-load. (3-0) |
| **Electron (stock)** | Arbitrary store extensions an explicit **non-goal** (reaffirmed Feb 2026, MV3 issue closed "not planned"). Only a DevTools-oriented API subset; unpacked-only; not persisted across restarts; `chrome.tabs` heavily limited, `chrome.storage` local-only. (3-0) |
| **Electron + community stack** | `electron-chrome-web-store` + `electron-chrome-extensions` (samuelmaddock/electron-browser-shell) is the only real Web-Store-install path — **maintained through mid-2025, quiet in the 12 months since** (not archived). (3-0, medium confidence on maintenance) |
| **CEF** | **OPEN — the deciding question.** CEF's Chrome runtime has extension support per a 2021 maintainer statement (Alloy runtime only a subset), but nothing about 2026 status or cef-rs exposure survived verification. **This is spike #1.** |

**Decided approach (Path A): the CRX side-load bridge.** Web Store extensions are CRX files on Google's CDN at a well-known URL pattern. Sourcerer does what `electron-chrome-web-store` does for Electron: user pastes a Web Store link → fetch CRX by extension ID → unpack → `AddBrowserExtensionAsync` into the WebView2 profile. Costs we own: update checking (re-fetch on schedule) and install UX (fits the applet model anyway). The real unknown is **runtime compatibility** — whether the user's must-have extensions actually function inside WebView2's extension runtime (not verified at full Chrome parity for every `chrome.*` API) → spike #1.

If the must-haves don't run and extensions stay a hard requirement, the fallbacks: CEF's chrome runtime (status unverified), or ungoogled-chromium as a **separate CDP-controlled process** (extensions installed normally in it, modulo §1b's store friction) with window reparenting — more moving parts, but extensions behave as in Chrome.

## 3. Tab memory lifecycle — solved problem, copy Chromium itself

The "one browser, N tabs, most hibernated, restore on click" mechanism **is Chromium's own, built-in**, routine since Chrome 108's Memory Saver (Dec 2022; unchanged through 2026 — Chrome 140 only swapped the heuristic for an ML model). Two tiers, both embedder-accessible (all 3-0 verified):

1. **FREEZE** — page paused (CPU/tasks stopped), content kept, instant resume. Saves CPU more than memory. Chromium `performance_manager` primitive; CDP `Page.setWebLifecycleState('frozen')` (experimental); WebView2 `TrySuspendAsync` ("similar to putting a tab to sleep in Edge" — Edge's Sleeping Tabs *is* this Chromium freeze tech, not Microsoft-proprietary).
2. **DISCARD** — content fully torn down (as if closed); title+favicon stay in the tab strip; auto-reload on activation. `chrome.tabs.discard` / embedder discard APIs. Caveat: reload is a full new navigation — JS state, form data, scroll lost unless the page opts into restoration (`document.wasDiscarded` for detection).

**Design lesson from Auto Tab Discard (the proven open-source prior art):** always use the engine's native discard, **never** DOM-replacement placeholder pages (The Great Suspender approach) — those retain DOM/listener memory and pollute history.

**Recommended lifecycle:** `active → frozen (after inactivity; instant restore) → discarded (after longer inactivity or memory pressure; reload on click)`, with per-tab pinning to exempt tabs. Dockview tab stays mounted (title/favicon chrome is ours); only the native surface hibernates.

## 4. Assistant observation/control surface — CDP, raw

Partially covered by verification; direction is clear:

- **CDP is the one surface that spans both engine legs** (verified: works against Chromium-anything and against Lightpanda at transport level). It is Chromium-only tech, but every candidate engine here speaks it.
- Extracted (not verified this run): **Browser Use dropped Playwright for raw CDP in Aug 2025** — Playwright's Node relay adds latency at agent call volumes; **Anthropic's Claude for Chrome is extension-based** (`chrome.debugger`-style), an alternative surface if we were extension-first, which we are not.
- For Sourcerer: the assistant sidecar speaks **raw CDP to the embedded engine** — `Target.createTarget` (pre-load session tabs), `DOM`/`Runtime`/`Accessibility` domains (read what the user views; a11y tree is the token-cheap representation), `Runtime.evaluate`/`Input` (act/inject). No Playwright/Puppeteer dependency needed against CEF; puppeteer-core is the fallback client if raw CDP proves tedious.
- **Not verified:** WebDriver BiDi maturity, `chrome.debugger` specifics, Playwright-against-CEF. Low priority given the CDP direction.

## 5. Password vault boundary — OPEN, needs its own pass (KeyPass phase)

**Zero claims survived verification** on this question (verify budget went to engines/extensions/lifecycle). Extracted leads worth the follow-up pass:

- **1Password "Secure Agentic Autofill"** (launched with Browserbase, Oct 2025): the agent only *signals* a credential is needed; the extension injects the secret directly into the browser; "the AI agent and underlying LLM never need to see nor handle the credentials." Closest commercial prior art for our exact requirement.
- **KeePassXC-Browser protocol**: native-messaging broker (`keepassxc-proxy` relaying stdio ↔ named pipes/Unix sockets to the vault) — the open-source broker-process pattern to copy.
- Sketch for Sourcerer: vault = separate trusted process (DPAPI/Windows Credential Manager at rest); assistant calls `host.keypass.fill(tabId, credentialHandle)`; the *vault process* performs the CDP `Input`/`DOM` injection on its own CDP session; the assistant's observation channel must not echo the typed value (mask password fields in DOM/a11y extraction). **Unverified design sketch — do not build without the dedicated research pass.**

## v1 architecture (DECIDED direction, per §1a)

1. **Engine:** **WebView2** — the engine Tauri already borrows; Power Browser tabs = additional webviews (wry multi-webview), one shared browser process, zero bundle cost. Engine seam kept abstract (tab = URL + state → engine backend) for a later Lightpanda/headless scraping leg. Fallbacks: cef-rs, then ungoogled-chromium sidecar.
2. **Docking:** each tab = dockview panel; webview positioned over the pane via the Phase-4-reserved pane-geometry host capability; hidden when the dock tab hides. Background/assistant tabs = hidden webviews sharing the user's profile/logins.
3. **Lifecycle:** active → frozen (`TrySuspendAsync`) → discarded, Chromium-native primitives; never DOM-replacement.
4. **Assistant:** raw CDP from the sidecar (remote-debugging port; puppeteer-core as optional client); a11y-tree-first reads; target-filter so the assistant never touches the shell UI webview; `Target.*` for session pre-loading.
5. **Extensions:** CRX side-load bridge (§2) — fetch/unpack/`AddBrowserExtensionAsync`, self-managed updates.
6. **KeyPass:** broker-process capability-handle design; dedicated research pass in the KeyPass phase (1Password agentic autofill + KeePassXC protocol as prior art).

## Cross-phase integration notes (checked against Phase 4 context 2026-07-09)

- **No Phase 4 decision changes.** The applet contract (D-01..D-19) is compatible as-is; the pane-geometry host capability stays a reserved additive seam.
- **`host.open` open-payload:** Power Browser tabs and assistant session pre-loading want "open applet with initial state" (URL per tab). Optional second param, purely additive — recorded in Phase 4's deferred ideas so nothing precludes `host.open(key, params?)`.
- **WebView2 environment flags are shell-owned:** `AreBrowserExtensionsEnabled` and the CDP remote-debugging port are set at *environment creation* — the same environment the shell UI webview uses, in `src-tauri` startup code owned by Phases 1/2. The Power Browser phase will add flags to shared shell setup (additive, but an ownership-boundary touchpoint like Phase 02↔07).
- **Overlay z-order:** native webviews render above the shell's DOM — shell overlays (pickers, drag ghosts, menus) can be obscured while a browser tab is visible. Mitigation (hide/clip webviews during overlay interactions) is Power Browser-phase work; folded into spike #2's scope.

## Open questions (ranked — these are the spikes)

1. **Do the user's must-have extensions actually run in WebView2?** Half-day spike: bare Tauri window, `AreBrowserExtensionsEnabled`, side-load 2–3 must-haves, see what breaks. This decides whether Path A survives. (Prerequisite: list the must-have extensions.)
2. **Webview-over-pane mechanics:** can a second WebView2 be positioned over a dockview pane in the Tauri 2 window with solid scroll/DPI/z-order behavior, and hidden/suspended cleanly on tab-hide? No verified claims on compositing mechanics.
3. **Vault boundary architecture** (§5) — zero verified claims; dedicated pass in the KeyPass phase.
4. **Lightpanda native Windows timeline + actual CDP domain coverage** (DOM/Runtime/Network/Accessibility) — vendor coverage claim was refuted; recheck when building the scraping leg (beta, fast-moving).
5. *(Fallback-only, only if spike #1 fails)* CEF chrome-runtime extension support in 2026 + cef-rs exposure.

## Refuted during verification

- "Lightpanda's CDP covers most commonly needed features" (lightpanda.io testimonial) — **0-3**. Treat coverage as partial.

## Key sources

Primary: [tauri-apps/cef-rs](https://github.com/tauri-apps/cef-rs) · [lightpanda-io/browser](https://github.com/lightpanda-io/browser) · [WebView2 AddBrowserExtensionAsync docs](https://learn.microsoft.com/en-us/dotnet/api/microsoft.web.webview2.core.corewebview2profile.addbrowserextensionasync?view=webview2-dotnet-1.0.2903.40) · [WebView2Feedback #3694](https://github.com/MicrosoftEdge/WebView2Feedback/issues/3694) · [Electron extensions docs](https://www.electronjs.org/docs/latest/api/extensions) · [electron-browser-shell](https://github.com/samuelmaddock/electron-browser-shell) · [Chrome Memory Saver blog](https://developer.chrome.com/blog/memory-and-energy-saver-mode) · [Auto Tab Discard](https://github.com/rNeomy/auto-tab-discard/) · [Edge Sleeping Tabs FAQ](https://techcommunity.microsoft.com/discussions/edgeinsiderannouncements/sleeping-tabs-faq/1705434) · [1Password agentic autofill](https://1password.com/blog/closing-the-credential-risk-gap-for-browser-use-ai-agents) · [KeePassXC-Browser](https://github.com/keepassxreboot/keepassxc-browser) · [Browser Use: Playwright→CDP](https://browser-use.com/posts/playwright-to-cdp)

All "current as of" statements checked 2026-07-09/10. Most likely to shift within months: Lightpanda (beta, Windows port in development) and cef-rs (fast release cadence).
