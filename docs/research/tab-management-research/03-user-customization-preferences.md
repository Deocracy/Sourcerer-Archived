# User Customization Preferences in Navigation/Tab Management

Research date: 2026-08-19. Sourced via web search across extension stores, Mozilla Connect, vendor blogs, and press coverage (citations inline).

## 1. Extension popularity and what it signals

| Tool | Install/rating signal | What it tells us |
|---|---|---|
| **OneTab** (Chrome) | 2M+ users, 4.4★ | Largest tab-manager audience by far. Core appeal: one-click "convert all tabs to a list" for memory relief (claims ~95% RAM savings), not organization. Complaint pattern: occasional list loss on crash/update — users treat it as a safety valve, not a system of record. |
| **Tree Style Tab** (Firefox) | Long-running top-rated add-on | Power users report 600–1000+ tabs open and navigable via parent/child tree nesting. Reviewers say it's *the reason* they stay on Firefox — no other browser natively supports opening links as nested child tabs. Complaint pattern: auto-enabled new features silently disrupt hand-built tree layouts — this user segment wants stability over new defaults. |
| **Sidebery** (Firefox) | Positioned as the modern TST successor, well-reviewed on AMO/HN/Lobsters | Adds folders (manual grouping), per-domain/URL auto-coloring, sort-by-title/url/access-time, and auto-unload of collapsed branches. Long-time TST users migrate and don't migrate back — signals demand for TST's tree model *plus* bookmark integration and inactivity-based unloading. |
| **Session Buddy** (Chrome) | 25,101 reviews, 4.66★ aggregate (CWS shows 4.0) | Valued as a "secondary bookmarking system" for save/restore of whole sessions with local export/import/backup. Complaints: CPU spikes, silent data loss, no cloud sync, and one report of unannounced phone-home telemetry — trust and local-only storage are recurring friction points. |
| **Toby** (Chrome) | 300,000+ individuals across 144 countries; teams at Google, Netflix, Amazon, HubSpot, IBM; 4.6★ | Collection-based (drag-and-drop "workspaces" of saved tabs), praised for feeling like a native browser feature. Recent complaints cluster on monetization: a 60-item free-tier cap, forced login, confusing per-workspace billing — signal that free power-user tools face backlash the moment they gate the org structure itself. |
| **Workona** | Positioned as team/cloud tier above Toby | Strength: workspaces (task-based tab sets) with Slack/Drive integration for co-founders and multitaskers. Weakness repeatedly cited: unreliable cloud sync, stability issues at large collection sizes — sync durability is the make-or-break dimension once teams rely on it. |
| **Tabs Outliner** (Chrome) | 4.5★, longstanding cult following | Tree-structured, annotatable, Drive-backed session tree. Complaints: unmaintained, unstable across Chrome updates, dated/cluttered UI, paywalled key features — shows a segment that wants maximum structural power and will tolerate a rough UI for it, but not tool abandonment. |
| **The Great Suspender + its successors** | Removed from Chrome Web Store (Feb 2021) for injected malware/tracking after an ownership change; subsequent "successor" forks have repeated the same pattern | Direct evidence that memory-saving/suspension tools carry unusually high trust requirements: the extension must run code against every active tab to unload/reload it, so users who adopt one are making a standing trust bet, and the market has burned that trust more than once. This is why suspension/hibernation increasingly gets pulled into the browser itself (Vivaldi's native Tab Hibernation, Chrome's own memory saver) rather than left to third parties. |
| **Tab Manager Plus** (Chrome) | 4.4★ | Praised for duplicate-tab detection, block/list view toggle, cross-window reorganization. Complaint: no tab-group awareness, no undo, ~2 years without updates — table-stakes utility features (undo, dedupe) are treated as non-negotiable once a user has 20+ tabs. |

## 2. Customization dimensions (options, adopters, evidence)

