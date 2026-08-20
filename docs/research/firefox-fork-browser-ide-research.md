# Research Record: Custom Browser+IDE — Electron vs Firefox Fork (2026-08-19)

## Question
Build a web browser (eventually hosting Theia IDE) — Electron's limits, and whether a Firefox fork with a rebuilt GUI is the better substrate.

## Electron-as-browser limitations
1. **No Widevine DRM** — Netflix/Disney+/Spotify-web etc. won't play. Fix: Castlabs "Electron for Content Security" fork + VMP code-signing (L3 only, so 720p/SD caps on some services). Headless-browser relay tricks do NOT work: headless Chrome/Firefox disable the CDM, protected video captures as black frames, and capture would be DMCA §1201 circumvention anyway.
2. **Security ceiling** — with sandbox/contextIsolation/process-per-tab you get near Chromium-renderer parity, but structurally you can't match Chrome: patch latency (Electron release cycle vs Chrome's days-fast 0-day fixes), no Safe Browsing, weaker browser-layer site-isolation enforcement. Fine for personal/embedded browsing; not Chrome-grade against fresh renderer exploits.
3. Partial extensions API only (uBlock etc. mostly broken), no profile sync/translate/autofill, ~200MB+ footprint.
4. DRM affects: all paid video streaming, Spotify/Apple Music web players, some broadcaster catch-up. YouTube free / Twitch / everything non-media unaffected.

## Prior art
Electron browsers succeed as *scoped* browsers (Min — still maintained; Ferdium-class webapp containers), stall as general browsers (Wexond, Beaker — both dead; useful leftover: `electron-chrome-extensions`). Serious products (Arc, Sidekick, Brave, Vivaldi) build on raw Chromium.

## Firefox fork path
- Firefox's entire GUI (tabs, toolbar, urlbar) is **XHTML/CSS/privileged JS** — a tab is literally `gBrowser.addTab(url, …)`; web content lives in `<browser remote="true">` elements (Gecko's WebContentsView equivalent). Engine underneath is C++/Rust; GUI rebuild never touches it until you need unexposed engine capability.
- **No Electron-of-Gecko exists**: XULRunner (dead 2015), Positron (dead 2017), qbrt (dead), GeckoView (Android-only), Servo (not web-compatible). Desktop Gecko ships only as Firefox; forking the whole browser is the only path.
- Internal APIs (`gBrowser`, JSWindowActors, `Services.*`) have **no stability contract** — the perpetual cost is rebasing against Mozilla's ~4-week release train. Zen Browser / Floorp are live templates: patch-set + new frontend over stock Firefox with rebase CI. Start from their tooling, not raw mozilla-central.
- Realistic spectrum: (1) restyle existing frontend (Zen-style, months, sustainable) → (3) gut visible UI but keep tabbrowser/session machinery as SDK (pragmatic) → (2) true from-scratch frontend (nobody has sustained this).
- Bonus: fork inherits Widevine, Mozilla patch stream, local translation (Bergamot), and the **Firefox AI Runtime** (on-device ONNX/wllama inference callable from privileged JS) — Chrome's AI stack (Gemini Nano, window.ai) is proprietary and absent from Chromium forks.

## Theia integration
- Firefox contains no Node — Theia's backend cannot live in Gecko. Shape: bundle Node + Theia **browser-target** backend as a child process on localhost; custom GUI's `<browser>` loads it. Theia integration ≈ days; the Firefox fork itself is the months-scale cost.
- **Unified tab strip** (browser tabs + Theia editor tabs as one movable set): feasible. Recommended = Option A "mirror/proxy": Theia in one persistent `<browser>`, a small Theia extension exposes shell state (open/activate/close widgets) over postMessage/WebSocket, chrome-level tab strip owns the unified model, Theia's own tab bar hidden via CSS. Option B (one Theia secondary-window widget per `<browser>`) works via @theia/core secondary-window support but is heavier/flakier. Hard part: drag-and-drop across the strip↔Lumino dock boundary. ~2–4 weeks on top of the fork baseline.

## Verdict
- Electron route: IDE + browser tabs in weeks; sacrifices DRM (or Castlabs) and accepts security ceiling.
- Firefox-fork route: only pays off if the **browser half is the product core**; then Zen-style fork + Theia-on-localhost + unified tab proxy is the architecture.
