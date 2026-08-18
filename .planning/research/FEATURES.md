# Feature Research

**Domain:** Branded Theia-fork product foundation (IDE/workbench platform rebrand, not a from-scratch app)
**Researched:** 2026-08-18
**Confidence:** HIGH (Theia's own docs + DeepWiki cross-reference + two live adopter products, Theia IDE itself and Arduino IDE 2, all agree)

## What Theia Gives You For Free

This is the core finding: almost everything Sourcerer's v1.0 Tauri shell built by hand (docking, panels, command routing, theming, settings persistence, extension registration) is a **platform capability already shipped in `@theia/core`** — the milestone's job is to fork, configure, and skin it, not rebuild it.

| Capability | What it is | Maturity |
|---|---|---|
| Docking/panel shell | Lumino-based application shell — tabbed panels, drag-to-dock, split views, resize, maximize/restore. Same problem space as v1.0's dockview-core center dock, but framework-native. | Core, stable |
| Extension system | InversifyJS dependency injection; extensions bind implementations to contribution-point interfaces (`WidgetFactory`, `CommandContribution`, `MenuContribution`, `KeybindingContribution`, `PreferenceContribution`). Two extension kinds: native Theia extensions (TS, DI-integrated) and VS Code extensions (via a plugin-host compatibility layer). | Core, stable |
| Command palette | Ctrl/Cmd+Shift+P palette; commands registered once via `CommandContribution`, then bindable to palette, menu, or keybinding. | Core, stable |
| Settings/preferences | `@theia/preferences` — Settings UI works like VS Code's User/Workspace settings; extensions register their own preference schemas. | Core, stable |
| Keybindings | `@theia/keymaps` — user-editable `keymaps.json`, conditional keybinding contexts, VS Code-familiar defaults. | Core, stable |
| Theming | CSS custom-property based theme system, VS Code theme-JSON compatible; swapping a theme is a first-class extension point. | Core, stable |
| Theia AI framework | Full agentic chat framework: `@theia/ai-chat`, `@theia/ai-core`, provider abstraction (OpenAI, Anthropic, Azure, Google, local via Ollama/LlamaFile), chat-agent registration via `@` mentions, session persistence to disk. This is a superset of what v1.0 built bespoke for the Dashboard Assistant (Pi sidecar behind `host.ai()`). | Core, actively developed (major 2025 feature push) |

**Implication for Sourcerer:** the applet framework's job shrinks from "build a dock, a rail, a persistence layer, and a host API" to "map `host.storage`/`host.ai()`/`host.open()` onto Theia's existing preference/DI/AI-framework equivalents, and register applets as Theia extensions/widgets instead of registry-loaded React components." That mapping decision is explicitly **out of scope for this milestone** (Sourcerer's own carry-over review is deferred per PROJECT.md) but every finding below assumes it's coming.

## Feature Landscape

### Table Stakes (a "branded Theia product" is not shippable without these)

| Feature | Why Expected | Complexity | Notes |
|---|---|---|---|
| Application name (`applicationName` in `theia/frontend/config`) | Drives window title, About dialog, generated app metadata | LOW | One config key in `application-package.json`; window title auto-formats as `<workspace> — <appName>` or just `<appName>` with no workspace open |
| Window/taskbar icons (all platforms) | OS chrome, taskbar, dock, installer icon — an unbranded default Theia icon reads as "unfinished fork" immediately | LOW–MEDIUM | Icons live under `applications/electron/resources/`; each OS has different required sizes/formats (`.ico` Windows, `.icns` macOS, `.png` set Linux) — mechanical but must not be skipped |
| Product Extension package (branding layer) | Theia IDE's own pattern: a dedicated extension (`theia-blueprint-product`-equivalent) owns Getting Started page, About dialog, app-specific menu items | MEDIUM | This is the "how" for everything else in this table — create `sourcerer-product` extension early, don't scatter branding across core patches |
| Default target + platform choice (`electron` vs `browser`) | `theia/target` in `application-package.json`; Sourcerer is a desktop app → `electron` | LOW | Matches v1.0's desktop-only posture; browser target not needed |
| Reproducible local build (`yarn`/`npm` build + `theia build`/`theia start`) | The milestone's stated goal is literally "building locally; dev workflow documented and repeatable" | MEDIUM | This is the actual deliverable of the milestone, not a side effect — budget real time for Node/Electron native-module rebuild quirks (a known Theia pain point independent of Tauri's `cargo tauri dev` landmine) |
| Getting Started / Welcome page rebrand | Default Theia welcome page has Eclipse/Theia branding, links, and news feed — leaving it as-is is the single most visible "we didn't rebrand this" tell | MEDIUM | Documented pattern: subclass `GettingStartedWidget`, rebind via DI container module — same mechanism Theia IDE itself uses for its own Blueprint branding |
| About dialog rebrand | Version info, product name, license — same Product Extension as Getting Started | LOW | Bundled with the product extension work above |
| Default layout on first launch | Which panels are visible/positioned when a user opens Sourcerer cold, before any workspace-specific state exists | LOW–MEDIUM | Theia supports "have a widget open by default" via layout contribution; Sourcerer's dock-tree/rail concepts from v1.0 don't exist yet in Theia terms — first-launch layout should default to something coherent (e.g., explorer + editor + empty panel), not literally "None," but does not need to resemble the old bespoke shell yet |
| Stripped irrelevant IDE features | Full Theia IDE ships language servers, debug adapters, Git, terminal, etc. — a scholar's workbench does not need a debugger or task runner by default | MEDIUM–HIGH | This is a *removal* exercise via `application-package.json` dependency list (only include the Theia extension packages you want compiled in) — get this list right early, since it drives the whole build's footprint and startup time |