### 2.1 Tab bar position: top (horizontal) vs. side (vertical)
- **Options:** classic horizontal strip; sidebar/vertical list; auto-hide/expand-on-hover vertical (Zen, Edge, custom userChrome.css).
- **Who picks what:** users with many simultaneous tabs and/or wide/ultrawide monitors gravitate to vertical — full page titles readable, less horizontal squeeze as tab count grows. Users on small/narrow screens or who keep few tabs stay horizontal, since vertical eats width that's scarcer than height there.
- **Evidence of demand:** Vivaldi shipped vertical tabs in 2015; Edge in 2021; Arc launched vertical-only in 2023; Firefox shipped it in 136 (March 2025) after being requested on Mozilla Connect for "more than two years"; Chrome added it in 2026. That every major browser eventually added it, in that order, is itself the strongest demand signal — see Section 3.
- **Nuance:** Edge's vertical tabs are explicitly called "divisive" by reviewers — praised for screen real estate on ultrawide monitors, criticized for eating horizontal space when windows are snapped side-by-side. Not a universal win, a tradeoff users want the *choice* to make.

### 2.2 Grouping model: manual folders vs. automatic/AI grouping vs. flat + search
- **Manual grouping (collections/workspaces):** Toby, Workona, Vivaldi Tab Stacks, Chrome/Firefox native tab groups. Preferred by users organizing around a persistent task or project ("this workspace = the redesign project") who want the grouping to be stable and intentional.
- **Automatic/AI grouping:** Chrome's "Organize similar tabs" (rolled out as an AI feature, on-device inference as of Chrome 146/March 2026) and third-party auto-group extensions. Reception is described as positive — "something to get excited about" — precisely because it removes the up-front organizing labor, but it's explicitly *additive* to manual groups, not a replacement; users still expect to rename/recolor/adjust after the AI proposes a grouping.
- **Flat list + search:** OneTab's whole model, plus Chrome/Edge's Ctrl+Shift+A tab-search panel and Chrome's `@tabs` omnibox keyword. The stated rationale in coverage: "past 20 open tabs, shortcuts stop being useful and a tab search panel becomes faster" — i.e., search-based retrieval is what high-tab-count users fall back to once spatial/visual scanning stops scaling.
- **Tree/hierarchical grouping:** Tree Style Tab and Sidebery's parent-child nesting, where a tab's position in the tree *is* the grouping (child tabs = "opened from this page"). This is a distinct dimension from folder-style grouping — it's implicit and automatic-by-navigation rather than a manual/explicit act.

### 2.3 Persistence: session restore, save/export, cloud sync
- **Local session save/restore:** Session Buddy, Tabs Outliner — valued for reliability and offline control, but explicitly criticized for lacking cross-device access.
- **Cloud-synced workspaces:** Workona, Toby — valued by teams and multi-device users, but sync reliability is the top complaint ("unreliable per reviews") once adopted at scale.
- **Takeaway:** persistence is not a single toggle; users separate "I want my tabs to survive a crash" (local, must be bulletproof) from "I want my tabs to follow me to another device" (cloud, users tolerate some latency but not silent data loss — both tools got dinged hard for data loss reports).

### 2.4 Density / visual weight
- Vivaldi exposes per-user tab-appearance settings and UI corner-rounding under Settings → Themes; the existence of a dedicated "Customise" right-click menu on tabs signals this is a real, requested axis, not a hidden preference.
- Sidebery's tree can be shown as "simple flat list" or full tree with a configurable tree limit — density and hierarchy depth are tunable together, not separate knobs, in tools built for high tab counts.

### 2.5 Auto-cleanup vs. never-close
- **Suspend/hibernate inactive tabs to save memory:** Vivaldi's native Tab Hibernation ("Hibernate Background Tabs"), Chrome's Memory Saver, and historically The Great Suspender. This group wants tabs to *stay open indefinitely* (never auto-closed) but *not consume RAM* while idle — hibernation, not deletion, is the accepted tradeoff.
- **Bulk unload-to-list:** OneTab's model — tabs are removed from the tab strip entirely and become a static link list, explicitly trading immediate re-access for maximum memory recovery and visual decluttering.
- **Never-auto-close:** Tree Style Tab reviewers explicitly praise it for letting them keep 600–1000 tabs open with no forced pruning — this is a distinct user type (hoarders who treat tabs as a working-memory/to-do-list substitute) that actively resists any tool that auto-closes tabs.

