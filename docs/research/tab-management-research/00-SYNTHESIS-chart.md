# 00 — SYNTHESIS: Tab/Navigation Paradigm Chart

Synthesis of `01-paradigm-history-taxonomy.md` (01), `02-academic-studies.md` (02),
`03-user-customization-preferences.md` (03), and `04-everything-is-a-url.md` (04).
Compiled 2026-08-19.

Project context: the tab/navigation system for a unified browser + IDE (Firefox fork +
Eclipse Theia) in which every open thing — local file, folder, terminal, remote path,
web page — is a URI in one tab system. Target user: people who move through large
volumes of information and need to organize it.

Citation convention: `(01)`–`(04)` mark the source document for each claim. Figures that
could not be traced to a primary source are marked **UNVERIFIED** and must not be cited
downstream.

---

## 1. THE CHART

Thirteen paradigm families. Split into two tables over the same row keys so both stay
readable; row order is identical in Table A and Table B.

### Table A — Paradigm, exemplars, and flow path of use

Flow notation: the four-step lifecycle **open → find → stash → return**, spelled out per
paradigm.

| # | Paradigm family | Exemplar products | Flow path of use |
|---|---|---|---|
| 1 | **Flat tab strip** (horizontal) | NetCaptor 1998, Opera 4, Firefox, Chrome, Safari, IE7 (01) | **Open:** click link or Ctrl+T; tab appends to the right. **Find:** scan favicons/titles left-to-right, or Ctrl+Tab cycle. **Stash:** no stash — you either keep it open or close it. **Return:** click the tab; it holds live state (scroll, form input, auth session) (01). Degrades once tabs shrink below title legibility (01). |
| 2 | **Vertical tabs** (flat sidebar) | Vivaldi 2015, Edge 2021, Arc 2023, Firefox 136 (Mar 2025), Zen 2024, Chrome 145–146 (2026) (01)(03) | **Open:** same as flat strip; tab appends downward. **Find:** read full titles in a scrollable list; ~40 tabs stay legible vs ~10 horizontally (01). **Stash:** none native; scroll is the only overflow answer. **Return:** click the row. Trades scarce horizontal width for abundant vertical height — a win on wide monitors, a loss when windows are snapped side by side (03). |
| 3 | **Tree tabs** (hierarchical, auto-derived) | Tree Style Tab 2007, Sidebery ~2019, Tabs Outliner 2013, Sidewise (dead), Tree Tabs (01)(03) | **Open:** link opens as a *child* of its opener — structure is derived from navigation, no user action (01). **Find:** collapse sibling subtrees; 20 tabs from one article read as one node (01). **Stash:** collapse the subtree, or (Sidebery) auto-unload the collapsed branch (03). **Return:** expand the node. **Prune:** close-subtree closes the whole branch at once (01). |
| 4 | **Tab groups** (inline label above the tab) | Chrome 83 (May 2020), Safari 15 (2021), Firefox 133–141 (2024–25), Vivaldi Tab Stacks 2015, Edge (01) | **Open:** normal; then right-click → add to group, or drag one tab onto another and pause (Firefox) (01). **Find:** collapse the group to its coloured pill; expand the one you need. **Stash:** save-and-close the group — Chrome Saved Tab Groups (2024), Firefox save-and-close (2025), Safari iCloud-synced groups (2021) (01). **Return:** click the pill or restore the saved group; ungrouping is free and lossless (01). |
| 5 | **Workspaces / spaces** (container swaps the strip) | Arc Spaces 2022, Opera Workspaces Feb 2020, Vivaldi 6.0 (2023), Zen 2024, Edge Workspaces 2023, SigmaOS, Workona (01)(03) | **Open:** enter a workspace first, then open into it. **Find:** switch workspace → the whole strip is replaced by that context's tabs (01). **Stash:** leave a workspace; its tabs persist unseen. **Return:** switch back; Arc/Zen additionally bind the workspace to a cookie container or profile so identity switches with context (01). Groups are additive; workspaces are exclusive (01). |
| 6 | **Session dump** (OneTab-style bulk unload) | OneTab 2013, Session Buddy ~2011, Tab Session Manager, Tab Stash, Sessionic (01)(03) | **Open:** irrelevant — the paradigm activates at overflow. **Find:** the stash *is* the finding surface: a static list page or saved-session table, searchable across archives (01). **Stash:** one click converts all open tabs to a list and closes them; immediate RAM and strip relief (01)(03). **Return:** restore one item or the whole list. Reliability is the fault line — list loss on crash/update is the recurring complaint (03). |
| 7 | **Read-later queue** (item leaves the browser) | Pocket 2007–2025 (dead), Instapaper 2008, Safari Reading List 2011, Readwise Reader 2022, Toby 2017, Raindrop.io (01) | **Open:** normal browsing. **Find:** n/a at capture time. **Stash:** one click sends the URL to a *different application* with its own inbox semantics; the tab closes (01). **Return:** open the queue app later — and mostly never do; most saved items are never read (01). |
| 8 | **Bookmarks / folder hierarchy** | Every browser since Mosaic; Windows Quick Access, KDE Places panel (01)(04) | **Open:** normal. **Find:** navigate a folder tree, or the always-visible bookmarks bar. **Stash:** Ctrl+D, then decide a folder — the decision itself is the bottleneck (02). **Return:** measured failure: only 16% of bookmarked targets were retrieved via the bookmark facility, and only 4% via the folder hierarchy; the rest came from the always-visible bar (02). Preserves a URL only — no scroll position, no unsaved edits, no session state (04). |
| 9 | **Search / switcher** (retrieval-first) | Chrome Tab Search (Ctrl+Shift+A, 2020), Firefox `%` awesomebar switch-to-tab (2008), Vivaldi Quick Commands F2, Safari ⌘⇧\, VS Code ⌘P, Emacs `C-x b` / Ibuffer (01)(04) | **Open:** normal. **Find:** type a fragment of the title; fuzzy-match across open tabs, recently closed tabs, and other devices (01). **Stash:** none — the paradigm replaces stashing with cheap retrieval. **Return:** Enter switches to the existing tab rather than opening a duplicate. Zero setup, immediate payoff — the best effort-to-payoff ratio in the taxonomy (01). |
| 10 | **Suspension / auto-archive** (resource and attention relief) | The Great Suspender 2013 (removed as malware Feb 2021), Edge Sleeping Tabs 2021, Chrome Memory Saver 2022, Firefox `about:unloads`, Vivaldi Tab Hibernation, Arc auto-archive 2022, VS Code `workbench.editor.limit.value` (01)(03) | **Open:** normal. **Find:** unchanged — suspended tabs still occupy their slot; archived tabs move to a searchable archive (01). **Stash:** happens *to* you on a timer: suspend (keep the slot, drop the RAM) or auto-archive (drop the slot after 12h–30d, default 12h in Arc) (01). **Return:** click to reload the suspended tab, or search the archive. Suspension is the most-used tab-management technology on earth precisely because nobody chooses it (01). |
| 11 | **Preview / ephemeral tabs** | VS Code preview tabs (~2016), Theia `WidgetOpenHandler`, `workbench.editor.enablePreview` (01)(04) | **Open:** single click / go-to-definition / search-result click opens into *one reusable* italic slot; the next preview replaces it, and it is not added to the Open Editors list (04). **Find:** unchanged — the strip never accumulated the noise in the first place. **Stash:** n/a; the transient tab evaporates on the next preview. **Return:** double-click, edit, or explicit pin promotes it to a permanent tab (04). Solves accumulation at the source by separating "looking" from "keeping" (01)(04). |
| 12 | **Detachable sessions** (tmux-style) | GNU Screen 1987, tmux 2007, tmuxinator, VS Code `.code-workspace`, Theia `ShellLayoutRestorer` (01)(04) | **Open:** attach to a named session; open windows/panes inside it. **Find:** the session's own three-level hierarchy (server → session → window → pane), fully keyboard-driven via a prefix grammar (01). **Stash:** detach — the session keeps *executing* while you are gone. **Return:** reattach, possibly from a different machine, with live state intact (01). Declarative variant: the workspace is a config file, so a layout is reproducible code (01). |
| 13 | **Spatial / canvas** | Firefox Panorama / TabCandy (2010–2016, removed), Tiled Tab Groups add-on, Arc Easels, Kosmik, Heptabase, Stack Browser, Acme columns (01)(04) | **Open:** normal, then enter a separate zoomed-out canvas mode. **Find:** recognize a live thumbnail by its position on a 2D plane — spatial rather than text recall (01). **Stash:** drag the thumbnail into a named rectangle; returning to browsing shows only the active group's tabs. **Return:** re-enter the canvas, zoom into a group. Acme's variant removes the switcher entirely: resizable columns, look at the screen (04). |

