# 02 — Academic and Industry Research on Tab, Bookmark, Session and Saved-Page Management

Compiled 2026-08-19. Every study below was verified against a primary source (publisher record, author-hosted PDF, or archived full text) rather than a secondary summary. Where a claim could only be traced to a marketing blog or an AI-generated search snippet, it is explicitly flagged as **UNVERIFIED** and excluded from the evidence base.

The core deliverable — every empirically-grounded user segmentation found — is consolidated in **Part 6**.

---

## Part 1 — Browser tab research

### 1.1 Dubroy & Balakrishnan (2010) — the first real tab study

**Citation.** Patrick Dubroy and Ravin Balakrishnan. 2010. *A Study of Tabbed Browsing Among Mozilla Firefox Users.* In Proceedings of the SIGCHI Conference on Human Factors in Computing Systems (CHI '10). ACM, 673–682. DOI: 10.1145/1753326.1753426

**Method.** Mixed methods. A custom Firefox extension logged low-level UI and network events on participants' primary computers for 13–21 days each. Supplemented by randomly-prompted diary entries and interviews. Participants reviewed their own logs before submitting them (privacy design: URLs were withheld from researchers; only relationships between pages were retained).

**Sample.** 21 participants (13 F, 8 M; 15 aged 18–29, 4 in 30s, 2 in 50s). Deliberately non-CS-heavy: only 5 had a CS/engineering background; others from education, environmental science, business, psychology. 6 full-time students, 15 office workers. One dropped out mid-study.

**Key findings.**
- **Bimodal tab-creation distribution.** 13/21 participants clustered around 0.04 tabs created per navigation action (~4 tabs per 100 navigations). The remaining 8 clustered around 0.14 — roughly 3× the rate. Two extreme cases created 17 and 22 tabs per 100 navigations.
- **"Tab power users" = the upper mode, 8 of 21 (38%).** The four highest rates belonged to the four CS/programming participants, but the other four power users were a civil engineer and people from communications, marketing and the humanities. **Tab-heavy behaviour is not simply a technical-background effect.**
- Median open tabs was ≤6 for all but one participant (P14 at 17). Highest single peak: 42 tabs (P20). 7 participants had a median of only 1–2 tabs.
- Median/max divergence is large: P19 had a median of 1 tab but a max of 27. People who normally keep few tabs still burst.
- Two power users (P3, P14) switched tabs 2–3× more often than their tab-creation rate would predict — **creation rate and switch rate are separable dimensions**.
- **Tabs displaced the Back button.** Back accounted for a median 7.1% of navigation actions overall, 5.8% among tab power users (vs. 41% in Catledge & Pitkow 1995, ~30% in Tauscher & Greenberg 1997, 14% in Weinreich et al.).
- **Tabs are a revisitation mechanism.** A median 73.3% of tab switches were revisitations (range 45–91%; aggregate 77.7%). For 17 of 21 participants, tab switching was more frequent than Back navigation.
- 14 participants — including every power user — reported deliberately opening links in background tabs. All power users did this; none of the non-power-users did. This is the single clearest behavioural marker separating the two modes.
- Stated reasons tabs beat Back/history/bookmarks: no page reload, scroll position preserved, predictable (Back may need an unknown number of clicks), spatial/visual overview.

**Segmentation established.** *Tab power users* vs. *ordinary tab users* — an empirically bimodal, log-derived split, not a self-report label.

---

### 1.2 Chang, Hahn, Kim, Coupland, Breneisen, Kim, Hwong & Kittur (2021) — the canonical tab-overload paper

**Citation.** Joseph Chee Chang, Nathan Hahn, Yongsung Kim, Julina Coupland, Bradley Breneisen, Hannah S. Kim, John Hwong, and Aniket Kittur. 2021. *When the Tab Comes Due: Challenges in the Cost Structure of Browser Tab Usage.* In Proceedings of the 2021 CHI Conference on Human Factors in Computing Systems (CHI '21). ACM, Article 100, 1–15. DOI: 10.1145/3411764.3445585

**Method.** Three-part.
1. *Preliminary survey* (N=64; age 19–67, M=33.7, SD=10.6; 57% male; 77% US) to establish the prevalence of tab clutter.
2. *Study 1 — in-person interviews.* Purposive sampling of heavy-tab researchers at a research university and a research agency (California and Pennsylvania). Screening criteria: (a) actively conducting research projects or literature reviews, (b) self-reported often reaching 12+ open tabs. N=10 (age 19–32, M=23.0, SD=4.7, 40% male, mostly grad/undergrad). Each: one 45-minute interview plus **three additional 10-minute interviews over two weeks**, each time walking through *every* currently open tab and explaining why it was still open. Open coding, four iterations, two independent coders, four authors merging themes.
3. *Study 2 — MTurk survey.* N=103 (age 21–70, M=36.33, SD=10.65; 49.5% female; mostly US; 53% bachelor's, 24% high-school, 19% master's). Participants described each currently open tab (capped at 10 tabs; 7.8% hit the cap) and labelled it with the pressure categories from Study 1. Mean 6.15 tabs labelled per person (SD=3.69), **N=633 tabs total**.

**Key findings — prevalence (preliminary survey, N=64).**
- 59% agreed that if they checked their tabs right now they would find tabs that should be closed.
- Only 19% agreed laziness contributed to not closing them.
- **55% agreed they "feel like they can't let go of their tabs."**
- 28% often struggle to *find* the tab they need.
- 25% had experienced a browser or computer crash from too many tabs.
- **30% agreed they "have a tab hoarding problem."**
- Prior 2019 survey cited by the authors: 50.7% of N=75 considered tab clutter a problem.

**Key findings — thresholds (Study 2, N=103).**
- Asked at how many open tabs they start having management problems, responses were long-tailed with a **median threshold of 8 tabs** (Q1=5, Q3=12).
- **67% reach that threshold at least once a week** (17% daily; 9% 4–6×/week; 25% 2–3×/week; 16% once/week).

**Key findings — the two-force cost model (Table 1 of the paper).**

*Pressures to CLOSE tabs (4 categories):*
| Code | Name | Description |
|---|---|---|
| C1 | Limited Attention | Too many tabs makes it hard to focus; overwhelming, causes stress |
| C2 | Limited Screen Real-estate | Hard to navigate, loss of situational awareness |
| C3 | Limited Computing Power | Drains processing power; browser and machine slow down |
| C4 | Pressure to be Organized | Social and self pressure to avoid looking disorganized |

*Pressures to KEEP tabs open (6 categories), with the share of the 633 labelled tabs:*
| Code | Name | Description | % of tabs |
|---|---|---|---|
| O1 | Reminders and Resumption | Tab as reminder / progress marker for an unfinished task | **37.3%** |
| O2 | Revisiting References | Frequently-used tab kept for cheap access; diminishing return | **41.7%** |
| O3 | Avoid Costly Re-finding | Fear of not being able to find it again | **17.7%** |
| O4 | Sunk Costs and Aspirations | Hoping to process more than is realistically possible | **16.6%** |
| O5 | External Mental Model | Tabs/windows/desktops as externalised memory and task structure | **20.1%** |
| O6 | Uncertain Relevance | Cannot judge whether it will matter later | **5.7%** |
(Tabs could carry more than one pressure; the paper's figure reports these rounded as 37/42/18/17/20/6.)

**The "black hole" finding.** Participants reported that bookmarking or archiving a tab felt equivalent to losing it: bookmarks lack the reminding function, and archived tabs are rarely re-accessed. The paper's design-implications section names this explicitly as *"Avoid black-hole effect."* Kittur's public framing (CMU News, 2021): *"People feared that as soon as something went out of sight, it was gone. Fear of this blackhole effect was so strong that it compelled people to keep tabs open even as the number became unmanageable."*

**Emotional findings.** Participants reported stress, being overwhelmed, and **shame** about their tab count — some clean up tabs before screen-sharing or presenting (C4). One participant: *"it's...shameful in a way... I would have been giving a bad impression to these people that had to look at my screen."*

**Segmentation established.** No user typology is offered — the paper's contribution is a *pressure* taxonomy applied to tabs, not to people. It does, however, establish the self-identified "tab hoarder" (30% of the preliminary sample) as a real population, and its Limitations section notes the sample was not designed to detect demographic differences in tab behaviour.

---

### 1.3 Chang & Kittur (2021) — Tabs.do / Skeema (intervention evidence)

**Citation.** Joseph Chee Chang, Yongsung Kim, Victor Miller, Michael Xieyang Liu, Brad A. Myers, and Aniket Kittur. 2021. *Tabs.do: Task-Centric Browser Tab Management.* In Proceedings of the 34th Annual ACM Symposium on User Interface Software and Technology (UIST '21). ACM, 663–676. DOI: 10.1145/3472749.3474777

**Method.** Chrome extension reframing tabs as tasks; ML-based suggestions for grouping open tabs into task bundles using behavioural + semantic features. Field deployment with participants' real tasks in the wild.

**Key findings.** Decreased tab clutter; enabled rich task structures with lightweight interactions; more efficient context switching between tasks. The successor tool **Skeema** (same group) reported that early users *"significantly reduced the number of tabs and windows they kept open, reported much less stress connected to tabs, and remained more focused."*

**Relevance.** This is the strongest available evidence that **the problem is tractable by re-representation, not by discipline** — grouping-by-task lowers tab counts and stress without asking users to be tidier.

---

### 1.4 Ma, Lassila, Nurgalieva & Lindqvist (2023) — browsing clutter

**Citation.** Rongjun Ma, Henrikki Lassila, Leysan Nurgalieva, and Janne Lindqvist. 2023. *When Browsing Gets Cluttered: Exploring and Modeling Interactions of Browsing Clutter, Browsing Habits, and Coping.* In Proceedings of the 2023 CHI Conference on Human Factors in Computing Systems (CHI '23). ACM, 20 pages. DOI: 10.1145/3544548.3580690

**Method.** Interview study (N=16) plus online survey (N=400). Structural model predicting browsing clutter from browsing habits and coping strategies.

**Key findings.**
- "Browsing clutter" is multi-dimensional: **(a) number of tabs and windows, (b) web page content and interactive elements (ads, pop-ups), (c) navigation, (d) the search process.** Tab count is only one of four clutter sources — a point most tab tooling ignores.
- Sources of clutter run from *task characteristics* (importance, complexity) to *user habits* (multitasking, tab-closing behaviour).
- **Hoarding tabs and leaving them unvisited at day's end was common**, including tabs participants had forgotten about or that no longer served any task.
- **Coping taxonomy: problem-focused vs. emotion-focused.** Only problem-focused strategies (e.g. consciously setting an upper bound on open tabs) actually reduced clutter. Emotion-focused strategies (ignoring the stress, reframing it, complaining about it) perpetuated the problem.
- The authors' public framing (Aalto press release, April 2023): **one in four internet users report being overwhelmed by browser clutter.**
- Caution flagged by the authors: "organizing" tools can rearrange clutter without removing it.

**Segmentation established.** A *coping-strategy* segmentation (problem-focused vs. emotion-focused copers) with an outcome difference attached — the only tab-domain segmentation in the literature that is validated against an outcome rather than merely descriptive.

---

### 1.5 Rutishauser & Fritz (2026) — from tabs to structures

**Citation.** Roy Adrian Rutishauser and Thomas Fritz. 2026. *From Tabs to Structures: Understanding and Supporting Web Page Management.* In Proceedings of the 2026 CHI Conference on Human Factors in Computing Systems (CHI '26). DOI: 10.1145/3772318.3791979

**Method.** Field experiment with 29 knowledge workers (primarily students and IT professionals). Phase 1 observed existing web-management practice over time; Phase 2 deployed a technology probe (**Gstell**) to observe behavioural adaptation.

**Key findings.**
- Participants used browsers heavily for work, **switching tabs more than 60 times per hour**, and kept tabs open especially when future relevance was uncertain (matching Chang et al.'s O6).
- Many participants "frequently struggle with excessive but **inactive** browser tabs" — the load is dominated by tabs that are not being used, not by tabs in play.
- Existing bookmarking systems are characterised as **too rigid**, imposing high cognitive cost to create and maintain.
- The design position: occupy the space **between the chaotic flat structure of open tabs and rigid hierarchies**, by pairing *pile-like short-term storage* with *structured long-term storage*. This is Malone's 1983 files/piles distinction re-derived for browsers 43 years later.

---

### 1.6 Supporting log-scale tab evidence

**Huang & White (2010).** Jeff Huang and Ryen W. White. 2010. *Parallel Browsing Behavior on the Web.* In Proceedings of the 21st ACM Conference on Hypertext and Hypermedia (HT '10). ACM, 13–18. DOI: 10.1145/1810617.1810622
Log study of millions of toolbar users. **Parallel browsing is more prevalent than linear browsing.** Users switch tabs at least 57.4% of the time. Outclicks and tab switches follow a **power-law distribution** — i.e. tab-intensity across the population has no meaningful "typical user"; the tail dominates. Parallel browsing splits activity across tabs rather than increasing total pageviews.

**Huang, Lin & White (2012).** Jeff Huang, Thomas Lin, and Ryen W. White. 2012. *No Search Result Left Behind: Branching Behavior with Browser Tabs.* In Proceedings of WSDM '12. ACM, 203–212. Roughly three-fifths of participants preferred opening search results in tabs (as cited by Chang et al. 2021); at least 60% of a person's tabs were related to other tabs from the same task — the empirical basis for automated task-grouping.

**Hisatomi & Tajima (2025).** *Graph-Structure-Based Grouping of Web Browser Tabs into Tasks.* UIST '25 Adjunct. DOI: 10.1145/3746058.3758439. Automatic tab-to-task classification using tab-opening graph structure only, avoiding privacy-sensitive content and manual annotation.

---

## Part 2 — Bookmarks and saved pages

### 2.1 Abrams, Baecker & Chignell (1998) — the founding bookmark study

**Citation.** David Abrams, Ron Baecker, and Mark Chignell. 1998. *Information Archiving with Bookmarks: Personal Web Space Construction and Organization.* In Proceedings of the SIGCHI Conference on Human Factors in Computing Systems (CHI '98). ACM Press/Addison-Wesley, 41–48. DOI: 10.1145/274644.274651

**Method.** Survey (322 completed of 450 distributed; 200 fully complete; 76.5% male) at a university plus industry/government respondents, developed from a 12-person pilot. Plus **automated analysis of the actual bookmark archives of 50 users**, aggregated to trace one year of growth.

**Key findings.**
- **Bookmark filing strategy taxonomy — the direct ancestor of every tab-management typology (N=299 respondents classified):**

| Strategy | Definition | Share |
|---|---|---|
| **No filers** | Never organise; bookmarks stay in creation order | **26%** (37% checked "I don't organize bookmarks") |
| **Creation-time filers** | File each new bookmark into the right category immediately | **23%** |
| **End-of-session filers** | Organise all new bookmarks at the end of each session | **7%** |
| **Sporadic filers** | Organise occasionally; schedule explicit "spring-cleaning" sessions | **~44% (roughly half)** |

- The authors explicitly note these habits are "similar to the way users manage electronic mail archives" — i.e. they saw the Whittaker & Sidner mapping at the time.
- End-of-session filers spent significantly more time per session than sporadic or no-filers (p<0.05) and used folders most: **80% of end-of-session filers used folders**.
- **A visibility threshold drives structure.** One user: bookmarks get organised "when they no longer fit in my drop down menu." **Beyond ~35 bookmarks** (the easy-visibility threshold), folder count grew in direct linear relation to archive size (F[1,26]=38.52, p<.001).
- Growth is relentless and near-perfectly linear: N = 7.45 + 0.18·T (N = bookmarks, T = days), R=.996, p<0.001 — **about one new bookmark every 5.5 days per user**, with essentially no pruning.
- 86% of a user's bookmarks on average pointed to **distinct domains** — people bookmark *sites*, not pages.
- Context: a 1996 survey of 6,619 users (Pitkow) found >80% cited bookmarks as an information-locating strategy and 34% named "organizing retrieved information" a top-three web usability problem.

**Segmentation established.** No-filer / creation-time filer / end-of-session filer / sporadic (spring-cleaning) filer.

---

### 2.2 Cockburn & McKenzie (2001) — bookmark collections at scale

**Citation.** Andy Cockburn and Bruce McKenzie. 2001. *What Do Web Users Do? An Empirical Analysis of Web Use.* International Journal of Human-Computer Studies 54, 6, 903–922. DOI: 10.1006/ijhc.2000.0459

**Method.** Retrospective analysis of Netscape Navigator `history.dat` and `bookmarks.html` files recovered from **incremental system backups** over a four-month (119-day) window — so participants used their normal browser, unaware of instrumentation timing.

**Sample.** 17 unpaid volunteers (7 faculty, 3 programming staff, 3 tutors, 4 graduate students) in a Computer Science department. 84,841 page visits, 17,242 distinct URLs.

**Key findings.**
- **~81% revisitation rate** — far above Tauscher & Greenberg's 58% and Catledge & Pitkow's 61%. Individual rates ranged **61.0% to 92.3%**.
- **Bookmark collections are large and monotonically growing.** Mean maximum collection size **184 bookmarks (SD=166.15)**; mean **18.1 folders (SD=16.5)**. Range: one participant had zero bookmarks, two had over 500.
- Deletion rates were very low — collections continually grow. Two months after collection, many bookmarks were already stale.
- Bookmarks accounted for only ~2% of navigation actions in earlier studies despite the large collections. **People build big archives they barely navigate from.**
- Design implications the authors drew, still unaddressed 25 years later: bookmark systems must (1) scale to large collections, (2) actively assist in managing/pruning them, (3) special-case the handful of pages visited far more than all others.

---

### 2.3 Bergman, Whittaker & Schooler (2021) — bookmarks are created but not used

**Citation.** Ofer Bergman, Steve Whittaker, and Joel Schooler. 2021. *Out of Sight and Out of Mind: Bookmarks Are Created but Not Used.* Journal of Librarianship and Information Science 53, 2, 338–348. DOI: 10.1177/0961000620949652

**Method.** *Elicited personal information retrieval* — 50 participants were each asked to retrieve 21 target URLs, of which **5 were drawn from that participant's own bookmarks**, presented in random order so participants would not realise which were bookmarked.

**Key findings.**
- Most participants had created bookmarks, but **only 41 of 250 bookmarked targets (16%) were actually retrieved using the bookmark facility**.
- Of those 41, **only 9 (4% of all bookmarked targets) came from the bookmarks *menu hierarchy***; the other 32 came from the always-visible bookmarks bar.
- Conclusion: **unless a bookmark is in full view, it is not used.** Visibility, not organisation, determines retrieval.

**Design consequence.** This is the empirical core of the "black hole" fear in Chang et al. (2021): the users' fear is *correct*. Filing something out of sight statistically means never retrieving it that way again.

---

### 2.4 Jones, Bruce & Dumais (2001, 2002); Bruce, Jones & Dumais (2004) — keeping behaviours

**Citations.**
- William Jones, Harry Bruce, and Susan Dumais. 2001. *Keeping Found Things Found on the Web.* In Proceedings of the Tenth International Conference on Information and Knowledge Management (CIKM '01). ACM, 119–126. DOI: 10.1145/502585.502607
- William Jones, Susan Dumais, and Harry Bruce. 2002. *Once Found, What Then? A Study of "Keeping" Behaviors in the Personal Use of Web Information.* Proceedings of ASIST 39, 1, 391–402. DOI: 10.1002/meet.1450390143
- Harry Bruce, William Jones, and Susan Dumais. 2004. *Keeping and Re-finding Information on the Web: What Do People Do and What Do They Need?* Proceedings of ASIST 41, 1, 129–137. DOI: 10.1002/meet.1450410115
- William Jones. 2007. *Keeping Found Things Found: The Study and Practice of Personal Information Management.* Morgan Kaufmann. ISBN 978-0123708663

**Method.** Observational studies of information workers, later a two-phase staged-observation study across a sample of 214 individuals covering both keeping and re-finding.

**Key findings — the observed keeping methods.** People used, in parallel:
1. Emailing a URL to self
2. Emailing a URL to someone else (with comments)
3. Printing the page out
4. Saving the page as a file to disk
5. Pasting the URL into a document
6. Pasting the URL into a personal website
7. Bookmarking / Favorites
8. Doing nothing deliberately — **relying on being able to search for it again**

**The finding that matters.** The two tools browsers built explicitly for this job — **bookmarks and the history list — were not widely used**. People reached for email-to-self, paper and documents instead. Jones' framing: keeping is a *multi-purpose* activity carrying reminding, context and prospective-task functions that a URL store does not serve; and the *cost of the keeping decision itself* (deciding where something goes) is the bottleneck, not storage.

---

## Part 3 — Revisitation and re-finding: why "saving" matters at all

### 3.1 Tauscher & Greenberg (1997)

**Citation.** Linda Tauscher and Saul Greenberg. 1997. *How People Revisit Web Pages: Empirical Findings and Implications for the Design of History Systems.* International Journal of Human-Computer Studies 47, 1, 97–137. DOI: 10.1006/ijhc.1997.0125

23 subjects (CS staff/faculty/students plus telecom programmers), instrumented XMosaic. Defined the **recurrence rate**: the probability that any URL visited is a repeat of a previous visit. Found **58%**, and 61% on re-analysis of 55 of Catledge & Pitkow's subjects. Coined the characterisation of the web as a **"recurrent system"** — roughly three of every five pages visited had been seen before. Back accounted for ~30% of navigation actions.

### 3.2 Obendorf, Weinreich, Herder & Mayer (2007) — the definitive long-term click-stream study

**Citation.** Hartmut Obendorf, Harald Weinreich, Eelco Herder, and Matthias Mayer. 2007. *Web Page Revisitation Revisited: Implications of a Long-Term Click-Stream Study of Browser Usage.* In Proceedings of the SIGCHI Conference on Human Factors in Computing Systems (CHI '07). ACM, 597–606. DOI: 10.1145/1240624.1240719 (CHI 2007 Honourable Mention)

**Method.** Intermediary proxy (Scone/WBI) plus an instrumented Firefox logging all page requests, triggering user actions, and page characteristics, over **52–195 days per participant**. Two 90-minute interviews per participant (start and end).

**Sample.** 25 unpaid volunteers, 6 female (24%), ages 24–52 (M=30.5), 3–12 years web experience (mean 8), 64% CS-affiliated, 36% other. Germany, Netherlands, Ireland, New Zealand.

**Key findings.**
- **The recurrence rate is an artefact of definition.** Depending on how a "revisit" is counted, the same dataset yields:

| Definition | Recurrence rate |
|---|---|
| Truncated URI (ignore GET/POST params) | 69.4% |
| Full URI + GET params | **45.6%** |
| Full URI + GET + POST params | 43.7% |
| Page *content* fingerprint | 34.6% |

  Without log cleaning (ads, frames, auto-reloads), individual rates were 6–20% higher and the mean was 54.1%. The apparent rise from 58% (1997) to 81% (2001) **did not continue**; measured properly, contemporary revisitation was much lower than the folklore figure.
- **Huge inter-individual variance.** Per-user recurrence ranged from **17.4% to 61.4%**. The authors state plainly that drawing user-requirement conclusions from mean recurrence rates is "potentially misleading." They looked for demographic explanations (profession, gender, nationality) and **found no supportable effects** — the variation is personal habit, not category membership.
- Navigation action mix: links 43.5%, Submit 15.3%, Back 14.3%, New Window/tab 10.5%, Direct Access 9.4%, Reload 1.7%, Forward 0.6%.
- **The revisit-interval taxonomy that everything downstream uses:**

| Type | Interval | Share of all revisits | Intent |
|---|---|---|---|
| **Short-term** | < 1 hour | **72.6%** | backtrack / undo |
| **Medium-term** | 1 hour – 1 day | **12%** | re-utilise / observe |
| (medium-long) | 1 day – 1 week | **7.8%** | |
| **Long-term** | > 1 week | **7.6%** | rediscover |

- Back caused only 31% of all revisits; over 50% were triggered by other navigation, mostly link-following. Browser support (Back, history, bookmarks) is optimised for short-term revisits, which are already the easy case; **medium- and long-term revisits — where tools would actually help — have different patterns and are poorly supported.**

### 3.3 Weinreich, Obendorf, Herder & Mayer (2008) — the tab/window transition

**Citation.** Harald Weinreich, Hartmut Obendorf, Eelco Herder, and Matthias Mayer. 2008. *Not Quite the Average: An Empirical Study of Web Use.* ACM Transactions on the Web 2, 1, Article 5. DOI: 10.1145/1326561.1326566 (same dataset as 3.2)

- Opening a page in a **new window or tab rose from <1% of navigation actions in the mid-1990s to over 10%**.
- Participants had on average **2.1 windows or tabs open** when accessing a new page — parallel browsing was already the norm by 2005–2006.
- **Only 6 of 15 Firefox users (40%) regularly opened tabs** at that time; the rest used multiple windows or neither.
- **Tab users backtrack less**: frequent tab users had 9.9% Back usage vs. 18.3% for the nine who barely used tabs (t=2.311, p=0.038). Multiple-window users likewise (16.4% bottom third, t=2.509, p=0.026).
- Reported strategy split: **tabs for closely related tasks, separate windows for parallel tasks.** This is a spontaneous, user-invented two-level hierarchy — and it is the behaviour that tab-group features later formalised.
- Opening links in new tabs/windows **breaks the Back button's contract**, since the new surface has no history to return through.

### 3.4 Teevan, Adar, Jones & Potts (2007) — re-finding via search

**Citation.** Jaime Teevan, Eytan Adar, Rosie Jones, and Michael A. S. Potts. 2007. *Information Re-Retrieval: Repeat Queries in Yahoo's Logs.* In Proceedings of the 30th Annual International ACM SIGIR Conference (SIGIR '07). ACM, 151–158. DOI: 10.1145/1277741.1277770

One-year Yahoo query log of 114 anonymous users, plus a controlled survey of 119 additional volunteers. **As many as 40% of all queries are re-finding queries.** Changes to search result rankings measurably *hinder* re-finding. **Search is a de facto bookmarking system** — the "do nothing, I'll search for it again" strategy from Jones et al. is the single most-used keeping method by volume.

### 3.5 Adar, Teevan & Dumais (2008) — revisitation patterns at web scale

**Citation.** Eytan Adar, Jaime Teevan, and Susan T. Dumais. 2008. *Large Scale Analysis of Web Revisitation Patterns.* In Proceedings of the SIGCHI Conference on Human Factors in Computing Systems (CHI '08). ACM, 1197–1206. DOI: 10.1145/1357054.1357241

Five weeks of interaction logs from **over 612,000 opt-in users**, plus a survey of revisitation intent. Revisitation curves for 55,000 sampled URLs were clustered (repeated-bisection, cosine similarity) into **four page-level revisitation groups**:

| Group | Interval | Pages in sample | Characteristic content |
|---|---|---|---|
| **Fast** | < 1 hour | 23,611 | pornography & spam, hub-and-spoke sites, shopping & reference, auto-refresh, monitoring |
| **Medium** | 1 hour – 1 day | 9,421 | popular homepages, communication, .edu, browser homepages |
| **Slow** | > 1 day | 18,422 | entry pages, weekend activity, search engines used for revisitation, children's content, software updates |
| **Hybrid** | bimodal | 3,334 | popular but infrequently used, entertainment & hobbies |

Self-reported intent aligned with the clusters, and self-reported *regularity* of access decreased as the interval lengthened. **This is a segmentation of pages, not people** — but it is directly actionable: a tab/save system can classify a URL's expected revisit horizon and treat fast, medium, slow and hybrid pages differently.

### 3.6 Aula, Jhaveri & Käki (2005); Aula & Käki (2005)

**Citations.**
- Anne Aula, Natalie Jhaveri, and Mika Käki. 2005. *Information Search and Re-access Strategies of Experienced Web Users.* In Proceedings of the 14th International Conference on the World Wide Web (WWW '05). ACM, 583–592. DOI: 10.1145/1060745.1060831
- Anne Aula and Mika Käki. 2005. *Hundreds of Folders or One Ugly Pile — Strategies for Information Search and Re-access.* In Human-Computer Interaction — INTERACT 2005, LNCS 3585, Springer. DOI: 10.1007/11555261_78

**Method.** Survey of **236 experienced web users**.

**Key findings.** Experienced users rely on strategies the browser does not support — most notably **using several browser windows in parallel**, which they rated as important. Strategies recommended by earlier research (e.g. maintaining URL lists on a personal web page) were unimportant to them. Even experienced users had significant search and re-access problems.

**Segmentation established.** The companion INTERACT paper's title is the segmentation itself: **"hundreds of folders" organisers vs. "one ugly pile" non-organisers** — the filing/piling split observed directly in web re-access strategy. (Note: this paper's PDF uses non-extractable font encoding; findings above are taken from the abstract, publisher record, and the WWW '05 sibling paper.)

---

## Part 4 — PIM foundations: filing vs. piling

### 4.1 Malone (1983) — the origin

**Citation.** Thomas W. Malone. 1983. *How Do People Organize Their Desks? Implications for the Design of Office Information Systems.* ACM Transactions on Office Information Systems 1, 1, 99–112. DOI: 10.1145/357423.357430 (~940 citations)

**Method.** Series of interviews with professional and clerical office workers, observing their actual desks and offices.

**Key findings.**
- **Filers vs. pilers.** *Filers* ("neat") assign every document a category and place it in the corresponding location — folder, tray, or named pile. *Pilers* ("messy") accumulate documents in unstructured piles with little attempt at categorisation.
- **The reminding function.** A crucial function of desk organisation is **reminding the user of things to do**, not merely enabling retrieval. Piles are visible; files are not. This is the direct ancestor of Chang et al.'s O1 (Reminders and Resumption) and of the "black hole" fear.
- **Categorisation cost is the explanatory variable.** The cognitive difficulty of deciding *which* category a thing belongs to explains why people don't file. Hierarchies have overlapping, ambiguous categories, so diligent filing does not guarantee successful retrieval.

Everything in the tab literature is a rediscovery of these three points on a new substrate.

### 4.2 Whittaker & Sidner (1996) — email overload and the three-way split

**Citation.** Steve Whittaker and Candace Sidner. 1996. *Email Overload: Exploring Personal Information Management of Email.* In Proceedings of the SIGCHI Conference on Human Factors in Computing Systems (CHI '96). ACM, 276–283. DOI: 10.1145/238386.238530 (~1,150 citations)

**Method.** Interviews plus quantitative analysis of the actual mailboxes and folder structures of information workers at a large technology company.

**Key findings.** Email had already become a task-management and archiving system, not a communication tool. Inboxes held hundreds of messages mixing outstanding tasks, partially-read documents and conversation threads. The working inbox contains **to-dos, to-reads and indeterminate-status items**.

**Segmentation established — the most-cited typology in this entire field:**

| Type | Behaviour | Failure mode |
|---|---|---|
| **Frequent filers** | File or delete daily, keep the inbox near-empty | High filing effort; premature filing before value is known; lowest rate of "failed folders" (folders with <3 messages) |
| **Spring cleaners** | Let the inbox accumulate, then purge in occasional intensive sessions (every 1–3 months); heavy folder users | Backlog anxiety between cleanups |
| **No filers** | Never file; everything stays in the inbox | Overloaded inbox; reliance on search and scroll |

### 4.3 The lineage after Whittaker & Sidner

- **Bälter (1998).** Olle Bälter. *Electronic Mail in a Working Context.* Doctoral thesis, Royal Institute of Technology (KTH), Stockholm. Extends the trichotomy with folder-usage measurements; the "failed folder" concept (folders holding too few messages to reduce inbox complexity) comes from this line.
- **Gwizdka (2004).** Jacek Gwizdka. *Email Task Management Styles: The Cleaners and the Keepers.* CHI '04 Extended Abstracts. ACM, 1235–1238. DOI: 10.1145/985921.986032. **Segmentation: cleaners vs. keepers**, based specifically on handling of messages that reference *future tasks or events*. The difference was attributable to **email experience and to a requirement for flexibility of closure** — keepers deliberately preserve open-endedness. This is the closest email analogue to tab hoarding, since it is about prospective tasks rather than archives.
- **Fisher, Brush, Gleave & Smith (2006).** *Revisiting Whittaker & Sidner's "Email Overload" Ten Years Later.* CSCW '06. ACM, 309–312. DOI: 10.1145/1180875.1180922. Re-measured mailboxes a decade on with much larger storage quotas; found archives had grown by roughly an order of magnitude while inbox management strategies persisted.
- **Kalman & Ravid (2015).** Yoram M. Kalman and Gilad Ravid. *Filing, Piling, and Everything in Between: The Dynamics of E-mail Inbox Management.* JASIST 66, 12, 2540–2552. DOI: 10.1002/asi.23337. **Unobtrusive longitudinal telemetry of thousands of users worldwide over 8 months** — by far the largest-N study of this behaviour. Finding: **most email users are pilers, not filers**, and the classic trichotomy is better modelled as a continuum of filing intensity than as three discrete types.

### 4.4 Boardman & Sasse (2004) — the cross-tool study

**Citation.** Richard Boardman and M. Angela Sasse. 2004. *"Stuff Goes into the Computer and Doesn't Come Out": A Cross-Tool Study of Personal Information Management.* In Proceedings of the SIGCHI Conference on Human Factors in Computing Systems (CHI '04). ACM, 583–590. DOI: 10.1145/985692.985766 (~480 citations)
See also: Richard Boardman. 2004. *Improving Tool Support for Personal Information Management.* PhD thesis, Imperial College London. (Author-hosted at github.com/rickschmoo/phd-thesis-cross-tool-pim-2004)

**Method.** Two novel contributions: (1) **cross-tool** data — files, email and web bookmarks collected for the *same* participant, rather than one tool in isolation; (2) **longitudinal** re-visits for a subset of participants.

**Key findings.**
- **Strategies are tool-specific, not person-wide, in the general case** — but a subset of users apply a consistent organising philosophy across all three tools. The important consequence: you cannot infer someone's tab behaviour purely from their file behaviour, and vice versa, except for the consistently-organised minority.
- Folder structures across files and email frequently **overlapped in topic but were maintained separately**, producing duplicated and diverging hierarchies.
- Bookmark collections were the most neglected of the three — largely "acquired but not managed" — matching Abrams and Cockburn.
- The title captures the summary finding: information reliably goes in and does not reliably come back out.

### 4.5 Vitale, Janzen & McGrenere (2018) — hoarding and minimalism

**Citation.** Francesco Vitale, Izabelle Janzen, and Joanna McGrenere. 2018. *Hoarding and Minimalism: Tendencies in Digital Data Preservation.* In Proceedings of the 2018 CHI Conference on Human Factors in Computing Systems (CHI '18). ACM, Paper 587, 1–12. DOI: 10.1145/3173574.3174161

**Method.** Semi-structured interviews with 23 participants of diverse backgrounds about what digital "stuff" they had kept over the years, why, how they used it, what mattered. Iterative thematic analysis, constructivist stance. The hoarding/minimalism frame **emerged from participants' own vocabulary halfway through the study**, not from the interview guide.

**Sample split (23 participants).**

| Tendency | Participants | n |
|---|---|---|
| **Hoarding** | P1, P2, P3, P8, P9, P11, P12, P15, P17, P20, P23 | **11 (48%)** |
| **Minimalism** | P10, P13, P16, P19, P21, P22 | **6 (26%)** |
| Even mix of both | P6, P7, P18 | 3 (13%) |
| Not classifiable | P4, P5, P14 | 3 (13%) |

**Key findings.**
- **Hoarding** = keeping large amounts of digital "stuff", rarely deleting, often retaining data explicitly described as valueless. It has an **emotional component** (fear of forgetting, of letting the past go) and a **practical component** (job or external requirements). Hoarders reported frustration at "being on top of" their data.
- **Minimalism** = avoiding storing much, or regularly running clean-up passes.
- **The critical finding for design: preservation tendency is ORTHOGONAL to organisation.** Some participants were organised, some messy, *independent* of whether they hoarded or minimised. Hoarder≠messy and minimalist≠tidy.
- The authors are careful: hoarding here means an everyday tendency, **not** the clinical hoarding disorder (van Bennekom et al. 2015 documented the first clinical digital-hoarding case).
- Both tendencies served **identity construction** — what people keep is partly who they think they are.

**This is the single most important segmentation for tab-tool design**, because it establishes that "give hoarders a tidier UI" is a category error: the two axes must be handled separately.

---

## Part 5 — Industry and telemetry evidence

### 5.1 Mozilla — Firefox tab groups with local AI (2025)

**Source.** *Under the hood: How Firefox suggests tab groups with local AI.* The Mozilla Blog, 2025. https://blog.mozilla.org/en/firefox/ai-tab-groups/

**Verified statistics.**
- **The median number of tabs among users of the tab-grouping feature is ~25.**
- Some "power" users have tab counts **reaching the thousands**.
- No distribution, percentile, or group-size data is published in that post.

Firefox 141 (July 2025) shipped local, on-device AI tab-group suggestion and naming, after Tab Groups shipped in early 2025 following the removal of the original **Panorama** tab-groups feature (shipped Firefox 4, 2011; removed in Firefox 45, 2016, for low usage and maintenance cost).

**UNVERIFIED — do not cite.** Search results surfaced claims that Panorama had "<4.2% of daily active users per Mozilla Telemetry Q3 2016" and "average +117 MB per group." Neither traces to any Mozilla document, bug, or telemetry dashboard; both appear to be AI-generated. Mozilla Bugzilla #1210773 ("Add telemetry to analyze usage of tab groups / panorama") exists but does not publish the figures. Treat the removal rationale as documented-qualitative only.

### 5.2 Chrome and Edge

- Chrome **Tab Groups** shipped in Chrome 81 (2020); saved/named groups persisting across sessions and syncing across signed-in devices matured through 2024–2025.
- Chrome's **"Organize Similar Tabs"** AI feature shipped in Chrome 121 (January 2024) as an explicitly user-triggered action — it does not silently regroup tabs in the background.
- Microsoft Edge shipped a comparable AI tab organiser (2026).

**UNVERIFIED — do not cite.** Search snippets attributed to "a 2024 study from Stanford's HCI lab" (color-coded tabs reduce task-switching time 23%), "Nielsen Norman Group" (average knowledge worker keeps 40–60 tabs), and an "N=217 longitudinal eye-tracking/KLM study" (AI tab organisers make context switching 41% faster, 37% fewer "why did I open that" moments) could not be traced to any real publication. All three originate from SEO/marketing blogs for tab-manager products. **Do not use these numbers.** The genuine, citable equivalents are Chang et al. 2021 (median problem threshold 8 tabs), Mozilla 2025 (median ~25 tabs among grouping users), and Rutishauser & Fritz 2026 (>60 tab switches/hour).

### 5.3 What real telemetry evidence actually exists

The honest summary: **there is no published, methodologically-sound, large-N distribution of open-tab counts.** The three defensible anchors are:
1. Dubroy & Balakrishnan 2010 (N=21, logged): medians ≤6, one at 17, max 42, bimodal creation rate.
2. Chang et al. 2021 (N=103, self-report): median *problem threshold* 8 tabs; mean 6.15 tabs described per participant.
3. Mozilla 2025 (telemetry, self-selected feature users): median ~25, tail into the thousands.
The gap between #1/#2 and #3 is real growth plus selection bias, and nobody has published the curve.

---

## Part 6 — CATALOGUE OF EMPIRICAL USER SEGMENTATIONS

Every segmentation below is grounded in observed or measured behaviour, not speculation. Ordered by directness of relevance to tabs.

| # | Segmentation | Source | Domain | Basis | Population split |
|---|---|---|---|---|---|
| 1 | **Tab power users** vs. ordinary tab users | Dubroy & Balakrishnan 2010 | Browser tabs | Logged tab-creation rate (bimodal); background-link-opening habit | 8/21 (38%) power users |
| 2 | **Self-identified tab hoarders** vs. rest | Chang et al. 2021 | Browser tabs | Self-report agreement with "I have a tab hoarding problem" | 30% of N=64 |
| 3 | **"Can't let go" of tabs** vs. rest | Chang et al. 2021 | Browser tabs | Self-report | 55% of N=64 |
| 4 | **Problem-focused copers** vs. **emotion-focused copers** | Ma et al. 2023 | Browser clutter | Coping behaviour; validated against clutter outcome | Only problem-focused reduced clutter |
| 5 | **No filers / creation-time filers / end-of-session filers / sporadic (spring-cleaning) filers** | Abrams, Baecker & Chignell 1998 | Bookmarks | Survey + archive analysis | 26% / 23% / 7% / ~44% (N=299) |
| 6 | **Frequent filers / spring cleaners / no filers** | Whittaker & Sidner 1996 | Email | Interview + mailbox measurement | — (the canonical trichotomy) |
| 7 | **Filers** vs. **pilers** ("neat" vs. "messy") | Malone 1983 | Physical desks | Interviews + direct observation | — (origin of the distinction) |
| 8 | **Pilers dominate** — trichotomy is really a continuum | Kalman & Ravid 2015 | Email | Telemetry, thousands of users, 8 months | Most users are pilers |
| 9 | **Cleaners** vs. **keepers** | Gwizdka 2004 | Email (future-task messages) | Handling of prospective-task items; explained by experience + need for closure flexibility | — |
| 10 | **Hoarding** vs. **minimalism** tendencies (a spectrum) — **orthogonal to organised/messy** | Vitale, Janzen & McGrenere 2018 | All personal digital data | Interviews; participants' own vocabulary | 11 hoarders / 6 minimalists / 3 mixed / 3 unclassifiable of 23 |
| 11 | **"Hundreds of folders"** organisers vs. **"one ugly pile"** non-organisers | Aula & Käki 2005 | Web re-access | Survey of 236 experienced users | — |
| 12 | **Tool-specific strategists** vs. **cross-tool consistent organisers** | Boardman & Sasse 2004 | Files + email + bookmarks | Cross-tool data from same participants | Minority are cross-tool consistent |
| 13 | **Tab-preferring** vs. **window-preferring** vs. **neither** | Weinreich et al. 2008 | Browser windows/tabs | Click-stream logs | 6/15 Firefox users (40%) regularly used tabs |
| 14 | **Tabs for related subtasks, windows for parallel tasks** | Weinreich et al. 2008 | Browser windows/tabs | Interview-reported strategy, log-corroborated | User-invented two-level hierarchy |
| 15 | **Fast / medium / slow / hybrid revisitation** *(pages, not people)* | Adar, Teevan & Dumais 2008 | Web pages | Clustering of revisit curves, 612k users | 23,611 / 9,421 / 18,422 / 3,334 of 55,000 sampled URLs |
| 16 | **Short-term (<1h) / medium-term (1h–1d) / long-term (>1wk) revisits** *(events, not people)* | Obendorf et al. 2007 | Revisit events | Click-stream + interviews | 72.6% / 12% / 7.6% (plus 7.8% at 1d–1wk) |
| 17 | **Individual recurrence rate as a stable personal trait** — 17.4%–61.4% range, **no demographic predictor** | Obendorf et al. 2007 | Revisitation | Long-term logs; explicit search for profession/gender/nationality effects | No supportable demographic effects |

### 6.1 Reading the catalogue

Four independent findings, each from a different decade and domain, converge:

1. **The population is genuinely bimodal or long-tailed on accumulation, in every substrate.** Desks (Malone), email (Whittaker & Sidner, Kalman & Ravid), bookmarks (Abrams, Cockburn), files (Vitale), tabs (Dubroy, Huang & White's power law). A design that assumes a typical user is wrong by construction.
2. **Accumulation tendency and organisation ability are separate axes** (Vitale et al. 2018, explicitly; Boardman & Sasse implicitly). The four quadrants — organised hoarder, messy hoarder, organised minimalist, messy minimalist — are all populated.
3. **Filing behaviour is driven by categorisation cost, not by discipline or tooling** (Malone 1983; restated by Chang et al. 2021 as "the cost of categorization can be too high for many users"). Every intervention that lowered categorisation cost (Tabs.do, Skeema, Gstell) reduced accumulation; every intervention that merely provided a place to file did not.
4. **The "black hole" fear is empirically justified.** Bergman et al. 2021 measured it: 16% bookmark-retrieval usage overall, 4% via the hierarchy. Filing something out of view really does mean losing it. Any design that answers tab overload with "just bookmark it" is answering with the thing users have correctly learned not to trust.

### 6.2 The functional taxonomy of a saved page (synthesis)

Merging Chang et al.'s six keeping pressures, Malone's reminding function, and Obendorf's revisit horizons yields the design-relevant classification of *what a saved thing is for*:

| Function | Evidence | Right storage semantics |
|---|---|---|
| **Reminder / unfinished task** | O1 (37.3% of tabs); Malone's reminding function | Must stay visible; must resurface; must not be filed away |
| **Frequent reference** | O2 (41.7%); Adar's fast/medium groups | Pin; fast access; diminishing returns past a small number |
| **Insurance against re-finding cost** | O3 (17.7%); Teevan's 40% re-finding queries | Cheap archive + strong search; visibility not required |
| **Aspiration / read-later** | O4 (16.6%) | Should decay; low salience; sunk-cost aware |
| **External mental model / task structure** | O5 (20.1%); Weinreich's tabs-vs-windows split | Groups, workspaces, spatial arrangement |
| **Undecided relevance** | O6 (5.7%); Rutishauser & Fritz's "uncertain future relevance" | Pile-like short-term buffer with automatic ageing |

---

## Part 7 — Gaps in the literature

Genuine, exploitable gaps as of August 2026:

1. **No published distribution of open-tab counts** from any browser vendor at scale. The Mozilla ~25 median is the only telemetry figure in the open, and it is drawn from a self-selected feature-using population.
2. **No study connecting tab behaviour to a validated personality or cognitive measure.** Obendorf explicitly looked for demographic predictors of recurrence rate and found none; nobody has tested need-for-cognitive-closure, conscientiousness, or working-memory span, despite Gwizdka's "flexibility of closure" result pointing straight at it.
3. **No longitudinal study of whether AI tab grouping changes tab counts over months.** Chrome (2024), Firefox (2025) and Edge (2026) all shipped it; no independent evaluation exists. The claims circulating are marketing.
4. **Session/window management is almost unstudied** relative to tabs. Weinreich 2008's "tabs for related subtasks, windows for parallel tasks" is still the state of the art on the window layer, from 2005–2006 data.
5. **Cross-device tab state** — the same page open on phone, laptop and tablet — has no empirical literature at all.
6. **Nobody has replicated the Abrams filing-strategy proportions on tabs.** Applying his four-way instrument (no-filer / creation-time / end-of-session / sporadic) to tab-group creation would be a cheap, high-value study and would establish whether the 26/23/7/44 split transfers.

---

## References

**Browser tabs and sessions**
- Chang, J. C., Hahn, N., Kim, Y., Coupland, J., Breneisen, B., Kim, H. S., Hwong, J., & Kittur, A. (2021). When the tab comes due: Challenges in the cost structure of browser tab usage. *CHI '21*, Article 100. https://doi.org/10.1145/3411764.3445585
- Chang, J. C., Kim, Y., Miller, V., Liu, M. X., Myers, B. A., & Kittur, A. (2021). Tabs.do: Task-centric browser tab management. *UIST '21*, 663–676. https://doi.org/10.1145/3472749.3474777
- Dubroy, P., & Balakrishnan, R. (2010). A study of tabbed browsing among Mozilla Firefox users. *CHI '10*, 673–682. https://doi.org/10.1145/1753326.1753426
- Hisatomi, M., & Tajima, K. (2025). Graph-structure-based grouping of web browser tabs into tasks. *UIST '25 Adjunct*. https://doi.org/10.1145/3746058.3758439
- Huang, J., Lin, T., & White, R. W. (2012). No search result left behind: Branching behavior with browser tabs. *WSDM '12*, 203–212. https://doi.org/10.1145/2124295.2124322
- Huang, J., & White, R. W. (2010). Parallel browsing behavior on the web. *HT '10*, 13–18. https://doi.org/10.1145/1810617.1810622
- Ma, R., Lassila, H., Nurgalieva, L., & Lindqvist, J. (2023). When browsing gets cluttered: Exploring and modeling interactions of browsing clutter, browsing habits, and coping. *CHI '23*. https://doi.org/10.1145/3544548.3580690
- Rutishauser, R. A., & Fritz, T. (2026). From tabs to structures: Understanding and supporting web page management. *CHI '26*. https://doi.org/10.1145/3772318.3791979

**Bookmarks and keeping**
- Abrams, D., Baecker, R., & Chignell, M. (1998). Information archiving with bookmarks: Personal web space construction and organization. *CHI '98*, 41–48. https://doi.org/10.1145/274644.274651
- Bergman, O., Whittaker, S., & Schooler, J. (2021). Out of sight and out of mind: Bookmarks are created but not used. *Journal of Librarianship and Information Science*, 53(2), 338–348. https://doi.org/10.1177/0961000620949652
- Bruce, H., Jones, W., & Dumais, S. (2004). Keeping and re-finding information on the web: What do people do and what do they need? *Proceedings of ASIST*, 41(1), 129–137. https://doi.org/10.1002/meet.1450410115
- Jones, W. (2007). *Keeping found things found: The study and practice of personal information management.* Morgan Kaufmann.
- Jones, W., Bruce, H., & Dumais, S. (2001). Keeping found things found on the web. *CIKM '01*, 119–126. https://doi.org/10.1145/502585.502607
- Jones, W., Dumais, S., & Bruce, H. (2002). Once found, what then? A study of "keeping" behaviors in the personal use of web information. *Proceedings of ASIST*, 39(1), 391–402. https://doi.org/10.1002/meet.1450390143

**Revisitation and re-finding**
- Adar, E., Teevan, J., & Dumais, S. T. (2008). Large scale analysis of web revisitation patterns. *CHI '08*, 1197–1206. https://doi.org/10.1145/1357054.1357241
- Aula, A., Jhaveri, N., & Käki, M. (2005). Information search and re-access strategies of experienced web users. *WWW '05*, 583–592. https://doi.org/10.1145/1060745.1060831
- Aula, A., & Käki, M. (2005). Hundreds of folders or one ugly pile — Strategies for information search and re-access. *INTERACT 2005*, LNCS 3585. https://doi.org/10.1007/11555261_78
- Catledge, L. D., & Pitkow, J. E. (1995). Characterizing browsing strategies in the World-Wide Web. *Computer Networks and ISDN Systems*, 27(6), 1065–1073.
- Cockburn, A., & McKenzie, B. (2001). What do web users do? An empirical analysis of web use. *International Journal of Human-Computer Studies*, 54(6), 903–922. https://doi.org/10.1006/ijhc.2000.0459
- Obendorf, H., Weinreich, H., Herder, E., & Mayer, M. (2007). Web page revisitation revisited: Implications of a long-term click-stream study of browser usage. *CHI '07*, 597–606. https://doi.org/10.1145/1240624.1240719
- Tauscher, L., & Greenberg, S. (1997). How people revisit web pages: Empirical findings and implications for the design of history systems. *International Journal of Human-Computer Studies*, 47(1), 97–137. https://doi.org/10.1006/ijhc.1997.0125
- Teevan, J., Adar, E., Jones, R., & Potts, M. A. S. (2007). Information re-retrieval: Repeat queries in Yahoo's logs. *SIGIR '07*, 151–158. https://doi.org/10.1145/1277741.1277770
- Weinreich, H., Obendorf, H., Herder, E., & Mayer, M. (2008). Not quite the average: An empirical study of web use. *ACM Transactions on the Web*, 2(1), Article 5. https://doi.org/10.1145/1326561.1326566

**Personal information management, filing and piling**
- Bälter, O. (1998). *Electronic mail in a working context* (Doctoral thesis). Royal Institute of Technology (KTH), Stockholm.
- Boardman, R. (2004). *Improving tool support for personal information management* (PhD thesis). Imperial College London.
- Boardman, R., & Sasse, M. A. (2004). "Stuff goes into the computer and doesn't come out": A cross-tool study of personal information management. *CHI '04*, 583–590. https://doi.org/10.1145/985692.985766
- Fisher, D., Brush, A. J., Gleave, E., & Smith, M. A. (2006). Revisiting Whittaker & Sidner's "email overload" ten years later. *CSCW '06*, 309–312. https://doi.org/10.1145/1180875.1180922
- Gwizdka, J. (2004). Email task management styles: The cleaners and the keepers. *CHI '04 Extended Abstracts*, 1235–1238. https://doi.org/10.1145/985921.986032
- Kalman, Y. M., & Ravid, G. (2015). Filing, piling, and everything in between: The dynamics of e-mail inbox management. *JASIST*, 66(12), 2540–2552. https://doi.org/10.1002/asi.23337
- Kaye, J., Vertesi, J., Avery, S., Dafoe, A., David, S., Onaga, L., Rosero, I., & Pinch, T. (2006). To have and to hold: Exploring the personal archive. *CHI '06*, 275–284. https://doi.org/10.1145/1124772.1124814
- Malone, T. W. (1983). How do people organize their desks? Implications for the design of office information systems. *ACM Transactions on Office Information Systems*, 1(1), 99–112. https://doi.org/10.1145/357423.357430
- Vitale, F., Janzen, I., & McGrenere, J. (2018). Hoarding and minimalism: Tendencies in digital data preservation. *CHI '18*, Paper 587. https://doi.org/10.1145/3173574.3174161
- Whittaker, S., & Sidner, C. (1996). Email overload: Exploring personal information management of email. *CHI '96*, 276–283. https://doi.org/10.1145/238386.238530

**Industry sources**
- Mozilla. (2025). *Under the hood: How Firefox suggests tab groups with local AI.* https://blog.mozilla.org/en/firefox/ai-tab-groups/
- Mozilla. (2025). *Tackle tab overload with Firefox Tab Groups.* https://blog.mozilla.org/en/firefox/firefox-tab-groups/
- Carnegie Mellon University. (2021, May). *Overcoming tab overload.* https://www.cmu.edu/news/stories/archives/2021/may/overcoming-tab-overload.html
- Aalto University. (2023, April). *One in four internet users are overwhelmed by the clutter in their browser.*

**Source-access notes.** ACM DL blocks automated retrieval (Cloudflare). CHI '21 full text was read from the Internet Archive snapshot of the ACM full-HTML page (2022-02-03). Abrams 1998, Obendorf 2007, Weinreich 2008, Cockburn 2001, Dubroy 2010, Vitale 2018, Teevan 2007 and Adar 2008 were read from author- or institution-hosted PDFs. Aula & Jhaveri 2005 uses a non-extractable font encoding; its findings here come from the publisher abstract and the INTERACT companion paper. CHI '23 and CHI '26 findings come from publisher/repository abstracts plus the Aalto press release.