### 2.6 Keyboard vs. mouse/spatial
- Keyboard-driven retrieval (Ctrl+Shift+A / Cmd+Shift+A tab search, `@tabs` omnibox) is explicitly framed in coverage as the fallback once mouse/visual scanning of a packed tab strip stops working — a threshold effect around ~20 open tabs, not a fixed user-type split.
- Spatial/visual users rely on tree position (Tree Style Tab/Sidebery), tab color (Sidebery's per-domain auto-coloring), or tiled split-view (Vivaldi Tab Tiling) to locate tabs by "where it is / what color it is" rather than by typing a query.

### 2.7 File-manager parallel: Dolphin (KDE)
- Tabs (Ctrl+T/Ctrl+W) and split view (F3) are core, low-friction primitives; "switch between split panes with Tab key" is an opt-in behavior setting, not default — mirrors browsers' pattern of shipping a feature default-off until proven wanted.
- The Places panel is fully user-editable: drag any folder in to pin it, right-click to hide/edit/remove entries. Every panel (Places, Terminal, Information, etc.) can be detached, moved, or removed — Dolphin's customization model is panel-level rearrangement, not just content-level settings. This is the file-manager analog of a customizable sidebar/tab-bar chrome, not just tab behavior.

### 2.8 Bookmark/PKM parallel: Raindrop.io
- Explicitly supports **both** folders (Collections) and tags simultaneously, rather than forcing a choice — the vendor's own public stance (via X/Twitter) is a deliberate rejection of fully-automated AI categorization in favor of user-placed folders that an AI can *learn from* over time, keeping the user's manual placement as ground truth.
- This models a preference pattern also visible in tab tools: users want automation to *assist* their existing manual structure (Chrome's AI tab groups are additive/suggested, not replacing manual groups) rather than replace it outright.

## 3. Demand signals: what mainstream browsers were forced to add by user pressure

- **Firefox tab groups.** The single most-upvoted idea on Mozilla Connect since the platform launched in March 2022 — a request posted one day after launch drew roughly 4,500 votes/supporters and 64+ pages of community feedback. Mozilla's own blog frames the release explicitly as "you asked, we built it," crediting a named community-insights role (Jon Siddoway) for surfacing the demand. This is the clearest documented case of a browser vendor building a feature *because* of ranked community voting, not roadmap-driven design.
- **Firefox vertical tabs.** Requested on Mozilla Connect "more than two years" before shipping; Mozilla explicitly says the team evaluated split view, vertical tabs, and sidebar-pinned tabs before landing on the shipped design — while Vivaldi (2015), Edge (2021), and Arc (2023, vertical-only at launch) had already normalized the pattern for years, making Firefox and Chrome (2026) the last two majors to add it. Users had been bridging the gap for years via third-party extensions (Vertical Tabs Reloaded, Vertical Tree Tabs, Tab Center Reborn, Sidebery) — a visible signal that unmet demand routes through the extension ecosystem before it routes through vendor roadmaps.
- **Arc's collapse as a cautionary demand signal.** The Browser Company put Arc into maintenance mode in May 2025, citing failure to reach mainstream adoption — diagnosed in coverage as a "novelty tax": Arc's workspace/sidebar model was different enough from conventional browsing that most users weren't willing to relearn their habits, even though the power-user community that *did* adopt it reacted with what one piece called "grief," not indifference. The lesson for customization design: a radically different *default* paradigm is a hard sell even when a vocal subset loves it; the winning pattern (see Firefox/Chrome above) is opt-in customization layered onto a familiar default, not a wholesale re-architecture.
- **Zen Browser's rise filling the Arc vacuum.** Built on Firefox, open source, and explicitly reproducing Arc's most-loved surface (Workspaces, sidebar, Compact Mode, Split View, a "Mods" system) without the novelty-tax problem, because it's opt-in atop a conventional Firefox base rather than a ground-up reinvention. Coverage frames Zen as having "functionally filled in almost all of Arc's workflow" within about three months. This validates that the demand was specifically for Arc's *organizational features* (workspaces, split view, sidebar tabs), not for Arc's *departure from conventional browser UX* as such.
- **Chrome's AI auto-grouping.** Rolled out as an assistive layer ("Organize similar tabs") on top of Chrome's existing manual tab groups, processing shifted to on-device by Chrome 146 (March 2026) — read as validation that once a manual organization primitive (tab groups) proves popular, the next user-pressure vector is reducing the *labor* of using it, not replacing the primitive itself.
- **Native tab suspension/hibernation as a security-driven category shift.** The Great Suspender's malware incident (Feb 2021) and the repeated pattern of its "successor" extensions being sold and weaponized the same way is a demand signal in the negative: users clearly want memory-saving suspension, but third-party extensions kept betraying that trust, which is a plausible driver for vendors folding hibernation into the browser core (Vivaldi's native Tab Hibernation, Chrome's Memory Saver) rather than leaving it to the extension ecosystem.

## 4. Cross-cutting takeaways

- **Threshold effects, not fixed personas.** Several behaviors (switching from mouse-scanning to keyboard search, switching from tree-nesting to bulk-unload) are triggered by tab *count* crossing a threshold (~20 tabs cited repeatedly) rather than by a stable user "type" — the same person's preferred interaction changes as their tab count grows within a single session.
- **Automation must be additive, never a replacement for manual structure.** Chrome's AI grouping, Raindrop's explicit anti-full-automation stance, and Tree Style Tab reviewers rejecting auto-enabled new defaults all point the same direction: users want automation to propose or assist inside a structure they still control, not take over the structure.
- **Trust is a first-class customization concern for anything that touches tab contents.** Suspension/hibernation tools (Great Suspender) and session managers (Session Buddy's undisclosed telemetry) show that "does this run code against my open tabs and can I trust it not to phone home or get sold" is functionally part of how users evaluate tab tools, on par with feature set.
- **Persistence expectations split along a local-reliability vs. cross-device-sync axis**, and tools get punished hard (in reviews) for failing on *either* axis — data loss on local tools, unreliable sync on cloud tools — rather than users accepting a tradeoff on one axis in exchange for strength on the other.
- **Vendors converge on the same feature set the extension ecosystem already proved out.** Vertical tabs and tab groups both followed the same lifecycle: third-party extension → sustained multi-year community demand (visible in vote counts / long-running extension popularity) → native browser feature. This is a reusable predictor for what a navigation-focused product should treat as "will eventually be expected as native," not optional.

## Sources

- [Reviews for Tree Style Tab – Add-ons for Firefox](https://addons.mozilla.org/en-US/firefox/addon/tree-style-tab/reviews/)
- [Sidebery – Get this Extension for Firefox](https://addons.mozilla.org/en-US/firefox/addon/sidebery/)
- [Sidebery – Hacker News discussion](https://news.ycombinator.com/item?id=36796422)
- [OneTab - Chrome Web Store](https://chromewebstore.google.com/detail/onetab/chphlpgkkbolifaimnlloiipkdnihall?hl=en)
- [Session Buddy - Tab & Bookmark Manager - Chrome Web Store](https://chromewebstore.google.com/detail/session-buddy-tab-bookmar/edacconmaakjimmfgnblocblbcdcpbko?hl=en)
- [Toby: Tab Management Tool - Chrome Web Store](https://chromewebstore.google.com/detail/toby-tab-management-tool/hddnkoipeenegfoeaoibdmnaalmgkpip?hl=en)
- [Workona Reviews - Product Hunt](https://www.producthunt.com/products/workona/reviews)
- [Tabs Outliner - Chrome Web Store](https://chromewebstore.google.com/detail/tabs-outliner/eggkanocgddhmamlbiijnphhppkpkmkl?hl=en)
- [The great suspender (once again) contains malware | Partizion](https://www.partizion.io/blog/the-great-suspender-contains-malware)
- [This hugely popular Chrome extension was riddled with malware - TechRadar](https://www.techradar.com/news/this-hugely-popular-chrome-extension-was-riddled-with-malware)
- [You asked, we built it: Firefox tab groups are here | The Mozilla Blog](https://blog.mozilla.org/en/firefox/tab-groups-community/)
- [Native Tab Grouping / More Customizable Tab bar - Mozilla Connect](https://connect.mozilla.org/t5/ideas/native-tab-grouping-more-customizable-tab-bar/idi-p/303)
- [How Firefox's vertical tabs came to life with a little help from our community | The Mozilla Blog](https://blog.mozilla.org/en/firefox/vertical-tabs-and-the-firefox-community/)
- [Firefox 136 launches with vertical tabs - gHacks Tech News](https://www.ghacks.net/2025/03/04/firefox-136-launches-with-vertical-tabs-but-one-requested-feature-is-still-missing/)
- [The Arc Browser Is Dead - How-To Geek](https://www.howtogeek.com/the-arc-browser-is-dead/)
- [Is Arc Browser Dead? What Actually Happened - SupaSidebar](https://supasidebar.com/blog/is-arc-browser-dead)
- [5 reasons why Zen still can't replace Arc for me - XDA Developers](https://www.xda-developers.com/reasons-why-zen-still-cant-replace-arc-for-me/)
- [Switching from Arc to Zen Browser Three | LuMakes](https://lumakes.com/en/articles/zen-browser-setup/)
- [GitHub - abrahammurciano/firefox-css-tweaks](https://github.com/abrahammurciano/firefox-css-tweaks)
- [Vertical tabs in Firefox: Yes, it's really possible! - PCWorld](https://www.pcworld.com/article/823939/vertical-tabs-in-firefox-yes-its-really-possible.html)
- [Tab Stacks | Vivaldi Browser Help](https://help.vivaldi.com/desktop/tabs/tab-stacks/)
- [10 Tab Management Tricks in Vivaldi browser](https://vivaldi.com/blog/tab-management-tricks-vivaldi/)
- [Tab Features | Vivaldi Browser Help](https://help.vivaldi.com/desktop/tabs/tab-features/)
- [17 Dolphin File Manager Tips and Tweaks for KDE Users - It's FOSS](https://itsfoss.com/dolphin-tweaks/)
- [Dolphin/File Management - KDE UserBase Wiki](https://userbase.kde.org/Dolphin/File_Management)
- [Raindrop.io on X: AI-powered auto organization](https://x.com/raindrop_io/status/1726946271464747248)
- [Bookmarks - Raindrop.io Help](https://help.raindrop.io/bookmarks)
- [Microsoft Edge's vertical tabs are weird but I can't give them up - PCWorld](https://www.pcworld.com/article/785315/edges-vertical-tabs-are-weird-but-i-cant-give-them-up.html)
- [Microsoft is working to refine this divisive Edge feature - TechRadar](https://www.techradar.com/news/microsoft-is-working-to-refine-this-divisive-edge-feature)
- [This underrated Google Chrome feature turned me into a power user - Digital Trends](https://www.digitaltrends.com/computing/google-chrome-automatic-tab-groups/)
- [Chrome AI Tab Organizer: Practical Guide - TabGroupVault](https://tabgroupvault.com/blog/chrome-ai-tab-organizer-guide)
- [Tab Manager Plus for Chrome - Chrome Web Store](https://chromewebstore.google.com/detail/tab-manager-plus-for-chro/cnkdjjdmfiffagllbiiilooaoofcoeff)
- [4 killer tab search tricks for Chrome and Microsoft Edge - PCWorld](https://www.pcworld.com/article/395118/4-killer-tab-search-tricks-for-chrome-and-microsoft-edge.html)
- [Keyboard shortcut for "Search Tabs" - Mozilla Connect](https://connect.mozilla.org/t5/ideas/keyboard-shortcut-for-quot-search-tabs-quot/idi-p/9134)