### Table B — Adoption evidence, who uses it, status

| # | Paradigm family | Popularity / adoption evidence | Who uses it (segments from 02) | Status |
|---|---|---|---|---|
| 1 | Flat tab strip | Universal since IE 7 (2006) (01). Opening a page in a new window/tab rose from <1% of navigation actions in the mid-1990s to >10% by 2005–06 (02). Median 8 open tabs, Q1=5, Q3=12 (Chang et al. 2021, N=103) (01)(02). Only 6/15 Firefox users (40%) regularly used tabs in 2005–06 data (02). | Everyone. The **linear browser** majority — under ~10 tabs, closes as they go (01). Corresponds to *minimalists* and *cleaners* (02). | **Native everywhere.** Won because it is linear, horizontal, and zero-configuration — every later paradigm has had to beat "does nothing, costs nothing" (01). |
| 2 | Vertical tabs | Shipped by every major vendor in sequence: Vivaldi 2015 → Edge 2021 → Arc 2023 → Firefox 136 Mar 2025 → Chrome 2026 (01)(03). Requested on Mozilla Connect for "more than two years" before Firefox shipped it (03). Edge re-promoted it in April 2026 in response to Chrome shipping (01). Reviewers call Edge's implementation "divisive" (03). | Users above the ~20-tab threshold, and wide/ultrawide-monitor users (03). *Hoarders* of either organisation style; also *project jugglers*. Small-screen and low-count users stay horizontal (03). | **Native everywhere as of 2026**, but flat only. Vendors took the cheap 80% (legible titles, room for ~40 tabs) and left hierarchy to extensions (01). |
| 3 | Tree tabs | Tree Style Tab: **136,347 AMO users**, 4.5★/2,266 reviews, maintained continuously since 2007 (01). Sidebery: **50,926 users**, 4.7★/1,067 reviews, Mozilla Recommended (01). Reviewers report 600–1000+ open tabs navigable via nesting, and cite TST as the reason they stay on Firefox (03). ~0.05% of the Firefox base — the same band Panorama died in, surviving because its author, not Mozilla, pays for it (01). | The **depth-first researcher**: branching exploration, 50–300 tabs (01). Overlaps *tab power users* (8/21 = 38%, bimodal by logged creation rate) (02) and *organised hoarders* (02). High willingness to configure; explicitly resists auto-closing and resents auto-enabled new defaults that disturb hand-built trees (03). | **Extension-only — the biggest unabsorbed idea in the taxonomy** (01). No browser ships it natively; Chrome's implementation (Sidewise) died with MV3. |
| 4 | Tab groups | Chrome 83, May 2020, default-on to a ~65% market-share browser (01). Mozilla reports **over 18 million people** used Firefox tab groups in the first year, against Panorama's 0.01% — same company, same nominal feature, three orders of magnitude apart (01). The single most-upvoted idea on Mozilla Connect since launch: ~4,500 votes, 64+ pages of feedback (03). Median tab count among Firefox grouping users is **~25** (02). | **Project jugglers** — 2–5 concurrent contexts — plus mainstream users generally (01). Maps to *creation-time filers* (23%) and *end-of-session filers* (7%) from Abrams et al. (02); *frequent filers* in Whittaker & Sidner terms (02). | **Native everywhere.** Fully absorbed; the pill-in-strip form is now the industry meaning of "tab group" (01). |
| 5 | Workspaces / spaces | Opera Feb 2020, Vivaldi 6.0 2023, Edge 2023, Zen 2024 (01). Arc: large designer/PM mindshare but never a measurable slice of global browser share; feature freeze Oct 2024, sunset announced 27 May 2025, The Browser Company acquired by Atlassian for **$610M**, closed 21 Oct 2025 (01). Zen: ~42k GitHub stars, took over Arc's audience within ~3 months (01)(03). Toby: 300,000+ users across 144 countries (03). Workona free tier capped at 5 workspaces (01). | The **aesthetics-forward knowledge worker** (designers, PMs, founders) and teams (01). Also *cross-tool consistent organisers* — the minority who apply one organising philosophy everywhere (Boardman & Sasse 2004) (02). Corresponds to *organised hoarders* and *organised minimalists* alike (02). | **Partially native** — Vivaldi, Opera, Edge, Zen; **absent from Chrome, Firefox, Safari** (01). Arc is dead; the surviving lesson is that the demand was for the *organisational features*, not the departure from conventional browser UX (03). |
| 6 | Session dump | OneTab: **~2,000,000 users**, 4.4★/14.6k ratings — the most-installed tab tool ever (01)(03). Session Buddy: **~900,000 users**, 4.7★/25.1k ratings (01)(03). The 2× gap is the strongest available evidence that **effort-before-first-payoff is the binding adoption constraint**: OneTab has one interaction and zero taxonomy work; Session Buddy asks you to think about sessions (01). | The **archivist / hoarder** — treats tabs as an unsorted inbox they intend to process (01). Maps to *no filers* (26%) and *sporadic / spring-cleaning filers* (~44%) (02), *pilers* (Malone 1983; Kalman & Ravid 2015 found most users are pilers) (02), and the 30% who self-identify as having "a tab hoarding problem" (02). Low tool loyalty, because none of these lists actually get read (01). | **Partially absorbed** — Chrome Saved Tab Groups 2024, Firefox save-and-close 2025, Safari synced groups 2021 cover the common case. Extensions survive on *search across archives* and *cross-device backup*, which natives still do poorly (01). |
| 7 | Read-later queue | Safari Reading List (2011) is probably the most-used read-later feature in the world by default distribution alone (01). **Pocket is dead**: acquired by Mozilla 2017, a button in the default toolbar for eight years, shut down 8 Jul 2025, all data deleted 8 Oct 2025 (01). Instapaper survives by being small; Readwise Reader absorbed the Pocket refugees (01). Toby's free tier is now capped at 60 saved tabs (01)(03). | Readers and knowledge workers rather than multi-project jugglers (01). Maps to Chang et al.'s **O4 Sunk Costs and Aspirations, 16.6% of labelled tabs** — hoping to process more than is realistically possible (02). | **Dying as a standalone category.** Saving is easy and free; reading the queue is the hard part, and most saved items are never opened (01). Native equivalents (Reading List) survive on zero-cost distribution. |
| 8 | Bookmarks / hierarchy | Universal since the 1990s. Collections grow near-perfectly linearly: N = 7.45 + 0.18·T, R=.996 — one new bookmark every 5.5 days, with essentially no pruning (02). Mean maximum collection 184 bookmarks, 18.1 folders (Cockburn & McKenzie 2001) (02). Bookmarks account for ~2% of navigation actions (02). **Retrieval failure measured: 16% of bookmarked targets retrieved via bookmarks; 4% via the folder hierarchy** (Bergman et al. 2021) (02). Beyond ~35 bookmarks — the drop-down visibility threshold — folder count grows linearly with archive size (02). | Nominally everyone; effectively almost nobody as a *retrieval* path. Users' "black hole" fear of filing is **empirically correct** (02). Jones et al. found people reach for email-to-self, printing, and pasting URLs into documents instead (02). | **Native everywhere and functionally failed.** Cockburn's 2001 design implications — scale, active pruning assistance, special-case the heavily-visited handful — are still unaddressed 25 years later (02). |
| 9 | Search / switcher | Chrome Tab Search shipped Chrome 87/88 (2020–21), on by default (01). Firefox's `%` switch-to-tab dates to the 2008 Awesomebar and most users have never noticed it (01). Coverage rationale: "past 20 open tabs, shortcuts stop being useful and a tab search panel becomes faster" (03). Search is a de facto bookmarking system — **as many as 40% of all queries are re-finding queries** (Teevan et al. 2007) (02). Chang et al.: 28% often struggle to *find* the tab they need (02). | The **keyboard maximalist** — command palettes, tmux, i3, Vimium (01) — plus *everyone else by accident*, since Chrome ships it on by default (01). Also the fallback mode any user enters above ~20 tabs; this is a **threshold effect, not a stable persona** (03). | **Native everywhere. The best effort-to-payoff ratio in the taxonomy** (01). Directly attacks re-finding cost — O3, 17.7% of kept tabs (02). |
| 10 | Suspension / auto-archive | The Great Suspender peaked at **2M+ installs**, was sold in 2020, shipped remote-code-execution and ad-fraud tracking, and was removed from the store and remotely disabled by Google in Feb 2021 (01)(03). Edge Sleeping Tabs reports **~26% average memory saving** per sleeping tab (01). Chrome Memory Saver on by default since Chrome 108/110 (01). Arc's auto-archive defaults to 12 hours, adjustable to 30 days (01). | Everyone, invisibly (01). Auto-archive specifically serves *hoarders of both organisation styles* by removing the decision. Never-auto-close is a distinct and vocal counter-segment: TST reviewers keeping 600–1000 tabs actively resist any tool that closes for them (03). | **Suspension: fully native everywhere** — and the most-used tab-management technology on earth (01). **Auto-archive: unclaimed** — proven in Arc, now orphaned by Arc's death, native nowhere (01). Trust is a first-class constraint: anything that runs code against every tab is a standing trust bet the market has burned twice (03). |
| 11 | Preview / ephemeral tabs | Shipped in VS Code since ~2016 and in Theia; governed by `workbench.editor.enablePreview` (01)(04). **No browser has it** — a link click or bookmark click always creates a durable tab (04). Windows File Explorer added tabs in 2022 and explicitly did *not* add a preview/pinned distinction; every opened folder is a full permanent tab (04). | Would serve every segment, because it acts before segmentation does: it prevents accumulation rather than managing it. Highest value to *depth-first researchers* and *tab power users*, whose branching generates the transient opens (01)(02). | **Unclaimed opportunity — the highest-value unimplemented item in the taxonomy.** Proven for a decade in VS Code, absent from every browser (01)(04). |
| 12 | Detachable sessions | GNU Screen 1987, tmux 2007 and dominant (01). Structurally impossible in browsers today: a shell process is server-side and survives client detach; a web page is client-side state that dies with the renderer (01). This is why browsers keep reinventing *archives* (OneTab, Arc auto-archive, saved groups) instead of detachment — an archive is the best available substitute (01). Declarative workspace files (`tmuxinator`, `.code-workspace`) exist in terminals and IDEs and in **no browser** (01). Theia's `ShellLayoutRestorer` serializes the entire layout tree — splits, tab order, active widget, area sizes (01). | The **keyboard maximalist**, systematically underserved by browsers and well served by IDEs and terminals (01). Also anyone who works across machines; **cross-device tab state has no empirical literature at all** (02). | **Dead end for pure browsers; a live advantage for a browser-IDE hybrid.** Theia already has the layout-tree serializer and `secondaryWindow` detach; browsers can move a tab to a new window but cannot move a *pane* out of a tiled layout and back (01). |
| 13 | Spatial / canvas | Firefox Panorama: **about 0.01% of users** engaged with it; deprecated in Firefox 44, removed in Firefox 45 (March 2016) for near-zero usage plus entanglement with the tab implementation that blocked Electrolysis (01). On removal, each group's tabs were converted to a bookmarks folder — a lossy migration (01). The spatial paradigm found product-market fit **as a research surface, not as a tab strip** (Kosmik, Heptabase, Arc Easels) (01). | Researchers, students, and spatial thinkers running parallel projects — a tiny, loud group (01). Overlaps *organised hoarders* (02). | **Dead as a tab paradigm; alive as a note-taking surface** (01). The canonical cautionary tale: it failed because it was **modal** (you leave browsing to manage browsing), **manual** (arrangement work up front with no immediate payoff), and **invisible** (nothing in the default UI announced it) (01). |

