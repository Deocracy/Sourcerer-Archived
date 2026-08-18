# Stack Research

**Domain:** Tauri 2 desktop shell — dockable multi-pane workspace + plugin ("applet") framework, React 18 + Vite + TypeScript frontend, Rust backend
**Researched:** 2026-07-06
**Confidence:** HIGH (versions verified against npm registry; Tauri patterns verified against v2.tauri.app official docs); MEDIUM on a few library-choice calls flagged below

## Recommended Stack

### Core Technologies

| Technology | Version | Purpose | Why Recommended |
|------------|---------|---------|-----------------|
| Tauri | 2.11.x (CLI `@tauri-apps/cli` 2.11.4, JS API `@tauri-apps/api` 2.11.1) | Rust-backed desktop shell, single frameless window | Handoff-specified; 2.x is the current stable major (2.0 shipped Oct 2024, actively patched through 2.11 as of mid-2026). Gives you a native window API (`decorations:false`, `startDragging`), a Rust command layer for the `ai_complete` seam, and a plugin ecosystem (store/sql) that maps directly onto `host.storage`. |
| React | 18.2.0 (pin to 18.x, NOT 19) | UI runtime | **Deliberate pin, do not upgrade to React 19.** Handoff explicitly specifies React 18; the ported prototype code, `React.createElement`-style applet contract, and hook usage were authored against 18 semantics. React 19 changes ref-as-prop, `useEffect` cleanup timing nuances, and removes some legacy APIs — upgrading mid-build risks subtle drag/hit-testing regressions in ported algorithms. Revisit the pin as its own decision once the shell is stable, not during initial build. |
| Vite | 5.4.x (see note below — NOT the current Vite 8 line) | Dev server + bundler | **Do not default to "latest."** npm's `latest` tag currently resolves to Vite 8.1.3, which requires Node 20.19+/22.12+ and switched to Rolldown/Oxc + Lightning CSS — a materially different toolchain than what Tauri's official `create-tauri-app` React-TS template and most current Tauri examples are pinned against (Vite 5 or 6). Since Tauri's dev workflow depends on Vite's dev-server-behind-a-Rust-webview integration (`beforeDevCommand`, HMR over the Tauri devUrl), stick with the version `create-tauri-app` scaffolds for you rather than hand-picking Vite 8. Confirm the exact pin by running `npm create tauri-app@latest` fresh and reading its generated `package.json` — do not hardcode Vite 8 into the roadmap. |
| TypeScript | 5.6.x+ (latest stable, currently 6.0.3 on npm — verify against Vite's peer range at scaffold time) | Type safety across shell + applet contracts | Handoff-specified. Use whatever `create-tauri-app`'s React-TS template pins; do not chase bleeding-edge TS majors independently of the Vite/React template, since `@vitejs/plugin-react` and `vite` have their own peer ranges. |
| Rust | stable toolchain (via `rustup`), edition 2021 | Tauri backend | Tauri 2 requires a recent stable Rust; use whatever `rustup update stable` gives you at scaffold time. No reason to pin an old toolchain. |

**IMPORTANT version-pinning caveat:** Do not npm-install `vite@latest` / `typescript@latest` / `react@latest` blindly — at research time `latest` resolves to Vite 8.1.3, React 19.2.7, and unqualified installs will silently upgrade past the handoff's React 18 requirement and past Tauri's currently-verified Vite line. **Scaffold with `npm create tauri-app@latest` and let it choose the compatible Vite/TS versions; separately downgrade React (and `@types/react`, `@types/react-dom`) to the 18.2.x line.** This is the single highest-risk version trap in this stack — flag it for the roadmap's first phase (project scaffolding).

### Supporting Libraries

| Library | Version | Purpose | When to Use |
|---------|---------|---------|-------------|
| `@tauri-apps/plugin-store` (JS) + `tauri-plugin-store` (Rust crate) | 2.4.3 (JS) | JSON key-value persistence backing `host.storage` and workspace/layout state | From day one — this is the handoff's chosen persistence layer. Maps near-1:1 onto the prototype's `localStorage`-backed `host.storage.get/set/remove`. Use one store file (e.g. `store.bin`/`.json`) per logical concern (e.g. `shell-state.json`, `applets.json`) or a single store keyed by `sourcerer:<key>:<k>` exactly as the applet contract specifies. |
| `@tauri-apps/plugin-sql` (JS) + `tauri-plugin-sql` (Rust crate) | 2.4.0 | SQLite (and optionally Postgres/MySQL) access | **Not in v1.** Handoff explicitly defers this to when an applet needs real relational data (e.g., once Notes or a future applet outgrows flat JSON). Don't scaffold it speculatively — adding a plugin later is cheap; premature schema design is not. |
| Zustand | 5.0.14 | Single shell store (`dockTree`, `activePaneId`, `railOrder`, `railWidth`, `asstWidth`, `assistantOpen`, `savedLayouts`, per-tab applet state) | From day one. See rationale below. |
| `zustand/vanilla` + `zustand/middleware` (`persist`, `subscribeWithSelector`) | bundled with zustand 5.x | Persisting the shell store to `tauri-plugin-store` and subscribing to slices without re-rendering the whole tree | Use `subscribeWithSelector` to drive fine-grained persistence writes (e.g. only write layout to disk when `dockTree`/`railOrder`/widths change, debounced) rather than persisting on every render. |
| `@fontsource/ibm-plex-sans`, `@fontsource/ibm-plex-mono`, `@fontsource/ibm-plex-serif` | 5.2.8 (sans; mono/serif track the same major) | Locally bundled IBM Plex fonts, no runtime Google Fonts fetch | Import the specific weights actually used (400/500/600 sans; 400/500 mono; 400 + italic serif) via Fontsource's per-weight subpath imports (e.g. `@fontsource/ibm-plex-sans/400.css`) to avoid bundling unused weight files. Confirms to "bundle locally" constraint and keeps bundle size proportional to actual usage. |
| CSS Modules (`*.module.css`) + a single `tokens.css` (CSS custom properties) | native to Vite, no extra package | Bespoke pixel-perfect styling keyed to design tokens | See CSS strategy section below — this is the recommended approach over CSS-in-JS. |
| `nanoid` or Tauri's own UUID approach | nanoid ^5.x | Generating tab/instance IDs (`host.instanceId`), pane IDs, layout IDs | Small, dependency-free, URL-safe unique IDs — matches the prototype's need for stable per-tab instance identifiers without pulling in a heavier UUID library. |

### Development Tools

| Tool | Purpose | Notes |
|------|---------|-------|
| `create-tauri-app` (via `npm create tauri-app@latest`) | Scaffolds the Tauri 2 + React + TS + Vite project | Always scaffold fresh rather than hand-assembling `Cargo.toml`/`tauri.conf.json``/vite.config.ts` from memory — the official scaffolder tracks current compatible version ranges and correct `tauri.conf.json` schema (v2 config format differs meaningfully from v1). |
| `@vitejs/plugin-react` | Vite's React fast-refresh plugin | Whatever version `create-tauri-app` pins; peer-locked to the Vite major it ships with (6.x of the plugin requires Vite ^8, so if the scaffolder gives you Vite 5/6/7 it will also give you a matching older `@vitejs/plugin-react`. Don't mix majors by hand.) |
| ESLint + `@typescript-eslint` | Linting | Standard for a TS + React project; not handoff-specified but table-stakes for a multi-phase build with a plugin contract other "applets" must conform to. |
| `cargo clippy` / `cargo fmt` | Rust linting/formatting | Standard Rust hygiene for the `src-tauri` backend, especially since the `ai_complete` command surface will grow. |
| Tauri CLI dev workflow (`tauri dev`, NOT `cargo tauri dev` per project memory) | Local dev loop | **Project-specific landmine carried over from the related Databasise/Sourcerer engine project:** prior experience in this workspace found `cargo run` from `src-tauri` more reliable than `cargo tauri dev` for certain sidecar/editable-install setups. Verify which invocation is reliable for *this* fresh scaffold early — don't assume the old landmine applies unmodified, but don't assume it's resolved either. |

