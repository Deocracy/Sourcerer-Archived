# Tab Management Paradigms — Full History & Taxonomy

Research date: 2026-08-19. Scope: every significant approach to managing many
open documents, from the first MDI browsers (1994) to the AI-browser era (2026),
plus the non-browser lineages (IDEs, file managers, terminal multiplexers) that
solved the same problem differently.

---

## 0. The problem being solved

Tabs were introduced when a browsing session was 3–8 pages. They have barely
changed since. The authoritative modern study is Chang et al., **"When the Tab
Comes Due: Challenges in the Cost Structure of Browser Tab Usage"**, CHI 2021
(CMU HCII — Joseph Chee Chang, Nathan Hahn, Yongsung Kim, Julina Coupland,
Bradley Breneisen, Hannah S. Kim, John Hwong, Aniket Kittur). It explicitly
frames itself as the first in-depth tab study in over a decade.

Its core finding is a **cost structure**, not a feature gap. Two opposing
pressures act on every tab:

- **Pressures to keep it open** — the tab is a reminder ("prospective memory
  cue"); re-finding cost is high and uncertain; closing it feels like losing
  work already done; a tab holds transient state (scroll position, form input,
  auth session) that bookmarks do not capture.
- **Pressures to close it** — limited screen width shrinks tabs to unreadable
  favicons; RAM/CPU pressure; attention cost; the visible clutter itself causes
  anxiety.

Numbers (all soft, methodology varies):

- Chang et al.: median 8 open tabs (Q1 = 5, Q3 = 12), range from <3 to ~400.
- Popular 2025–26 aggregations put the Chrome per-session mean around 11 tabs,
  cross-browser ~10.
- Power-user surveys routinely find 40–50% of respondents above 20 tabs.
- Roughly a quarter of surveyed users call browser clutter a "somewhat or
  serious" concern; ~57% call it mild; ~18% are not bothered.

**Key structural insight:** every paradigm below is an attempt to change one of
those costs. They fall into a small number of families:

1. **Spatial** — give tabs more room or a second dimension (vertical lists,
   trees, zoomable canvases, tiling).
2. **Hierarchical** — introduce a container above the tab (groups, stacks,
   workspaces, spaces, profiles).
3. **Temporal / archival** — move tabs off the strip into a list you can return
   to (session managers, OneTab, read-later queues).
4. **Retrieval-first** — stop organizing, make finding cheap (tab search,
   command palettes, fuzzy switchers).
5. **Resource relief** — keep the tab but stop paying for it (suspend/discard,
   sleeping tabs, memory saver).
6. **Semantic / agentic** — the browser understands the tabs (Dia, AI
   grouping, SigmaOS task model).

Nearly every commercially successful feature is family 2, 4, or 5. Families 1
and 3 dominate the passionate-minority extension ecosystem. Family 6 is where
2025–26 money went.

---

## 1. Prehistory: MDI and the invention of the tab (1994–2006)

| Year | Product | What it did |
|---|---|---|
| 1994 | **BookLink InternetWorks** | Tabbed windows — widely cited as the first tabbed browser, though its tabs represented the navigation stack rather than independent documents. Bought by AOL. |
| ~1996 | **Opera (MDI)** | Multiple document interface — child windows inside one parent window, with minimize/tile/cascade. Not tabs; the ancestor of tabs. Opera's MDI heritage is why Vivaldi (founded by Opera co-founder Jon von Tetzchner) is still the most window-manager-like browser. |
| 1997–98 | **SimulBrowse → NetCaptor** (Adam Stiles) | An IE shell. First release with tabbed browsing shipped January 1998. This is the implementation most modern browsers actually descend from. |
| 1999 | **IBrowse** (Amiga) | Tabbed browsing on Amiga. |
| 2000 | **Opera 4** | Tabs proper in a mainstream browser. |
| 2001 | **Mozilla 0.9.5** | Built-in tabbed browsing (via the MultiZilla/tabbed-browsing extension work in 2000). |
| 2002–04 | **Phoenix / Firebird / Firefox** | Tabs become the mass-market default; the "get Firefox, get tabs" pitch. |
| 2003 | **Safari 1.0** | Tabs on macOS. |
| 2006 | **Internet Explorer 7** | Last holdout adopts tabs; the paradigm is now universal. |

**Status:** absorbed completely. MDI itself is dead in browsers (it survives in
IDEs — see §12).

**Why this matters:** the tab strip won because it is *linear, horizontal, and
zero-configuration*. Every paradigm since has had to beat "does nothing, costs
nothing." Almost none have.

---

## 2. Zoomable / spatial canvas: Firefox Panorama (TabCandy)

- **Names:** TabCandy (prototype, 2010) → Tab Sets → Panorama → "Tab Groups"
  (shipped name).
- **Origin:** Aza Raskin (then Mozilla Labs creative lead), with Ian Gilman,
  Michael Yoshitaka Erlewine, Raymond Lee, Sean Dunn, Kevin Hanes, Tim Taubert.
  Sketch-to-ship took roughly seven months.
- **Shipped:** Firefox 4, March 2011. Keyboard shortcut Ctrl+Shift+E.
- **Mechanism:** a full-screen zoom-out view where every tab becomes a live
  thumbnail on a 2D canvas. You drag thumbnails into rectangular "groups";
  resizing a group re-flows it; groups can be named. Returning to the browser
  shows only the active group's tabs in the tab strip. Ctrl+Tab could be scoped
  to the current group. Essentially: *the browser as a desk you can stand back
  from.*
- **User problem:** context separation without extra windows, plus visual
  (thumbnail) recall instead of text recall. It attacked re-finding cost with
  spatial memory.
- **Adoption:** the killer number is Mozilla's own — **about 0.01% of users**
  engaged with it. Press love (TechCrunch, Lifehacker) never converted.
- **Death:** Bugzilla 1222550 "Product plan: remove tab groups (panorama)".
  Deprecation notice in Firefox 44; removed in **Firefox 45, March 2016**.
  Stated reasons: near-zero usage, a large accumulated bug/usability backlog,
  and a maintenance cost that could not be justified — critically, Panorama was
  deeply entangled with the tab implementation, blocking other work
  (Electrolysis/multiprocess in particular). On removal, each group's tabs were
  migrated into a **bookmarks folder per group**.
- **Afterlife:** community add-ons ported it (quicksaver's "Tab Groups"
  add-on, later "Tiled Tab Groups" on AMO, still versioned in 2020s). The
  WebExtensions transition (Firefox 57, 2017) killed the faithful ports because
  no WebExtension API can replace the tab strip.
- **Who liked it:** researchers, students, people who think spatially and keep
  parallel projects. A tiny, loud group.
- **Verdict / lesson:** the canonical cautionary tale. A beautiful, heavily
  covered feature can be a 1-in-10,000 feature. It failed because it was
  **modal** (you leave browsing to manage), **manual** (grouping is work done
  up front with no immediate payoff), and **invisible** (nothing in the default
  UI told you it existed). Every later group feature avoided all three:
  Chrome's groups are inline, incremental, and visible in the strip.

---

## 3. Vertical tabs and the tree paradigm

### 3a. Tree Style Tab (TST)

- **Origin:** Piro (Hiroshi Kamiyama / piro_or), Firefox extension, **first
  released 2007** — nearly two decades of continuous maintenance, including a
  full rewrite for WebExtensions after Firefox 57.
- **Mechanism:** replaces the horizontal strip with a sidebar in which tabs are
  a *tree*: a tab opened from a link becomes a child of its opener. Subtrees
  collapse, drag to re-parent, close-subtree, "promote to top level." The tree
  is derived automatically from browsing behaviour — this is its central
  insight and the reason it beat Panorama on effort-per-payoff.
- **User problem:** research branching. Twenty tabs from one Wikipedia article
  are one collapsible node, not twenty favicons.
- **Adoption (checked 2026-08-19):** AMO lists **136,347 users**, 4.5 stars from
  2,266 reviews, last updated July 2026. That is large for a power-user
  extension and minuscule against Firefox's user base — the same ~0.05% band
  Panorama died in, except it survives because its author, not Mozilla, pays
  for it.
- **Who:** developers, academics, researchers, anyone whose browsing is
  depth-first.

### 3b. Sidebery

- **Origin:** mbnuqw, ~2019, Firefox. Mozilla **Recommended** extension, MIT.
- **Mechanism:** vertical tab tree *plus* first-class **panels** (multiple
  independent tab lists in one sidebar) and native binding to **Container
  Tabs** — so a panel can be a container, i.e. its own cookie jar. Bookmarks
  panel in the same sidebar. Effectively TST + workspaces + containers.
- **Adoption:** AMO **50,926 users**, 4.7 stars, 1,067 reviews, updated July
  2026. Smaller than TST but higher-rated; it is the modern default
  recommendation for the tree crowd.

### 3c. Other tree implementations

- **Sidewise Tree Style Tabs** (Chrome, ~2012–2015) — the Chrome answer to
  TST, rendered in a docked popup window because Chrome has no sidebar API.
  Abandoned; MV3 finished it.
- **Tree Tabs** (Firefox/Chrome, ~2018) — highly configurable tree sidebar with
  its own layout engine; niche, still around, low install base.
- **Tabs Outliner** (Chrome, ~2013, Vladyslav Volovyk) — see §8.

### 3d. Vertical tabs going mainstream

This is the single clearest case of a power-user paradigm being absorbed.

| Browser | Vertical tabs since |
|---|---|
| Vivaldi | 2015 (launch — tab bar position: left/right/bottom) |
| Microsoft Edge | **March 2021** — first mainstream browser with a native vertical tab sidebar; redesigned again April 2025 |
| Arc | 2023 (sidebar was the *only* tab UI) |
| Firefox | Labs/Nightly in 131 (2024) → **shipped to release in Firefox 136, March 2025**, refined through 138+ |
| Zen | 2024 (sidebar-first, Arc-like) |
| Google Chrome | **Chrome 145–146, staged rollout Q1–Q2 2026** — right-click tab strip → "Show tabs vertically", or Settings → Appearance → Tab strip position → Side |

Notably the mainstream versions are **flat lists, not trees**. Vendors took the
cheap 80% (readable titles, room for 40 tabs, wide-screen friendly) and left
the hierarchy — the expensive, opinion-heavy part — to extensions. Edge began
re-promoting vertical tabs in April 2026 specifically because Chrome had
shipped it.

---

## 4. Tab groups (the hierarchy that actually won)

### 4a. Google Chrome Tab Groups

- **Shipped:** Chrome 83, **19 May 2020**, staged rollout.
- **Mechanism:** right-click a tab → "Add to new group". A named, coloured pill
  appears inline in the strip; click the pill to collapse the whole group to
  the pill. Drag tabs in/out. **2024** added *Saved tab groups* — a group can be
  closed and kept, restored later, and synced across devices (requires
  Chrome Sync + sign-in). 2026 renders groups as collapsible headers inside the
  new vertical tab panel.
- **Why it won where Panorama lost:** non-modal (you never leave the page),
  incremental (group two tabs, get value immediately), visible (the pill lives
  in the strip you already look at), and zero-loss (ungrouping is free).
- **Adoption:** shipped on by default to a ~65% market-share browser; it is now
  the de facto meaning of "tab group" across the industry. Every other vendor
  copied the pill-in-strip form.
- **Who:** everyone; especially people juggling 2–4 concurrent projects who do
  not want a second window.

### 4b. Firefox Tab Groups (2024–25 revival)

Nine years after killing Panorama, Mozilla shipped groups again — with the
opposite design philosophy.

- **Timeline:** landed behind a flag in **Firefox 133** (Nov 2024); enabled for
  most users on **Firefox 138** (2025); built in for all users as of **Firefox
  141** (2025); Firefox 145 added "peek inside a group without expanding it."
- **Mechanism:** drag one tab onto another and pause → a group is created →
  name it and pick a colour. Save-and-close a group to reclaim strip space;
  saved groups reappear in the new-tab / tab-overview surfaces. Works with
  native vertical tabs.
- **Adoption:** Mozilla reported **over 18 million people** used tab groups in
  the first year — versus Panorama's 0.01%. Same company, same nominal feature,
  three orders of magnitude difference, entirely due to interaction design and
  default-on placement.
- **Lesson (the single most important data point in this document):** the
  paradigm was never the problem. Modality and discoverability were.

### 4c. Safari Tab Groups

- **Shipped:** Safari 15 / iOS 15 / iPadOS 15 / macOS 12 Monterey, **2021**.
- **Mechanism:** groups live in the sidebar and are closer to *named saved
  sessions* than Chrome-style inline pills — switching a group swaps the entire
  set of visible tabs. **iCloud-synced across devices** from day one; that sync
  is Safari's differentiator. **iOS 16 / macOS Ventura (2022)** added **Shared
  Tab Groups** — collaborative, live-updating groups shared with other people,
  with presence indicators. **Safari 17 (2023)** added **Profiles**, and tab
  groups nest under a profile.
- **Who:** Apple-ecosystem users doing trip planning, shopping research, and
  household/team coordination — a noticeably less technical audience than the
  tree-tab crowd.
- **Status:** alive, mainstream, and the only mainstream implementation with
  real-time multi-user sharing.

### 4d. Others

- **Vivaldi Tab Stacks** — see §7 (predates all of the above).
- **Opera Workspaces** — Opera 67, **February 2020**, three months before
  Chrome's groups; a workspace is an icon in the left rail that swaps the whole
  tab strip. Also in Opera GX (May 2020).
- **Edge** — inherits Chromium tab groups; added **Collections** (2020, a
  side-panel of saved link cards with notes and Excel/Word export) and **Edge
  Workspaces** (2023, *shared* named tab sets for teams, browser-side
  collaboration aimed at enterprise).

---

## 5. Workspaces, spaces, and profiles (the container-above-the-tab)

The distinction that matters: **a group is a label on tabs that stay in one
strip; a workspace swaps the strip.** Groups are additive, workspaces are
exclusive. Workspaces attack *attention* cost rather than *space* cost.

### 5a. Arc Browser — Spaces (the high-water mark)

- **Origin:** The Browser Company of New York, Josh Miller. Beta 2021, wide
  release 2022–23, macOS-first.
- **Mechanism, and it is a real re-think:**
  - Sidebar-only; no horizontal tab strip exists.
  - **Spaces** — swipeable columns of the sidebar. Each has its own theme
    colour, its own pinned tabs, and optionally its own **profile** (separate
    cookies/logins). Switching space repaints the whole browser.
  - **Three tab tiers:** Favourites (icon row, pinned across all spaces) →
    Pinned tabs (persistent within a space, behave like bookmarks-that-are-open
    and never auto-close) → Today's tabs (ephemeral).
  - **Auto-archive** — this is the crucial mechanic. Unpinned tabs
    automatically disappear after a set interval (default 12 hours, adjustable
    to 30 days) into a searchable Archive. Tab bankruptcy is declared *for* you,
    on a timer.
  - Split View, Easels (canvas), Boosts (per-site CSS/JS), Little Arc
    (ephemeral single-page window for links from other apps).
- **User problem:** it inverted the default. Tabs are garbage-collected unless
  you say otherwise; the strip never fills up because it cannot.
- **Adoption:** enormous press and designer/PM mindshare, modest real market
  share (never a measurable slice of global browser share). macOS-only for most
  of its life; Windows arrived late and thin.
- **Status:** **effectively dead.** Feature freeze October 2024; on **27 May
  2025** Josh Miller announced in the "Keeping Tabs" newsletter that Arc was
  being sunset in favour of **Dia**. **Atlassian acquired The Browser Company
  for $610M, closed 21 October 2025**; the whole team now works on Dia. Arc
  remains downloadable at arc.net in maintenance mode with Chromium security
  updates only, and as of mid-2026 there is no realistic path to new features.
- **Who liked it:** designers, PMs, founders, students — an aesthetics-forward,
  non-engineer power user. Its refugees went to Zen, Vivaldi, and SigmaOS.

### 5b. Vivaldi Workspaces

- **Shipped:** Vivaldi 6.0, **2023**.
- **Mechanism:** named, icon-tagged workspaces that swap the visible tab set;
  tab stacks nest *inside* workspaces; tab tiling works within them. Vivaldi is
  the only browser that composes all three layers (workspace → stack → tile).
- **Status:** alive, actively promoted (2025 press coverage). Vivaldi's share is
  ~0.1–0.5% globally but its users are extreme power users.

### 5c. Zen Browser

- **Origin:** open-source **Firefox fork**, launched 2024, stable 2025. ~42k
  GitHub stars, weekly builds.
- **Mechanism:** explicitly "Arc without the AI and without the company risk."
  Sidebar-first vertical tabs, **Workspaces** (each an isolated set of tabs +
  pinned tabs, optionally bound to a **Firefox Container** for cookie
  isolation — the cleanest fusion of workspace and container anywhere), **Split
  View** of 2–4 tabs in a binary-tree layout (Chrome's split is 2-pane only),
  Compact Mode (chrome disappears entirely), Glance (peek at a link in an
  overlay).
- **Status:** alive and the main beneficiary of Arc's death. This is the
  archetype for "Firefox fork + rebuilt tab GUI" and directly relevant to the
  TheiaIDE project's thesis.

### 5d. Opera Workspaces / SigmaOS / Edge Workspaces

- **Opera** (Feb 2020) — icon rail, swaps tab strip. Alive, unremarkable, and
  notably *pre-dates* Chrome tab groups.
- **SigmaOS** (2021, macOS, Chromium; Fast Company: "this radical new browser
  will transform your relationship with tabs") — **workspaces + tabs-as-tasks**.
  Every tab is a to-do item you can mark done (which removes it), snooze to a
  time, or lock. Keyboard-driven. Still shipping in 2026, small.
- **Edge Workspaces** (2023) — shared named tab sets, enterprise/team framing.

### 5e. Containers and profiles (the *identity* axis)

Not tab management per se, but constantly conflated with it, and the two keep
merging.

- **Firefox Multi-Account Containers** (Mozilla extension, **2017**, built on
  the `contextualIdentities` API from the Contextual Identity Project) — a tab
  is tagged with a container (Personal, Work, Banking, Shopping); each container
  is a separate cookie/storage jar with a coloured underline on the tab. Site
  → container assignment can be automatic. **Facebook Container** (2018) was the
  breakout consumer spin-off.
- **Chrome Profiles** — heavier: separate window, separate everything,
  including extensions and history. Coarse but universally available.
- **Safari Profiles** (Safari 17, 2023) — profiles contain tab groups.
- **Convergence:** Arc bound spaces to profiles; Zen binds workspaces to
  containers; Sidebery binds sidebar panels to containers. Everyone
  independently concluded that "which project am I in" and "which identity am I"
  are usually the same question.

---

## 6. Session managers (the archival family)

Core flow: **dump the whole window/session to a stored list → close it →
restore later.** Solves the "I need my RAM and my strip back but I cannot lose
this" bind. The oldest continuously-alive extension category.

| Product | Origin | Mechanism | Adoption (2026-08-19) | Status |
|---|---|---|---|---|
| **Session Buddy** | ~2011, Chrome | Full session snapshot manager: saved sessions with window structure preserved, search across saved tabs, crash recovery, local-first/privacy-first | **~900,000 users**, 4.7★ from 25.1k ratings, updated Apr 2026 | Alive, the category leader |
| **OneTab** | 2013, Chrome+Firefox | One button: convert *all* open tabs into a single list page and close them. Immediate RAM relief. Restore individually or all. Shareable web-page export of a list | **~2,000,000 users**, 4.4★ from 14.6k ratings, updated Jul 2026 | Alive; the most-installed tab tool ever |
| **Tab Session Manager** | ~2017, cross-browser (FF/Chrome/Edge) | Auto-saves sessions on a timer, manual named saves, cloud sync via Google Drive, open-source | Hundreds of thousands across stores | Alive |
| **Tab Stash** | ~2019, Firefox | Stashes tabs into *bookmarks* rather than a private database — so it syncs via Firefox Sync for free and survives the extension dying | Niche, well-loved | Alive |
| **TabXpert, Sessionic, Super Session Manager** | 2020s | MV3-era entrants filling gaps after older tools decayed | Small | Alive |

**Why OneTab is 2× everything else combined:** it has exactly one interaction
and it delivers its payoff (memory freed, strip cleared) in one click, with
zero taxonomy work. Session Buddy asks you to think about sessions; OneTab asks
nothing. The install-count gap between them is the strongest available evidence
that **effort-before-payoff is the binding constraint on tab tooling
adoption.**

**Who:** OneTab — anyone who hits 60 tabs and panics. Session Buddy —
methodical people and crash-burned researchers. Tab Session Manager — Firefox
users who want automation.

**Absorption:** browsers have partly eaten this. Chrome's Saved Tab Groups
(2024), Firefox's save-and-close groups (2025), and Safari's synced Tab Groups
(2021) each cover the common case. The extensions survive on *search across
archives* and *cross-device/backup* — which the natives still do poorly.

---

## 7. Stacking and tiling (Vivaldi's lineage)

- **Vivaldi Tab Stacks** — present essentially since Vivaldi's 2015 launch,
  with direct design ancestry in Opera's MDI era. Drag one tab onto another to
  stack. Vivaldi 2.3 (**March 2019**) added *auto-stacking* by domain. Three
  render styles: **compact** (a single tab showing the stack), **two-level**
  (a second strip appears below showing the stack's members), and **accordion**
  (the stack expands in place). It is Chrome's tab groups, five years earlier,
  with more display options.
- **Vivaldi Tab Tiling** — select 2+ tabs → tile them side by side, in a grid,
  or stacked, *inside one browser window*, with a real splitter. Combined with
  stacks: tile a whole stack in one action. This is the browser adopting the
  **window manager** model rather than the document model.
- **Others tiling:** Zen Split View (2–4 panes, binary tree), Arc Split View,
  Edge Split Screen (2023, 2 panes), Chrome split-screen (2025, 2 panes),
  Safari does not.
- **Who:** people comparing documents — translators, developers reading docs
  against code, analysts, shoppers.
- **Status:** alive and slowly absorbed. Every major browser now has 2-pane
  split; only Vivaldi and Zen do n-pane.

---

## 8. Unusual and experimental

- **Tabs Outliner** (Chrome, ~2013, Vladyslav Volovyk) — the most conceptually
  ambitious tab tool ever shipped. It is a **persistent outliner document** of
  your entire browsing life: open tabs, closed tabs, windows, and hand-typed
  notes all live in one editable tree in a separate window. Closing a tab does
  not delete it — it greys out and stays in the tree as a restorable node. You
  can annotate, re-order, and file browsing sessions like an org-mode file.
  Paid ($15 one-time historically). Faced extinction with the Manifest V2
  sunset; **updated to MV3 in August 2024** and still shipping (v1.4.x). Cult
  following; its users describe it as a second brain, and its detractors
  describe it as a second job.
- **Sidewise Tree Style Tabs** (Chrome, ~2012) — TST for Chrome via a docked
  popup window. Dead.
- **BrainTool** (2021–) — outliner/tab-manager hybrid that stores everything as
  a plain-text **org-mode file**; explicitly markets itself as the Tabs Outliner
  successor. Tiny, alive, beloved by the org-mode crowd.
- **Stack Browser** (2021–, Chromium) — pages are **cards**, cards group into
  **stacks**, stacks into **spaces**. Effectively a Tweetdeck/dashboard model
  for the web, aimed at people who live in 6 SaaS apps at once. Still
  developed as of the latest public information; roadmap talks about becoming
  "multiplayer". Very small.
- **SigmaOS** (2021–) — tabs as **tasks** (mark done, snooze, lock); workspaces
  in a left rail; heavy keyboard focus. Alive, small, macOS-only.
- **Dia** (The Browser Company → Atlassian, 2025–) — the AI-native successor to
  Arc. Chromium, macOS 14+/Apple Silicon; Windows behind a waitlist; no
  Linux/iOS. Version 1.33.0 as of mid-2026; cross-device Sync shipped in 1.26.0.
  Tab management is now **semantic**: the assistant reads across your open tabs
  and answers over them, and **Skills** let you save a natural-language
  instruction as a reusable command. Dia Pro $20/month. This is the industry's
  current bet: not a better tab *container*, but a layer that makes the
  container matter less because retrieval is conversational.
- **Spatial/canvas tools outside browsers** (Kosmik, Heptabase, tldraw-likes,
  Arc's Easels) — the Panorama idea reborn as a note-taking app rather than a
  browser feature. Notably this is where the spatial paradigm actually found
  product-market fit: **as a research surface, not as a tab strip.**
- **Firefox "Tab Groups" community add-ons / Tiled Tab Groups** — the Panorama
  afterlife, kept alive by quicksaver and successors until WebExtensions
  starved them.

---

## 9. Retrieval-first: search and switchers

The paradigm that quietly won on the *cheap* end: don't organize, just find.

- **Chrome Tab Search** — tested Oct 2020, shipped in Chrome 87/88 (2020–21) as
  the caret/down-arrow at the right end of the tab strip; `Ctrl/⌘+Shift+A`.
  Searches open tabs *and* recently closed ones across all windows. Later
  extended to tabs on other devices.
- **Firefox** — the "List all tabs" dropdown (V arrow), plus the **`%` keyword
  in the address bar** which searches open tabs and switches to them ("Switch to
  Tab"), a genuinely old Firefox feature (Awesomebar era, 2008) that most users
  have never noticed.
- **Vivaldi Quick Commands** — `F2` / `⌘E`. A universal command palette over
  open tabs, bookmarks, history, settings, browser commands, and web search.
  Predates and exceeds Chrome's tab search; it is the browser equivalent of
  VS Code's `⌘P`/`⌘⇧P`.
- **Safari** — `⌘⇧\` tab overview grid with a search field.
- **Extensions:** Quick Tabs, Search All Tabs (2020, full-text search of
  *content* not just titles), Tab Manager Plus. Several hundred thousand
  installs collectively.
- **User problem:** direct attack on re-finding cost — the single largest
  driver of tab-keeping in the CHI 2021 study. If finding a closed tab were
  free, hoarding would stop.
- **Who:** keyboard-driven users, developers. Also, importantly, *everyone
  else by accident*, since Chrome ships it on by default.
- **Status:** universally absorbed. This is the paradigm with the best
  effort-to-payoff ratio in the entire taxonomy: zero setup, immediate value.

---

## 10. Read-later queues (moving the item out of the browser entirely)

| Product | Timeline | Mechanism | Status |
|---|---|---|---|
| **Read It Later → Pocket** | 2007 as *Read It Later* (Nate Weiner); renamed **Pocket** 2012; acquired by **Mozilla 2017**; bundled into Firefox's new-tab and toolbar | Save URL to a cloud queue; server-side article extraction into clean reader text; offline sync; tagging; recommendations | **DEAD.** Announced May 2025, **shut down 8 July 2025**; export window closed 8 October 2025 when all data was deleted. Mozilla cited changed content-consumption habits (and killed Fakespot at the same time) |
| **Instapaper** | 2008 (Marco Arment) → Betaworks → Pinterest 2016 → **management buyout 2018** → independent | The original clean-reader read-later app; text-first, highlights, speed-reading, Kindle delivery | Alive, independent, small, profitable-by-being-small |
| **Safari Reading List** | **2011 (Safari 5.1 / iOS 5)** | Built-in, iCloud-synced, offline-saved list; one click from the share menu | Alive, invisible, and probably the most-*used* read-later feature in the world by sheer default distribution |
| **Toby** | 2017, Chrome | Replaces the new-tab page with a **visual board** of collections of saved tabs; drag tabs into columns; team sharing | Alive but enshittified: **free tier capped at 60 saved tabs**, Chrome-only, v1.13.1 (June 2026) |
| **Workona** | 2018 | **Workspace manager as a web app** — cloud workspaces containing tabs, notes, and links to Drive/Notion/Jira resources; the tab is one resource type among many; strong team/collaboration angle | Alive, commercial. Free tier now capped at **5 workspaces**; Pro ~$8/month |
| **Readwise Reader** | 2022 | Read-later + highlight review + RSS + newsletters, one inbox | Alive; the main Pocket refugee destination |
| **Raindrop.io, Matter, Omnivore (dead 2024, acq. ElevenLabs), Wallabag (self-hosted)** | — | Bookmark/read-later variants | Mixed |

**Core flow:** "this is not for now" → one click → the tab closes → the item
appears in a *different application* with its own queue semantics. The
conceptual leap is admitting that most open tabs are not tasks but **intentions**,
and intentions belong in an inbox, not a strip.

**Who:** readers and knowledge workers rather than multi-project jugglers.

**The Pocket lesson:** even with Mozilla's distribution — a button in Firefox's
default toolbar and content on every new-tab page for eight years — a
read-later product could not sustain itself. Saving is easy and free; *reading
the queue* is the hard part, and most saved items are never opened. Read-later
queues are where tabs go to die politely.

---

## 11. Resource relief: suspend, discard, sleep

Not organization at all — a pressure valve that removes the *cost* of keeping
tabs so no organization is needed. Arguably the most consequential family,
because it changed what "too many tabs" means.

- **The Great Suspender** (Chrome, ~2013, Dean Oemcke) — auto-suspends tabs
  after N minutes, replacing them with a lightweight placeholder that reloads
  on click; whitelists, per-tab pinning, form-state warnings. Peaked at **2M+
  installs**.
  **Death:** the original author sold the extension to an unknown buyer in 2020;
  the new maintainer shipped remote-code-execution and tracking capability used
  for ad fraud. Google **removed it from the Web Store and remotely disabled it
  on users' machines in February 2021** — an unusually aggressive action. Forks
  ("The Marvellous Suspender", TGS-without-analytics) survive but never regained
  scale. It remains the canonical supply-chain-attack story of the extension
  ecosystem.
- **Native replacements (the absorption):**
  - **Edge Sleeping Tabs** — 2020/2021, based on Chromium's freezing; reported
    ~26% average memory saving per sleeping tab; on by default with a
    configurable timer.
  - **Chrome Memory Saver** — Chrome 108/110, 2022–23; discards inactive tabs,
    surfaces "memory saved" in the UI, with an always-keep-active exception
    list. On by default.
  - **Firefox** — automatic tab unloading under memory pressure since ~2021
    (`about:unloads`); lazy session restore since Firefox 4 (2011) — arguably
    Mozilla shipped the first version of this idea.
  - **Safari** — aggressive background-tab throttling and page-cache eviction.
- **Second-order effect:** once the browser silently pays the RAM cost for you,
  the *visible clutter* cost is all that remains — which is exactly why the
  2020s wave of paradigms (groups, vertical tabs, spaces, auto-archive) is
  about **attention and legibility**, not performance. Suspension solved the
  1990s–2010s complaint and left the 2020s one exposed.
- **Who:** everyone; invisible by design. The most-used tab management feature
  on earth is one nobody thinks of as tab management.

---

## 12. Tab management OUTSIDE browsers

These lineages solved the same surface problem with different constraints — and
in almost every case they are *ahead* of browsers, because the documents are
cheap, local, and re-derivable.

### 12a. IDE editor tabs — VS Code

- **Preview tabs** (italic title): single-clicking a file in the explorer or a
  search result opens it in a *reusable* slot — the next preview replaces it.
  Double-click, edit, or explicit pin promotes it to a permanent tab. **This is
  the single best idea in the whole taxonomy and no browser has it.** It solves
  tab accumulation at the *source* by distinguishing "looking" from "keeping".
  A browser equivalent would be transformative: middle-clicked links open as
  preview tabs and are auto-recycled.
- **Pinned tabs** — VS Code **1.46, May 2020**; refined in 1.50/1.51 (pinned
  tabs shrink to icons, can be moved to a separate first tab row). Pinned tabs
  never get recycled by preview and survive "close others".
- **Editor Groups** — the main area is a recursive **split grid**, not a strip.
  Each group has its own independent tab bar and its own active editor. Drag a
  tab into a group; `⌘1/2/3` targets a group. This is the tiling paradigm as
  the *default*, not an add-on.
- **Other mechanisms:** `⌘P` fuzzy Quick Open (retrieval-first, and the
  ancestor of Vivaldi Quick Commands), `workbench.editor.limit.value` (a hard
  cap on open editors, auto-closing the least-recently-used — **automatic tab
  bankruptcy, opt-in, per-group**), tab scroll vs. wrap, "Open Editors" list
  view in the sidebar, and full workspace layout persistence.

### 12b. Eclipse Theia (relevant to this project)

Theia's model is strictly more general than a browser's, and worth stating
precisely since we have the source locally at
`sources/theia/packages/core/src/browser/shell/`:

- **`ApplicationShell`** partitions the window into areas:
  `'main' | 'top' | 'left' | 'right' | 'bottom' | 'secondaryWindow'`.
- The main and bottom areas are **`TheiaDockPanel`** (a Lumino/PhosphorJS
  `DockPanel`): a recursive tree of split panels whose leaves are tab bars.
  Any widget can be dragged to any leaf, split in any direction, or dropped
  into a new tab bar. There is no privileged "tab strip" — the strip is just
  the leaf node of a layout tree.
- **`secondaryWindow`** — a widget can be detached into a separate OS window
  and dragged back. Browsers can move a tab to a new window, but cannot move a
  *pane* out of a tiled layout and back.
- **`ShellLayoutRestorer`** serializes the entire layout tree (splits, tab
  order, active widget, area sizes) and restores it on startup — sessions are
  layout-complete, not just URL-lists.
- Side panels are **`SidePanelHandler`** with icon rails and per-view tab bars
  — the "vertical tabs" idea, but for tool views rather than documents.

The gap versus browsers is not capability, it is *cost model*: Theia widgets
are cheap and stateless-ish; browser tabs carry expensive processes and
irreproducible authenticated state, which is exactly why browsers cannot
casually close and re-open documents the way IDEs do.

### 12c. File managers — Dolphin, and the split-view lineage

- **Dolphin** (KDE) — tabs since KDE 4 (2008), plus **F3 split view**: each
  tab contains *two* independent panes with independent navigation and
  selection. `Ctrl+T` new tab, tabs persist across sessions, and each tab
  remembers per-pane state.
- Lineage: **Norton Commander (1986) → Midnight Commander → Total Commander →
  Double Commander / Krusader** — the "two panes, tabs in each pane" orthodox
  file manager model, which is 20 years older than browser tabs and still the
  power-user standard.
- **The distinguishing idea:** the split panes exist to serve *transfer between
  contexts* (drag file left→right), not just comparison. Browsers have no
  analogue because there is nothing to move between two web pages.
- macOS Finder tabs (2013, Mavericks), Windows File Explorer tabs (2022 —
  sixteen years after Dolphin).

### 12d. Terminal multiplexers — screen and tmux

- **GNU Screen** — Oliver Laumann, University of Erlangen, **1987**. Windows
  inside one terminal, plus the decisive feature: **detach and reattach**.
- **tmux** — 2007 (Nicholas Marriott), now dominant. Hierarchy:
  **server → sessions → windows → panes.** Windows behave like tabs; panes are
  splits within a window; sessions are named, detachable workspaces.
- **What multiplexers have that browsers do not:**
  1. **Detach/reattach with running state intact** — the session keeps
     executing while you are gone, and you can reattach *from a different
     machine*. The closest browser analogue (Chrome Sync's "tabs from other
     devices") only syncs URLs, not live state.
  2. **Named, scriptable, reproducible sessions** — `tmuxinator`/`tmux
     new-session -s work \; split-window ...` means a workspace is *code*. No
     browser lets you declare a workspace in a config file.
  3. **A three-level hierarchy as the default**, not an add-on.
  4. **Fully keyboard-driven with a prefix grammar** — no pointing at all.
- **Why browsers can't copy it directly:** a shell process is server-side and
  survives client detach; a web page is client-side state that dies with the
  renderer. This is *the* structural difference between the two families, and
  it is why browsers keep reinventing archives (OneTab, Arc auto-archive, saved
  groups) instead of detachment — an archive is the best you can do when you
  cannot keep the process running.

### 12e. Other adjacent lineages worth noting

- **Window managers** (i3, sway, yabai, Windows Virtual Desktops, macOS Spaces)
  — tiling + named workspaces at the OS level; Arc/Vivaldi workspaces are this
  idea pulled inside one app. Users who already run i3 tend to want the browser
  to be one window and manage everything at the WM level — the exact opposite of
  the Arc philosophy.
- **Email** — the paradigm every tab tool eventually reinvents: inbox +
  archive + search + snooze. Superhuman/Gmail's "snooze until" is exactly
  SigmaOS's tab snooze and Arc's auto-archive.
- **IDE/editor "workspaces"** (VS Code `.code-workspace`, JetBrains projects)
  — declarative, file-backed, version-controllable workspace definitions. Again,
  no browser has this.

---

## 13. Cross-cutting analysis

### 13a. What actually predicts adoption

Ranked by the evidence in this document:

1. **Effort before first payoff.** OneTab (1 click, 2M users) ≫ Session Buddy
   (taxonomy work, 900k) ≫ Tabs Outliner (a lifestyle, niche). Panorama
   demanded arrangement before any benefit and got 0.01%.
2. **Modality.** Anything that makes you *leave browsing to manage browsing*
   dies. Panorama was modal; Chrome groups are inline. Same feature, opposite
   outcomes.
3. **Default-on distribution.** Firefox tab groups: 18M users in a year, versus
   Panorama's 0.01% — the difference is on-by-default placement in the surface
   people already look at. Suspension is the most-used tab-management technology
   on earth precisely because nobody chose it.
4. **Automatic structure beats manual structure.** Tree Style Tab survives 19
   years because the tree builds itself from opener relationships. Vivaldi's
   auto-stacking by domain, Arc's auto-archive timer, and VS Code's preview-tab
   recycling all win for the same reason.
5. **Reversibility.** Chrome groups can be ungrouped for free; Panorama's
   removal *converted your groups to bookmarks*, which is what a lossy model
   looks like from the inside.

### 13b. The absorption ledger (2026)

| Paradigm | Origin (extension/niche) | Now native in |
|---|---|---|
| Vertical tabs | TST 2007, Vivaldi 2015 | Edge 2021, Firefox 2025, Chrome 2026 — **fully absorbed** |
| Tab groups | Vivaldi stacks 2015, Opera 2020 | Chrome 2020, Safari 2021, Firefox 2025 — **fully absorbed** |
| Tab search | Quick Tabs, Vivaldi F2 | Chrome 2020, Safari, Firefox `%` — **fully absorbed** |
| Suspension | The Great Suspender 2013 | Edge 2021, Chrome 2022, Firefox — **fully absorbed** |
| Split/tile | Vivaldi 2015 | Edge 2023, Chrome 2025, Arc/Zen — **partially** (2-pane only) |
| Saved sessions | Session Buddy 2011 | Chrome saved groups 2024, Firefox 2025, Safari 2021 — **partially** |
| Workspaces/spaces | Opera 2020, Arc 2022 | Vivaldi 2023, Edge 2023, Zen 2024 — **partially**; absent from Chrome/Firefox/Safari |
| **Tab trees** | TST 2007 | **nowhere native** — still the biggest unabsorbed idea |
| **Auto-archive / expiring tabs** | Arc 2022, SigmaOS | **nowhere native** — and Arc is dead |
| **Preview/ephemeral tabs** | VS Code 2016-ish | **no browser has it** |
| **Detach/reattach live sessions** | tmux 1987 | **structurally impossible in browsers today** |
| **Declarative workspace files** | tmuxinator, .code-workspace | **no browser has it** |

### 13c. User archetypes

- **The linear browser** (majority) — under 10 tabs, closes as they go. Uses
  nothing; benefits invisibly from suspension and default groups.
- **The project juggler** — 2–5 concurrent contexts. Tab groups and workspaces
  are aimed exactly here; this is where mainstream absorption happened.
- **The depth-first researcher** — branching exploration, 50–300 tabs. Tree
  Style Tab, Sidebery, Tabs Outliner. Small, loud, high willingness to
  configure, and the source of nearly every idea later absorbed.
- **The archivist / hoarder** — treats tabs as an unsorted inbox they intend to
  process. OneTab, Session Buddy, Pocket. Large population, low tool loyalty,
  because none of these actually get read.
- **The keyboard maximalist** — command palettes, tmux, i3, Vimium. Wants
  retrieval and scripting, not visual organization; systematically underserved
  by browsers, well served by IDEs and terminals.
- **The aesthetics-forward knowledge worker** — Arc's constituency: designers,
  PMs, founders. Buys into a whole philosophy, then gets stranded when the
  company pivots. Currently migrating to Zen.

### 13d. Where the frontier is in 2026

Three live fronts:

1. **Semantic/agentic** (Dia, AI grouping, Chrome/Edge AI sidebars). Bet: if an
   assistant can answer across your tabs, the container stops mattering.
   Unproven, well funded, and now Atlassian-owned.
2. **Fork-and-rebuild-the-GUI** (Zen, Floorp, Waterfox, and the whole
   post-Arc wave). Bet: Firefox's chrome is XUL/CSS and therefore *replaceable*
   by a community — which is exactly the thesis behind this project's
   Firefox-fork + Theia unified-tabs direction. Zen is the proof it works
   socially and technically.
3. **The unclaimed ideas.** The three highest-value unimplemented items,
   ranked by evidence in this document: **preview/ephemeral tabs** (kills
   accumulation at the source, proven for a decade in VS Code), **auto-expiry
   with a searchable archive** (proven in Arc, now orphaned), and
   **declarative/scriptable workspaces** (proven in tmux and VS Code, absent
   from every browser). A browser-IDE hybrid is the only product category
   positioned to take all three, because it already has the IDE-side precedent
   and the layout engine to host it.

---

## 14. Timeline (condensed)

```
1986  Norton Commander — dual-pane file management
1987  GNU Screen — detachable terminal sessions
1994  BookLink InternetWorks — tabbed windows
1996  Opera MDI
1998  NetCaptor/SimulBrowse — modern tabbed browsing
2000  Opera 4 tabs
2001  Mozilla 0.9.5 tabs
2003  Safari tabs
2006  IE 7 tabs — paradigm universal
2007  Tree Style Tab · tmux · Read It Later (→Pocket)
2008  Instapaper · Dolphin tabs+split
2010  TabCandy prototype (Aza Raskin)
2011  Firefox 4 Panorama · Safari Reading List · Session Buddy
2013  OneTab · The Great Suspender · Tabs Outliner · Finder tabs
2015  Vivaldi 1.0 — tab stacks + vertical tabs + tiling
2016  Firefox 45 REMOVES Panorama · VS Code preview tabs
2017  Firefox 57 WebExtensions (kills legacy tab add-ons) · Multi-Account
      Containers · Mozilla buys Pocket · Toby
2018  Workona · Facebook Container
2019  Sidebery · Vivaldi auto-stacking
2020  Opera 67 Workspaces (Feb) · Chrome 83 Tab Groups (May) · VS Code 1.46
      pinned tabs · Chrome Tab Search · Edge Collections
2021  Safari 15 Tab Groups · Edge vertical tabs (Mar) · Edge Sleeping Tabs ·
      The Great Suspender REMOVED as malware (Feb) · CHI 2021 tab study ·
      SigmaOS · Stack · Arc beta
2022  Arc wide release (Spaces, auto-archive) · Chrome Memory Saver ·
      Shared Tab Groups (iOS 16)
2023  Vivaldi 6.0 Workspaces · Edge Workspaces · Edge Split Screen ·
      Safari 17 Profiles
2024  Zen Browser launch · Chrome saved+synced tab groups · Firefox 133
      tab groups behind flag · Arc FEATURE FREEZE (Oct) · Tabs Outliner→MV3
2025  Firefox 136 vertical tabs (Mar) · Firefox 138/141 tab groups to all ·
      ARC SUNSET announced (May 27) · POCKET SHUTDOWN (Jul 8) ·
      Atlassian acquires The Browser Company $610M (Oct 21) · Zen stable
2026  Chrome 146 native vertical tabs (Mar–Apr) · Firefox 145 group peek ·
      Firefox reports 18M+ tab-group users · Dia 1.33 under Atlassian ·
      Edge re-promotes vertical tabs
```

---

## 15. Sources

Primary and load-bearing:

- Chang, Hahn, Kim, Coupland, Breneisen, Kim, Hwong, Kittur — *When the Tab
  Comes Due: Challenges in the Cost Structure of Browser Tab Usage*, CHI 2021.
  https://dl.acm.org/doi/10.1145/3411764.3445585 · https://joe.cat/CHI-browser-tabs/
- Mozilla Bugzilla 1222550 — *Product plan: remove tab groups (panorama)*.
  https://bugzilla.mozilla.org/show_bug.cgi?id=1222550
- Mozilla Discourse — *Discontinuation of Tab Groups (Panorama)*.
  https://discourse.mozilla.org/t/discontinuation-of-tab-groups-panorama/5145
- MozillaWiki — *Firefox/Projects/TabCandy/Work*.
  https://wiki.mozilla.org/Firefox/Projects/TabCandy/Work
- Softpedia — *Firefox 45 Will Drop Tab Groups (Panorama)* (Jan 2016).
  https://news.softpedia.com/news/firefox-45-will-drop-tab-groups-panorama-495775.shtml
- Mozilla Blog — *Firefox tab groups just got an upgrade* / *You got more with
  Firefox in 2025* (18M users figure).
  https://blog.mozilla.org/en/firefox/tab-groups-updates/ ·
  https://blog.mozilla.org/en/firefox/firefox-2025-updates/
- Mozilla Support — *Tab groups in Firefox*.
  https://support.mozilla.org/en-US/kb/tab-groups
- Mozilla Connect — *Sidebar and Vertical Tabs launch in release 136*.
  https://connect.mozilla.org/t5/firefox-labs/firefox-sidebar-and-vertical-tabs-try-them-out-in-nightly/m-p/63231
- 9to5Google — *Chrome tab groups will start rolling with version 83*.
  https://9to5google.com/2020/05/13/chrome-tab-groups/
- Slashdot — *Chrome 83 Released With ... Tab Groups* (19 May 2020).
  https://slashdot.org/story/20/05/19/185244/
- The Hacker News / 9to5Google / Android Police — *The Great Suspender contains
  malware*, removed and remotely disabled Feb 2021.
  https://thehackernews.com/2021/02/warning-hugely-popular-great-suspender.html ·
  https://9to5google.com/2021/02/04/the-great-suspender-extension-has-been-removed-from-chrome-web-store-for-containing-malware/
- gHacks — *Arc Browser has been discontinued, but the company's building a new
  browser Dia* (27 May 2025).
  https://www.ghacks.net/2025/05/27/arc-browser-has-been-discontinued-but-the-companys-building-a-new-browser-dia/
- Engadget / Mozilla Blog — *Mozilla is shutting down Pocket* (shutdown 8 Jul
  2025; data deleted 8 Oct 2025).
  https://www.engadget.com/apps/mozilla-is-shutting-down-its-read-it-later-app-pocket-183247576.html ·
  https://blog.mozilla.org/en/mozilla/building-whats-next/
- Vivaldi — *Tab Stacks*, *Tab Tiling*, *Workspaces*, *Vivaldi 6.0*, *Quick
  Commands guide*.
  https://help.vivaldi.com/desktop/tabs/tab-stacks/ ·
  https://help.vivaldi.com/desktop/tabs/tab-tiling/ ·
  https://vivaldi.com/blog/vivaldi-on-desktop-6-0/ ·
  https://vivaldi.com/blog/quick-commands-guide/
- gHacks / TechCrunch — *Opera 67 Stable launches with Workspaces* (Feb 2020).
  https://www.ghacks.net/2020/02/25/opera-67-stable-launches-with-workspaces-feature/ ·
  https://techcrunch.com/2020/02/25/operas-new-browser-can-tame-your-tabs/
- Apple Support — *Share Tab Groups and collaborate in Safari on Mac*.
  https://support.apple.com/guide/safari/share-tab-groups-collaborate-safari-mac-ibrw7f189b02/mac
- Mozilla Support — *Multi-Account Containers*.
  https://support.mozilla.org/en-US/kb/containers
- AMO listings (checked 2026-08-19): Tree Style Tab 136,347 users
  https://addons.mozilla.org/en-US/firefox/addon/tree-style-tab/ ; Sidebery
  50,926 users https://addons.mozilla.org/en-US/firefox/addon/sidebery/
- Chrome Web Store (checked 2026-08-19): OneTab ~2,000,000 users
  https://chromewebstore.google.com/detail/onetab/chphlpgkkbolifaimnlloiipkdnihall ;
  Session Buddy ~900,000 users
  https://chromewebstore.google.com/detail/session-buddy-tab-bookmar/edacconmaakjimmfgnblocblbcdcpbko ;
  Tabs Outliner
  https://chromewebstore.google.com/detail/tabs-outliner/eggkanocgddhmamlbiijnphhppkpkmkl
- VS Code release notes — *May 2020 (version 1.46)*, pinned tabs.
  https://code.visualstudio.com/updates/v1_46
- Fast Company — *This radical new browser will transform your relationship
  with tabs* (SigmaOS).
  https://www.fastcompany.com/90669133/radical-browser-tabs-sigmaos
- Adam Stiles — *Tabbed Browser DNA* (NetCaptor/SimulBrowse origin);
  OSnews *pt. VIII: Tabs*; Wikipedia *Tab (interface)*, *NetCaptor*,
  *GNU Screen*.
  https://ajstiles.wordpress.com/2005/02/11/tabbed_browser_/ ·
  https://www.osnews.com/story/20191/pt-viii-tabs/
- BrainTool — *The Best Tabs Outliner Alternative* (MV3 status of Tabs
  Outliner). https://braintool.org/2024/07/01/The-best-Tabs-Outliner-alternative.html
- Local source read for §12b: `sources/theia/packages/core/src/browser/shell/`
  (`application-shell.ts` `Area` union, `theia-dock-panel.ts`,
  `shell-layout-restorer.ts`, `side-panel-handler.ts`).

Secondary/aggregator sources (Zen feature details, Dia 2026 status, Chrome 146
vertical-tabs rollout dates, Toby/Workona 2026 pricing tiers, tab-count
averages) are SEO-heavy comparison blogs — SupaSidebar, superchargebrowser,
tabgroupvault, leap-tabs, efficient.app, aboutchromebooks. Treat their exact
version numbers and percentages as indicative, not authoritative; the
directional claims are corroborated across several of them.