**Cross-chart note.** The paradigm was rarely the problem. Firefox shipped the same nominal
feature twice — Panorama at 0.01% adoption and Tab Groups at 18M users in a year — and the
entire difference was modality, discoverability, and default-on placement (01). Treat every
row above as a *mechanism* whose adoption is determined by delivery, not by concept.

---

## 2. User segmentation summary

### 2.1 The two orthogonal axes

The single most important structural finding for design: **accumulation tendency and
organisation ability are independent axes** (Vitale, Janzen & McGrenere 2018, established
explicitly; Boardman & Sasse 2004, implicitly) (02). All four quadrants are populated.

|  | **Organised** | **Messy** |
|---|---|---|
| **Hoarding** (keeps everything, rarely deletes) | Tree-tab and Tabs Outliner users; builds and maintains structure at 300–1000 tabs (01)(03) | The default overloaded state: OneTab and Session Buddy users; the 30% who self-report "a tab hoarding problem" (01)(02) |
| **Minimalism** (avoids storing; runs clean-up passes) | Closes as they go, keeps a small deliberate set of pinned references (02) | Under 10 tabs by attrition rather than by system; the "linear browser" majority (01) |

Sample split in Vitale et al. (N=23): **11 hoarders (48%), 6 minimalists (26%), 3 even mix,
3 unclassifiable** (02). Designing "a tidier UI for hoarders" is a category error — the two
axes must be handled by separate mechanisms (02).