## Installation

```bash
# Scaffold (choose React + TypeScript template when prompted)
npm create tauri-app@latest

# Inside the scaffolded project — pin React to 18.x (do NOT accept a React 19 default)
npm install react@18.2.0 react-dom@18.2.0
npm install -D @types/react@^18 @types/react-dom@^18

# Persistence
npm install @tauri-apps/plugin-store
cargo add tauri-plugin-store --manifest-path src-tauri/Cargo.toml

# State management
npm install zustand

# Fonts (import only the weights you use, per-file)
npm install @fontsource/ibm-plex-sans @fontsource/ibm-plex-mono @fontsource/ibm-plex-serif

# IDs
npm install nanoid

# Dev tooling
npm install -D eslint @typescript-eslint/parser @typescript-eslint/eslint-plugin
```

Rust side (`src-tauri/Cargo.toml`), add alongside the scaffolded `tauri` dependency:

```toml
[dependencies]
tauri = { version = "2", features = [] }
tauri-plugin-store = "2"
serde = { version = "1", features = ["derive"] }
serde_json = "1"
```

## Alternatives Considered

| Recommended | Alternative | When to Use Alternative |
|-------------|-------------|-------------------------|
| Zustand (single shell store) | Redux Toolkit | If the shell state graph grows to need time-travel debugging, strict action-log auditing, or a large team needs enforced action/reducer conventions. For a single-shell-store desktop app with one primary maintainer porting an existing prototype, Zustand's near-zero boilerplate and `zustand/vanilla` (usable outside React, e.g. for persistence-layer logic) is the better fit. |
| Zustand | Jotai (atomic state) | If applet-private state needs fine-grained atom-level subscriptions across many independent small pieces of state. The shell's state shape here is explicitly one cohesive object (`dockTree`, rail state, etc.) per the handoff's "State Management" section — a single store, not atoms, matches that shape directly. |
| Native pointer events for drag/dock/resize (ported from prototype) | `dnd-kit` / `react-dnd` / `react-beautiful-dnd` | Only if a *future* applet (not the shell itself) needs generic sortable-list drag-and-drop and doesn't want to hand-roll it. **For the shell itself: do not introduce a DnD library.** The prototype already has working, pixel-tuned hit-testing (`hitTest`, `performDock`, `prune`, `startDockDrag`) with a 5px movement threshold, 5-zone dock chips, and edge-split detection — this is bespoke geometry logic no generic DnD library replicates out of the box, and forcing it through a library's abstraction (virtual drag overlays, sensor config, collision detection APIs) would fight the existing design rather than help it. Port the prototype's plain pointer-event handlers (`pointerdown`/`pointermove`/`pointerup`, `setPointerCapture`) directly into React with `useRef`+native listeners or a small custom hook. |
| CSS Modules + CSS custom-property tokens | Tailwind CSS | Tailwind's utility classes fight a bespoke, pixel-exact token system (specific hex values, 1px borders, 0 border-radius everywhere, precise 34px/36px/40px metrics) — you'd spend more time configuring Tailwind's theme to match tokens exactly than just writing token-driven CSS directly. Reasonable to reconsider only if the team standardizes on Tailwind for *other* projects and wants consistency at the cost of extra config. |
| CSS Modules + tokens | CSS-in-JS (styled-components, vanilla-extract, Emotion) | vanilla-extract is worth a second look if compile-time type-checked style tokens become valuable (it generates static CSS, avoids runtime cost) — but for a from-scratch bespoke shell, plain CSS Modules + a `tokens.css` custom-properties file is simpler, has zero runtime cost, zero extra dependency, and every token is just a CSS variable referenced everywhere (`var(--color-fg)`), which is exactly what the design tokens table needs. Runtime CSS-in-JS (styled-components/Emotion) adds bundle weight and a render-time cost with no benefit here since there's no dynamic per-instance theming beyond the one dark theme in the handoff. |
| `tauri-plugin-store` (JSON) for v1 | `tauri-plugin-sql` (SQLite) | Handoff already flags this transition point: move to SQL once an applet needs relational queries, multi-row filtering, or data volume where flat-JSON round-trip cost matters (e.g. a Library applet indexing hundreds of documents). Don't pre-adopt it in the shell/Notes-only phase. |
| `@tauri-apps/plugin-store` scoped per-key namespace | Raw `localStorage` (browser storage) inside the webview | The applet framework's `host.storage` contract is *currently* implemented via `localStorage` in the HTML prototype (per `reference/applets/README.md`: "Persistence... is localStorage under `sourcerer:<appletKey>:<key>`"). In the real Tauri app, `localStorage` would be scoped to the webview and lost on certain reset/profile scenarios, and doesn't survive OS-level app-data conventions users expect (backups, multi-window, future sync). Route `host.storage` through `tauri-plugin-store` from the start — it's explicitly what the handoff calls for and gives you a real file on disk under the OS app-data directory. |