### Differentiators (branding-specific value beyond "it compiles")

| Feature | Value Proposition | Complexity | Notes |
|---|---|---|---|
| Theia AI framework wired to Sourcerer's model/provider choices from day one | Theia AI's provider abstraction (OpenAI/Anthropic/Azure/Google/Ollama/local) is a strict superset of the single hardcoded Pi-sidecar path v1.0 built — adopting it now, even minimally, avoids re-solving "how does the assistant call an LLM" a second time in a later milestone | MEDIUM (to wire *a* provider) / HIGH (to reach v1.0 Dashboard Assistant parity) | Explicitly deferred by PROJECT.md's "carry-over review" note — flag as a strong candidate for the *next* milestone, not this one |
| Custom toolbar / product-specific chrome | Arduino IDE 2 proves this is a well-trodden path — Theia is "very flexible... even when it comes to common workbench elements" | MEDIUM | Not needed for a bare foundation milestone; relevant once Sourcerer's applet-specific chrome (rail, assistant panel) gets ported into Theia terms |
| VS Code extension compatibility layer | Theia can run real VS Code extensions via a plugin host — a long-term differentiator (Sourcerer could reuse existing scholarly-tooling VS Code extensions e.g. citation managers, Markdown/LaTeX tooling) without writing them | LOW to enable, HIGH to depend on | Free to leave enabled; do not architect around it yet — table it as a later-milestone option, note it exists |
| Green accent theme (`#86A38C`) applied via Theia's theme system | Preserves the one piece of v1.0 visual identity worth carrying forward even though the shell itself is superseded | LOW–MEDIUM | Theme system is CSS-custom-property + VS Code theme-JSON compatible, so v1.0's `tokens.css` accent value ports cleanly; full pixel-perfect fidelity to the old bespoke_rails_shell handoff does **not** carry over as a hard constraint into Theia (Theia's own chrome geometry differs from the old 40px-bar/0-radius system) — treat as a color/identity port, not a pixel-fidelity port |

### Anti-Features (skip for this milestone)

| Feature | Why Requested | Why Problematic Now | Alternative |
|---|---|---|---|
| Full pixel-perfect port of the old bespoke_rails_shell tokens (40px title bar, 0-radius, exact rail/dock geometry) onto Theia's shell | Continuity with v1.0's design language | Theia's shell has its own Lumino-based layout geometry and interaction model; forcing v1.0's exact pixel spec onto it fights the framework the same way a component library would, defeating the reason for adopting Theia in the first place | Port the *identity* (green accent, IBM Plex fonts, product name) via Theia's theme system; treat exact chrome geometry as a later, deliberate design pass once the team decides how much of Theia's native chrome to keep vs override |
| Porting the Dashboard Assistant (Pi sidecar) into Theia AI this milestone | Momentum — "we already built an assistant, just plug it in" | PROJECT.md explicitly defers all carry-over review (Pi harness, Databasise plans, tokens, Nix/CI/cache) to a later phase; Theia AI's architecture (chat agents, `@`-mention routing, provider abstraction) is different enough from the bespoke `host.ai()` seam that a rushed port now risks re-doing it wrong twice | Get the bare fork branded and building first; scope the assistant-harness port as its own milestone with its own research pass |
| Rebuilding the applet framework / `host` API inside Theia this milestone | Desire to restore v1.0 feature parity quickly | Milestone scope is explicitly "a working foundation," not feature parity; mapping `host.storage`/`host.ai()`/`host.open()` onto Theia's DI/preference/AI-framework equivalents is a real architecture decision that needs its own research (which extension pattern, which contribution points) | Defer to the milestone immediately after this one; this milestone's success criterion is "Theia fork builds, runs, is branded" |
| Aggressive feature-stripping beyond what blocks branding/build (e.g. ripping out Git, terminal, debug support day one) | Minimize footprint, look "clean" | Theia's dependency-driven build means removing a package is cheap to redo later; over-stripping now risks breaking a build you haven't yet learned to rebuild reliably, and some "IDE" features (terminal, file explorer) are plausibly useful for a scholar's workbench regardless (e.g. running local scripts, browsing a research corpus) | Ship with Theia's default reasonably-sized package set for v1 of the fork; strip deliberately, package by package, once the build is proven reproducible — not as a first move |

## Feature Dependencies

```
Fork Theia (source or theia-blueprint template)
    └──requires──> application-package.json configured (target=electron, applicationName, extension list)
                       └──requires──> Product Extension package created (icons, About, Getting Started)
                                          └──enhances──> Default layout / first-launch experience

Reproducible local build (yarn/theia build)
    └──blocks──> everything else (branding is invisible if the fork doesn't compile/run)

Theme system (green accent, IBM Plex)  ──independent-of──> Product Extension (can land in parallel)

Theia AI framework adoption  ──deferred-until──> next milestone (assistant/applet carry-over review)
Applet-framework/host-API port  ──deferred-until──> next milestone (same review)
```

### Dependency Notes

- **Everything requires a reproducible build first.** The milestone's own stated goal ("Theia fork forked and building locally; dev workflow documented and repeatable") is the true root dependency — branding work landed on a build that isn't yet proven to rebuild cleanly is wasted if the build breaks and can't be diagnosed.
- **Product Extension is the correct single point** for name/icon/About/Getting-Started — Theia IDE's own architecture (and Arduino IDE 2's fork) both centralize branding this way rather than scattering patches through core. Create it once, early.
- **Theme identity porting is independent of everything else** and low-risk — safe to do in parallel with build/branding work, unlike the AI-framework and applet-framework decisions which are explicitly out of scope.
- **Theia AI and the applet/host-API port are the same class of deferred decision**: both require picking Theia's specific extension mechanism (contribution points, DI bindings) to receive v1.0 concepts, and PROJECT.md is explicit that this review happens in a later phase, not this milestone.