### 2.2 The consolidated segment model

Six segments, each traceable to observed behaviour rather than to speculation.

1. **Linear browser** (majority) — under ~10 tabs, closes as they go, uses no tab tooling and
   benefits invisibly from suspension and default groups (01). Empirical anchor: 7 of 21
   participants had a median of only 1–2 open tabs (Dubroy & Balakrishnan 2010) (02).
2. **Project juggler** — 2–5 concurrent contexts; the target of mainstream tab groups and
   workspaces, and where absorption actually happened (01). Anchor: the user-invented
   two-level hierarchy "tabs for related subtasks, windows for parallel tasks"
   (Weinreich et al. 2008) (02).
3. **Depth-first researcher** — branching exploration, 50–300 tabs, tree tools, high
   willingness to configure; the source of nearly every idea later absorbed by vendors (01).
   Anchor: *tab power users* are an empirically bimodal 8/21 (38%) by logged tab-creation
   rate — 0.14 tabs per navigation vs 0.04 for the lower mode — and all of them, and none of
   the others, deliberately opened links in background tabs (02).
4. **Archivist / hoarder** — treats the tab strip as an unsorted inbox they intend to process
   (01). Anchors: 55% agree they "can't let go of their tabs"; 30% self-identify as tab
   hoarders; 59% agree they currently have tabs that should be closed while only 19% blame
   laziness (02).