## What NOT to Use

| Avoid | Why | Use Instead |
|-------|-----|-------------|
| Any component library (MUI, Ant Design, Chakra, shadcn/ui) | Handoff is explicit: "no component library — the design is bespoke," pixel-perfect fidelity to final tokens with 0 border-radius everywhere. Component libraries impose their own spacing/radius/shadow defaults that fight this. | Bespoke components styled directly against `tokens.css`. |
| React 19 (or any "latest" React tag installed without pinning) | Breaks the explicit React 18 requirement in both `.planning/PROJECT.md` and the design handoff; introduces ref-as-prop and other semantic changes that could subtly affect ported drag/hit-testing code paths that were designed/tested against React 18's `act()`/effect timing. | React 18.2.0, pinned explicitly in `package.json`, not left to float. |
| Vite 8 (Rolldown/Oxc-based) without first confirming Tauri's official template has moved to it | At research time Vite 8 (npm `latest`) is a substantial toolchain shift (Rolldown instead of Rollup, Lightning CSS instead of esbuild for CSS min) released March 2026 — newer than most current Tauri example projects and community guides, which mostly document Vite 5/6/7 integration with Tauri's dev-server proxying. Using it unverified risks subtle HMR/dev-server integration issues with Tauri that won't show up until deep in a phase. | Whatever Vite version `npm create tauri-app@latest` scaffolds by default at project-start time — treat that as the source of truth, not a manually chosen "latest." |
| A generic drag-and-drop library (`dnd-kit`, `react-dnd`, `interact.js`) for the dock/rail/tab system | The project brief is explicit that "the prototype ports its own hit-testing" — the existing algorithms (5-zone dock chips, edge-split, tab-bar reorder with caret preview, rail reorder/pin/drag-out-to-dock) are proven and specific to this exact interaction design. Wrapping them in a generic DnD library's sensor/collision abstractions would mean re-deriving custom collision logic *inside* the library's constraints — net negative. | Native Pointer Events (`pointerdown`/`pointermove`/`pointerup`/`setPointerCapture`) ported near-1:1 from `support.js`'s `hitTest`/`performDock`/`startDockDrag`. |
| Redux (classic, with reducers/actions/thunks boilerplate) | Excess ceremony for a single, cohesive shell store with a small, well-defined shape (`dockTree`, `railOrder`, widths, `savedLayouts`). Redux's normalized-store/action-log approach solves problems (large team coordination, audit trails, time-travel debug across a huge state graph) this project doesn't have. | Zustand — one `create()` call, direct mutation via `set()`, `zustand/persist` middleware for the store-plugin sync. |
| Building the applet sandbox as literal separate bundles loaded at runtime via `React` passed through props (the prototype's exact mechanism) | The prototype passes `React` via props specifically because it has no bundler — every applet module runs directly in the browser off `<script type=module>` semantics. In the real Tauri+Vite app you have a real bundler; the handoff itself says "drop the React-via-props indirection if using a bundler." Keeping it would forgo tree-shaking, TypeScript typing across the applet boundary, and standard React DevTools support for no benefit. | Applets as ordinary `.tsx` modules that `import React` normally and are registered into a typed `registry.ts` (`manifest` + `App` exported per module, imported at build time for v1; a true runtime-loaded plugin system, if ever needed, is a separate, much larger architectural decision — dynamic `import()` of untrusted/external code is out of scope here). |

## Stack Patterns by Variant

**If the applet registry stays build-time (all applets are known at compile time, just some are "stub" placeholders):**
- Use static ES module imports in `registry.ts` (`import * as Notes from './applets/Notes'`), exactly mirroring the prototype's `registry.js` pattern.
- Because this is what the handoff's "Suggested build order" implies (applets built "one at a time, each replacing its stub via the registry") — there's no requirement in `.planning/PROJECT.md` for loading arbitrary third-party/external applet code at runtime, so a heavier dynamic-plugin-loader architecture (sandboxed iframes, dynamic `import()` from an app-data directory, permission manifests) is unnecessary complexity for v1.

**If a future milestone needs truly runtime-loaded, user-installable applets (not in this milestone's scope):**
- Revisit with dynamic `import()` of ES modules from a Tauri-managed app-data directory, likely combined with a stricter typed `host` API version-negotiation and a permission/capability model — this is a materially bigger design problem (security boundary around arbitrary code) and should be its own dedicated research pass, not bundled into this phase's stack decision.

**If/when an applet needs relational data (e.g. Library indexing many documents, or once Databasise integration mode is decided):**
- Add `tauri-plugin-sql` for that applet's storage needs specifically, while the shell's own layout/rail/dock state can reasonably stay on `tauri-plugin-store` indefinitely (it's small, infrequently-shaped data — a SQL migration buys nothing there).
- Because the handoff explicitly scopes the SQLite graduation to "when applets need real data," not the shell state itself.

## Version Compatibility

| Package A | Compatible With | Notes |
|-----------|-----------------|-------|
| `@tauri-apps/api@2.11.x` | `tauri` Rust crate `2.x` (same major/minor track) | Keep the JS API package and the Rust `tauri` crate version in lockstep on the same 2.x line; Tauri's plugin JS packages (`@tauri-apps/plugin-store`, `@tauri-apps/plugin-sql`) similarly track a `2.x` Rust crate counterpart — always add both halves (JS + `cargo add`) together. |
| `react@18.2.0` | `@types/react@^18`, `@types/react-dom@^18` | Do not let `@types/react` float to `^19` while `react` stays pinned at 18 — this is a common and confusing type-mismatch trap; pin both together explicitly in `package.json`. |
| `vite` (whatever `create-tauri-app` scaffolds) | `@vitejs/plugin-react` (matching major) | `@vitejs/plugin-react@6.x` peer-requires `vite@^8`; if the scaffolder gives an older Vite (5/6/7), it will pair it with an older, compatible `@vitejs/plugin-react` automatically — don't manually bump one without the other. |
| `zustand@5.x` | React 18 | Zustand 5's React binding relies on `useSyncExternalStore`, available natively in React 18+ — fully compatible, no shim needed. |
| `@fontsource/ibm-plex-*` packages | any bundler (Vite) via CSS imports | Import per-weight CSS files (e.g. `@fontsource/ibm-plex-mono/500.css`) rather than the "all weights" index import, to keep bundled font-file weight proportional to actual design-token usage (400/500/600 sans, 400/500 mono, 400+italic serif per the handoff). |

## Sources

- [Tauri v2 official docs — Window Customization](https://v2.tauri.app/learn/window-customization/) — HIGH confidence: `decorations:false`, `data-tauri-drag-region`, `startDragging` permission model
- [Tauri v2 official docs — Calling Rust from the Frontend](https://v2.tauri.app/develop/calling-rust/) and [Calling the Frontend from Rust](https://v2.tauri.app/develop/calling-frontend/) — HIGH confidence: command/Channel/event patterns for the `ai_complete` seam
- [Tauri v2 Store plugin docs](https://v2.tauri.app/plugin/store/) and [SQL plugin docs](https://v2.tauri.app/plugin/sql/) — HIGH confidence: persistence plugin contracts
- npm registry API (`registry.npmjs.org/<pkg>/latest`) direct queries — HIGH confidence for exact version numbers as of 2026-07-06: `@tauri-apps/api` 2.11.1, `@tauri-apps/cli` 2.11.4, `@tauri-apps/plugin-store` 2.4.3, `@tauri-apps/plugin-sql` 2.4.0, `zustand` 5.0.14, `vite` 8.1.3 (flagged as NOT recommended without scaffolder verification), `react` 18.2.0 (latest 18.x) / 19.2.7 (current `latest` tag, explicitly avoided), `typescript` 6.0.3, `@fontsource/ibm-plex-sans` 5.2.8, `@vitejs/plugin-react` 6.0.3
- [Vite 8.0 announcement](https://vite.dev/blog/announcing-vite8) — MEDIUM confidence (WebSearch-summarized, not independently re-verified against the raw post): Rolldown/Oxc + Lightning CSS shift, Node 20.19+/22.12+ requirement — basis for recommending against blind Vite 8 adoption
- WebSearch on Zustand 2026 adoption trends (multiple sources: pmndrs/zustand GitHub, npm listing, community write-ups) — MEDIUM confidence: crossed 50% usage share among React state libraries per community-reported surveys; used to corroborate (not sole basis for) the Zustand recommendation, which is primarily justified by fit-to-shape (single cohesive store) rather than popularity
- Project's own design handoff docs (`design-sync-setup-guide/design_handoff_sourcerer_tauri/README.md`, `reference/applets/README.md`) — HIGH confidence, primary source: explicit stack mandate, `host` API contract, persistence/AI seam requirements
- User's private cross-session memory (`Windows detached-launch gotchas`, `cargo run` vs `cargo tauri dev` landmine from the related Databasise/Sourcerer engine project) — MEDIUM confidence, carried over from a related but distinct project; flagged as needing re-verification in this fresh scaffold, not assumed to transfer unmodified

---
*Stack research for: Tauri 2 + React 18 + Vite + TypeScript desktop shell with dockable workspace and applet plugin framework*
*Researched: 2026-07-06*