## MVP Definition

### Launch With (this milestone — "working, rebranded fork")

- [ ] Theia forked (from `theia-blueprint`/`theia-ide` as the composition template, or Theia core directly — choose per Sourcerer's own repo-reorg needs) and building locally on the dev machine — the literal goal statement
- [ ] `application-package.json` set: `target: electron`, `applicationName: "Sourcerer"` (or decided product name), curated extension dependency list
- [ ] Product Extension package created: app icons (all platforms), About dialog text, window title behavior verified
- [ ] Getting Started / Welcome widget rebranded (no stock Theia/Eclipse branding visible on first launch)
- [ ] Documented, repeatable dev workflow (equivalent of v1.0's `tauri dev` landmine doc — capture whatever Theia's own build/run quirks turn out to be on this machine)
- [ ] Default first-launch layout is coherent (not empty, not confusing) — doesn't need to resemble v1.0's home dashboard yet

### Add After Validation (next milestone — carry-over review)

- [ ] Theme identity port: green `#86A38C` accent + IBM Plex fonts applied via Theia's theme system
- [ ] Decide `host.storage`/`host.ai()`/`host.open()` → Theia DI/preference/AI-framework mapping
- [ ] Decide applet-framework → Theia extension/widget registration mapping
- [ ] Port or replace the Pi-sidecar Dashboard Assistant using Theia AI's chat-agent framework

### Future Consideration (later)

- [ ] VS Code extension compatibility as a Sourcerer feature (reuse existing scholarly-tooling extensions)
- [ ] Custom toolbar / rail-equivalent chrome once applet framework decisions are made
- [ ] Deliberate, feature-by-feature stripping of unneeded default Theia IDE capabilities (debugger, task runner, etc.) once build is proven stable

## Feature Prioritization Matrix

| Feature | User Value | Implementation Cost | Priority |
|---|---|---|---|
| Reproducible local build + documented workflow | HIGH | MEDIUM | P1 |
| Product name / window title / icons | HIGH | LOW | P1 |
| Product Extension (About, Getting Started rebrand) | HIGH | MEDIUM | P1 |
| Sensible default first-launch layout | MEDIUM | LOW–MEDIUM | P1 |
| Theme identity port (green accent, IBM Plex) | MEDIUM | LOW–MEDIUM | P2 |
| Theia AI framework wiring | HIGH (long-term) | MEDIUM–HIGH | P3 (next milestone) |
| Applet/host-API mapping | HIGH (long-term) | HIGH | P3 (next milestone) |
| Feature stripping beyond build/branding needs | LOW (cosmetic) | MEDIUM | P3 |
| VS Code extension compatibility as product feature | MEDIUM | LOW to enable / HIGH to depend on | P3 |

**Priority key:**
- P1: Must have for this milestone (foundation)
- P2: Should have, low-risk to parallelize into this milestone
- P3: Explicitly deferred — next milestone or later

## Competitor/Adopter Feature Analysis

| Aspect | Theia IDE (theia-blueprint, the reference upstream product) | Arduino IDE 2 | Sourcerer's Approach |
|---|---|---|---|
| Branding layer | Dedicated Product Extension (`theia-blueprint-product`) owns icons/About/Getting Started/menus | Custom toolbar + Arduino-specific UI on top of Theia shell | Same pattern: one `sourcerer-product` extension as the single branding surface |
| Backend integration | N/A (Theia IDE is generic) | `arduino-cli` runs as an independent background daemon; Theia frontend talks to it — architecturally separate from the IDE shell | Directly analogous to how Databasise (Cozo+LightRAG engine) is meant to integrate — a separate backend service the Theia frontend talks to, not baked into the fork. Validates the "one server, two clients" framing already in PROJECT.md |
| AI integration | Full Theia AI framework, first-party, actively developed (chat agents, session persistence, multi-provider) | N/A (not AI-focused) | Confirms Theia AI is the natural eventual home for the Dashboard Assistant, but this milestone doesn't touch it |
| Extension model exposure | Showcases both native Theia extensions and VS Code extension compatibility | Uses native Theia extensions for Arduino-specific tooling | Sourcerer's future applets are the direct analog of Arduino's board/sketch tooling — native Theia extensions, not VS Code compat layer, is the expected fit (deferred decision, but this is the precedent) |

## Sources

- [Product Extension | eclipse-theia/theia-ide | DeepWiki](https://deepwiki.com/eclipse-theia/theia-ide/6.1-product-extension) — HIGH confidence, describes the branding-layer pattern this milestone should follow
- [Extending/Adopting the Theia IDE](https://theia-ide.org/docs/blueprint_documentation/) — HIGH confidence, official docs, applicationName/window-title/icon mechanics
- [Build your own IDE/Tool](https://theia-ide.org/docs/composing_applications/) — HIGH confidence, official docs, `application-package.json` target/extension composition
- [theia/dev-packages/cli/README.md](https://github.com/eclipse-theia/theia/blob/master/dev-packages/cli/README.md) — HIGH confidence, official CLI/build docs
- [Introducing Theia AI](https://eclipsesource.com/blogs/2025/03/13/introducing-theia-ai/) and [AI Integration | theia-ide | DeepWiki](https://deepwiki.com/eclipse-theia/theia-ide/2.3-ai-integration) — HIGH confidence, Theia AI framework capabilities and provider support
- [Theia Adopter Story: The new Arduino IDE 2.0 | Eclipse Foundation Blog](https://blogs.eclipse.org/post/john-kellerman/theia-adopter-story-new-arduino-ide-20) — HIGH confidence, primary adopter account of forking/customizing Theia
- [Extension System | eclipse-theia/theia-ide | DeepWiki](https://deepwiki.com/eclipse-theia/theia-ide/2.2-extension-system) and [Services and Contributions](https://theia-ide.org/docs/services_and_contributions/) — HIGH confidence, InversifyJS DI + contribution-point architecture
- [Customize Theia start widget - Theia Community](https://community.theia-ide.org/t/customize-theia-start-widget/2479) and [Theia Deep Dive, Part 2: Mastering Customization - DZone](https://dzone.com/articles/theia-deep-dive-mastering-customization) — MEDIUM confidence (community forum + third-party writeup, cross-checked against official docs), Getting Started widget rebranding pattern
- [Eclipse Theia Blueprint / theia-ide GitHub](https://github.com/eclipse-theia/theia-ide) — HIGH confidence, confirms Theia IDE == rebranded Theia Blueprint, the reference fork-and-brand template
- [Preferences](https://theia-ide.org/docs/preferences/) and [Commands/Menus/Keybindings](https://theia-ide.org/docs/commands_keybindings/) — HIGH confidence, official docs, settings/command-palette/keybinding built-ins
- Project's own `.planning/PROJECT.md` — HIGH confidence, primary source for milestone scope and explicit deferrals (carry-over review, Theia AI/applet mapping)

---
*Feature research for: Branded Theia-fork product foundation (Sourcerer v3.0 Theia Foundation milestone)*
*Researched: 2026-08-18*