5. **Keyboard maximalist** — wants retrieval and scripting rather than visual organisation;
   underserved by browsers, well served by IDEs and terminals (01).
6. **Aesthetics-forward knowledge worker** — designers, PMs, founders; buys a whole
   philosophy and gets stranded when the vendor pivots. Arc's constituency, now migrating to
   Zen (01)(03).

### 2.3 Inherited filing typologies that transfer

- **Malone 1983 — filers vs pilers.** Categorisation *cost*, not discipline, explains why
  people do not file; hierarchies have overlapping ambiguous categories, so diligent filing
  does not guarantee retrieval. Piles are visible and therefore remind; files are not (02).
- **Whittaker & Sidner 1996 — frequent filers / spring cleaners / no filers.** The canonical
  trichotomy, from email (02).
- **Abrams, Baecker & Chignell 1998 — bookmarks, N=299:** no filers **26%**, creation-time
  filers **23%**, end-of-session filers **7%**, sporadic/spring-cleaning filers **~44%** (02).
  Nobody has replicated these proportions on tabs — a named gap in the literature (02).
- **Kalman & Ravid 2015** — telemetry over thousands of users for 8 months: **most users are
  pilers**, and the trichotomy is better modelled as a continuum of filing intensity than as
  three discrete types (02).
- **Gwizdka 2004 — cleaners vs keepers**, specifically over messages referencing future
  tasks; the difference is explained by experience and by a **requirement for flexibility of
  closure** — keepers deliberately preserve open-endedness (02). This is the closest email
  analogue to tab hoarding.
- **Ma et al. 2023 — problem-focused vs emotion-focused copers.** The only tab-domain
  segmentation validated against an outcome: **only problem-focused strategies actually
  reduced clutter**; emotion-focused strategies perpetuated it (02).
- **Boardman & Sasse 2004** — strategies are tool-specific in the general case; only a
  minority apply a consistent philosophy across files, email and bookmarks. You cannot infer
  someone's tab behaviour from their file behaviour (02).

### 2.4 Behavioural thresholds (these matter more than personas)

- **~8 tabs** — the median count at which users report management problems starts (Q1=5,
  Q3=12); **67% reach it at least once a week**, 17% daily (Chang et al. 2021, N=103) (02).
- **~20 tabs** — the point at which visual/spatial scanning of the strip stops working and
  users switch to keyboard search; explicitly framed as a within-session threshold effect on
  the same person, not a user-type split (03). Power-user surveys routinely find 40–50% of
  respondents above 20 tabs (01, indicative).
- **~25 tabs** — median tab count among Firefox users of the tab-grouping feature; the tail
  reaches into the thousands (Mozilla telemetry 2025, self-selected population) (02).
- **~35 bookmarks** — the drop-down visibility threshold beyond which folder count begins
  growing linearly with archive size (Abrams et al. 1998) (02).
- **>60 tab switches per hour** among knowledge workers, with load dominated by *inactive*
  tabs rather than tabs in play (Rutishauser & Fritz 2026) (02).
- **Median 8 open tabs, range <3 to ~400** in Chang et al.; Dubroy's logged medians were ≤6
  with one participant at 17 and a maximum of 42 (01)(02). **There is no published,
  methodologically-sound, large-N distribution of open-tab counts from any vendor** (02).

### 2.5 Why users keep tabs (the demand the design must satisfy)

Chang et al. 2021, share of 633 labelled tabs (a tab may carry more than one) (01)(02):

| Code | Pressure to keep | Share | Right storage semantics (02) |
|---|---|---|---|
| O2 | Revisiting references | **41.7%** | Pin; fast access; diminishing returns past a small number |
| O1 | Reminders and resumption | **37.3%** | Must stay visible and resurface; must not be filed away |
| O5 | External mental model | **20.1%** | Groups, workspaces, spatial arrangement |
| O3 | Avoid costly re-finding | **17.7%** | Cheap archive plus strong search; visibility not required |
| O4 | Sunk costs and aspirations | **16.6%** | Should decay; low salience; sunk-cost aware |
| O6 | Uncertain relevance | **5.7%** | Pile-like short-term buffer with automatic ageing |

Opposing pressures to close: limited attention, limited screen real estate, limited computing
power, and social/self pressure to appear organised — including reported **shame**, with
participants cleaning up tabs before screen-sharing (01)(02).

---

## 3. Customization dimensions matrix

| Dimension | Options | Which segment picks which | Evidence |
|---|---|---|---|
| **Tab bar position** | Horizontal strip; vertical sidebar; auto-hide/expand-on-hover vertical | Horizontal: linear browsers, small/narrow screens, low counts. Vertical: depth-first researchers, archivists, anyone above ~20 tabs, wide/ultrawide monitors | Every major vendor shipped vertical in sequence 2015→2026; Firefox users requested it on Connect for 2+ years; Edge's version is called "divisive" — a tradeoff users want the *choice* to make, not a universal win (03) |
| **Grouping model** | Manual folders/workspaces; automatic/AI grouping; flat list + search; tree (implicit, derived from navigation) | Manual: project jugglers, cross-tool consistent organisers, creation-time filers. Auto/AI: everyone, as an *additive* layer. Flat+search: keyboard maximalists and anyone past ~20 tabs. Tree: depth-first researchers | Chrome's "Organize similar tabs" (2024, on-device by Chrome 146/2026) is explicitly additive — users still rename/recolor after the AI proposes (03). Raindrop.io publicly rejects full auto-categorization in favour of user placement as ground truth that AI learns from (03). Tabs.do/Skeema showed grouping-by-task lowered tab counts and stress **without asking users to be tidier** (02) |
| **Hierarchy depth** | Flat; one level (groups); n-level tree; workspace → group → tab → split | Flat: majority. One level: project jugglers. n-level: depth-first researchers and organised hoarders | Sidebery exposes "simple flat list" vs full tree with a configurable tree limit — density and depth are tuned together, not as separate knobs, in tools built for high counts (03). Vivaldi is the only browser composing workspace → stack → tile (01) |
| **Persistence** | Local session save/restore; cloud sync; automatic durable state; nothing | Local: archivists, privacy-sensitive users, anyone burned by data loss. Cloud: teams, multi-device users, aesthetics-forward workers | Users separate "survive a crash" (local, must be bulletproof) from "follow me to another device" (cloud). Tools are punished hard for failing on *either* axis — Session Buddy for silent data loss and undisclosed telemetry, Workona for unreliable sync at scale (03). IDEs already persist "what's open" as durable state by default with no user gesture; browsers make it opt-in (04) |
| **Auto-cleanup policy** | Never auto-close; suspend/hibernate in place; auto-archive on a timer; hard cap with LRU eviction; bulk unload to a list | Never-close: tree-tab hoarders at 600–1000 tabs, who actively resist any tool that closes for them. Suspend: everyone, invisibly. Auto-archive: hoarders willing to delegate the decision. Bulk unload: archivists at panic point | TST reviewers explicitly praise no forced pruning (03). Edge Sleeping Tabs ~26% memory saving per tab; Chrome Memory Saver default-on (01). Arc's 12h default archive (01). VS Code's `workbench.editor.limit.value` is opt-in automatic tab bankruptcy per editor group (01) |
| **Retrieval mode** | Visual/spatial scanning; colour and position; keyboard fuzzy search; command palette | Threshold-driven within a single person: spatial below ~20 tabs, keyboard above it | Explicitly framed as a count threshold rather than a persona split (03). Sidebery offers per-domain/URL auto-colouring for the spatial mode (03); Vivaldi Quick Commands (F2) and VS Code ⌘P are the palette form (01) |
| **Identity binding** | None; per-tab container; per-workspace container; full profile | Anyone separating work/personal/client contexts; privacy-conscious users | Firefox Multi-Account Containers (2017); Arc binds spaces to profiles; Zen binds workspaces to containers; Sidebery binds sidebar panels to containers; Safari 17 nests tab groups under profiles. Everyone independently concluded "which project am I in" and "which identity am I" are usually the same question (01) |
| **Density / visual weight** | Compact vs comfortable rows; corner rounding; thumbnails vs text; icon-only pinned tabs | Power users with high counts tune density down; low-count users leave defaults | Vivaldi exposes tab appearance and corner rounding under Settings → Themes, with a dedicated "Customise" right-click menu on tabs — a real, requested axis, not a hidden preference (03) |
| **Layout / splitting** | Single pane; 2-pane split; n-pane tree; recursive dock grid | Comparison work — translators, developers reading docs against code, analysts (01). Also file-transfer-between-contexts, the orthodox file-manager use case | Every major browser now has 2-pane split; only Vivaldi and Zen do n-pane (01). VS Code Editor Groups make a recursive split grid the *default* rather than an add-on, each group with its own tab bar and active editor (01). Dolphin's F3 gives each tab two independent panes with independent navigation (01) |
| **Panel arrangement** | Fixed chrome; movable/detachable panels; fully re-arrangeable shell | Keyboard maximalists and heavy customizers | Dolphin's customization model is panel-level rearrangement — every panel can be detached, moved, or removed — not just content-level settings (03). Theia's `ApplicationShell` already partitions into main/top/left/right/bottom/secondaryWindow with drag-anywhere docking (01) |
| **Trust surface** | Native; first-party extension; third-party extension with tab-content access | Everyone, but decisive for anything that reads or runs code against tab contents | "Does this run code against my open tabs and can I trust it not to phone home or get sold" is functionally part of feature evaluation. The Great Suspender was sold and weaponized; successor forks repeated the pattern; Session Buddy was reported for unannounced telemetry — a plausible driver for vendors folding hibernation into the browser core (03) |

---

## 4. Implications for the unified everything-is-a-URL tab system

Fifteen implications, drawn strictly from the evidence above.

1. **Adopt preview/ephemeral tabs as the default open mode for every scheme, not just
   `file:`.** Single-click, link-follow, and search-result activation open into one reusable
   slot; double-click, edit, or explicit pin promotes to a durable tab. This is the
   highest-value unimplemented idea in the taxonomy, proven for a decade in VS Code, present
   in no browser, and it prevents accumulation instead of managing it (01)(04). Preview state
   must be part of the tab model itself — a per-tab `preview | pinned | durable` state — so
   web pages, terminals, and folders all inherit it (04).
2. **Auto-archive on a timer to a full-text-searchable store, and make the archive the
   retrieval surface.** Arc proved the mechanic and is now dead, leaving it unclaimed (01).
   The archive must be searchable by content, not only by title, because re-finding cost
   (O3, 17.7% of kept tabs) and the 40% re-finding query rate are what actually drive
   hoarding (02).
3. **Never let anything become invisible without becoming searchable in the same action.**
   The "black hole" fear is empirically justified: 16% of bookmarked targets are retrieved
   via bookmarks and only 4% via the folder hierarchy (02). Any stash operation that lacks a
   strong search path reproduces the measured failure of bookmarks.
4. **Do not ship a folder hierarchy as the primary organising mechanism.** Bookmark archives
   grow linearly and are never pruned (one every 5.5 days, R=.996), account for ~2% of
   navigation, and fail at retrieval (02). Categorisation cost — deciding *where* something
   goes — is the bottleneck, not storage (02).
5. **Derive structure automatically from opener relationships, and let the user override it.**
   The tree that builds itself is why Tree Style Tab has survived 19 years while manual
   arrangement paradigms died (01). At least 60% of a person's tabs are related to other tabs
   from the same task, which is the empirical basis for automated grouping (02).
6. **Make automation strictly additive to manual structure.** Chrome's AI grouping proposes
   and the user renames/recolours; Raindrop keeps user placement as ground truth that AI
   learns from; TST users resent auto-enabled defaults that disturb hand-built trees (03).
   Never let an automatic pass silently reorganise a structure the user built.
7. **Keep every management action inline; never make the user leave working to manage
   working.** Modality killed Panorama at 0.01% adoption; inline pills got Firefox 18M users
   for the same nominal feature (01). A zoomed-out canvas may exist as an optional view, but
   it must not be the only path to grouping.
8. **Deliver payoff on the first interaction, before any taxonomy work.** OneTab's 2M
   installs against Session Buddy's 900k is the cleanest measurement of effort-before-payoff
   as the binding adoption constraint (01)(03). Provide a one-gesture "clear the strip, keep
   everything, searchable" action from day one.
9. **Exploit detachable, serializable layouts — this is the Theia advantage and the one
   capability browsers structurally cannot copy.** Theia's `ShellLayoutRestorer` already
   serializes the whole layout tree (splits, tab order, active widget, area sizes), and
   `secondaryWindow` detaches a widget into its own OS window and back; browsers can move a
   tab to a new window but not a *pane* out of a tiled layout (01). Terminal widgets in this
   system are genuinely server-side, so tmux-style detach/reattach with live state is
   achievable for the local-resource half of the tab space (01).
10. **Make the workspace a declarative, version-controllable file.** `tmuxinator` and
    `.code-workspace` prove it; no browser has it (01). A workspace definition that names
    URIs across schemes — files, folders, terminals, and web pages — is a capability nothing
    in the browser market offers.
11. **Persist "what's open" as durable state by default, with no save gesture.** IDEs do this
    and browsers make it opt-in; the divergence is historical, not technical (04). In a
    unified system where half the open resources are unsaved local edits with no permanent
    home elsewhere, the IDE default is the correct one.
12. **Build one addressing primitive that every open thing implements, and one open-handler
    registry that resolves it.** Every successful heterogeneous system has exactly one:
    Theia's `OpenerService` + `WidgetOpenHandler`, VS Code's `EditorInput`, KIO workers,
    Emacs buffer names with TRAMP encoding remoteness inside the name (04). Do not bolt a
    second navigation surface alongside the tab strip — Vivaldi Panels and Opera M2 stop the
    "everything is addressable" promise at the panel boundary, which blocks cross-surface
    search and drag (04).
13. **Ship a list/graph view of everything open, independent of the strip's spatial layout.**
    Once resource types are varied, looking at the tab strip stops being sufficient triage;
    Open Editors, Ibuffer, and Obsidian's graph are the proven forms (04). Ibuffer's grouping
    by predicate — including grouping all remote buffers under their host — is the most mature
    surveyed model for a heterogeneous open-resource list (04).
14. **Treat ~8 and ~20 open resources as the two design thresholds, and switch affordances at
    them rather than assuming a fixed persona.** Management problems begin at a median of 8,
    reached weekly by 67% of users; spatial scanning fails and keyboard search takes over
    around 20 (02)(03). The same person crosses both within one session.
15. **Handle accumulation and organisation with separate mechanisms, and never auto-close for
    the never-close segment.** Hoarding/minimalism is orthogonal to organised/messy, and all
    four quadrants are populated (02). Auto-archive must be a per-workspace policy with an
    explicit off switch, because the 600–1000-tab tree users actively reject any tool that
    prunes for them (03) — and because trust in anything that touches tab contents is a
    first-class evaluation criterion the market has burned twice (03).

---

## 5. Figures explicitly marked UNVERIFIED

Do not cite any of the following; each was traced to SEO/marketing content or
AI-generated snippets rather than a primary source (02):

- "Panorama had <4.2% of daily active users per Mozilla Telemetry Q3 2016" — **UNVERIFIED**;
  no Mozilla document, bug, or dashboard publishes this. The removal rationale is
  documented-qualitative only.
- "Average +117 MB per tab group" (Panorama) — **UNVERIFIED**.
- "Stanford HCI lab 2024: colour-coded tabs reduce task-switching time 23%" — **UNVERIFIED**;
  no such publication traced.
- "Nielsen Norman Group: average knowledge worker keeps 40–60 tabs" — **UNVERIFIED**.
- "N=217 longitudinal eye-tracking/KLM study: AI tab organisers make context switching 41%
  faster, 37% fewer 'why did I open that' moments" — **UNVERIFIED**.

Defensible substitutes for the above: Chang et al. 2021 (median problem threshold 8 tabs),
Mozilla 2025 (median ~25 tabs among grouping-feature users), Rutishauser & Fritz 2026
(>60 tab switches/hour) (02).

Additionally treat as *indicative, not authoritative* (01): cross-browser tab-count averages
(~10–11 per session), "40–50% of power-user survey respondents above 20 tabs", and 2026
version numbers/rollout dates sourced from comparison blogs — the directional claims are
corroborated across several such sources, the exact figures are not.
