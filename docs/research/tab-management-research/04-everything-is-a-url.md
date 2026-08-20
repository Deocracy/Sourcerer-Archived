# Everything Is a URL/Resource: Unified Navigation Systems Across Heterogeneous Tab Managers

Research pass on systems where a local file, a folder, a remote connection, and a web page are
all addressable objects inside one tab/navigation system. Goal: extract which management
paradigms survive contact with heterogeneous resource types, and which break.

---

## 1. Eclipse Theia and VS Code — the URI-as-universal-handle model

### The core abstraction

Both editors use a `URI` value type (scheme + authority + path + query + fragment) as the single
addressable handle for *everything* opened in the workbench — not just files. Schemes observed
in the wild:

- `file:` — real filesystem paths
- `untitled:` — an in-memory buffer with no backing file yet; VS Code derives the language mode
  from the *name* given to the untitled buffer even before it's saved
- `vscode-webview:` / `webview-panel:` — custom UI panels (extension-authored HTML/JS) that are
  still first-class "editors" in the tab strip even though there's no document behind them
- `vscode-remote:` — SSH/Codespaces/WSL remote resources, made to look identical to `file:` at
  the UI layer
- `git:`, `output:`, `debug:`, extension-defined custom schemes — diff views, output channels,
  and debug consoles are all URIs too

The unifying move: an `EditorInput` (VS Code) or `WidgetOpenHandler` (Theia) wraps *any* URI and
exposes a common contract (label, icon, dirty state, save/revert) to the workbench shell. The
shell never needs to know whether it's rendering a text buffer, a webview, or a diff — it just
asks the input/handler for a title and a widget.

- Theia's `OpenerService.open(uri, options)` walks a registry of `OpenHandler`s and picks the one
  that claims the URI (scheme + sometimes content sniffing); `WidgetOpenHandler` is the base class
  extensions subclass to bind a new scheme to a new widget type.
  ([opener-service.ts](https://github.com/eclipse-theia/theia/blob/master/packages/core/src/browser/opener-service.ts),
  [widget-open-handler.ts](https://github.com/eclipse-theia/theia/blob/master/packages/core/src/browser/widget-open-handler.ts))
- Theia explicitly does **not** distinguish "editor" from "view" at the shell level — anything
  can dock in the main area or side panels; the editor/view split is a convention, not an
  architectural wall. ([DeepWiki: Workspace and File Navigation](https://deepwiki.com/eclipse-theia/theia/3.5-workspace-and-file-navigation))
- VS Code's editor state machine: `IEditorService` (what's open, "open this URI") is layered over
  `IEditorGroupsService` (the split-pane layout of groups), and each group holds an
  `EditorGroupModel` tracking MRU order, pinned state, and sticky (pinned-at-start) editors.
  Source: `src/vs/workbench/services/editor/common/editorGroupsService.ts`,
  `src/vs/workbench/common/editor/editorGroupModel.ts`,
  `src/vs/workbench/browser/parts/editor/editorGroupView.ts`.
  ([DeepWiki: Layout System and Editor Management](https://deepwiki.com/microsoft/vscode/3.2-editor-features-and-contributions))

### Preview/pinned tabs — the mechanism that keeps a URI-addressable tab strip from exploding

This is the single most exportable idea from this pair of editors:

- **Single-click** (or "goto definition", or clicking a search result) opens the resource in a
  **preview tab** — reuses one italic, non-permanent tab slot, and critically, is **not added to
  the "Open Editors" list**.
- **Double-click**, or starting to type, **promotes** that tab to a permanent one: a new dedicated
  tab is created, it drops out of italics, and it now appears in Open Editors.
- Controlled by `workbench.editor.enablePreview`.
  ([nicoespeon: Why files open over existing tabs](https://www.nicoespeon.com/en/2020/01/change-vscode-preview-files/),
  [VS Code docs: User interface](https://code.visualstudio.com/docs/editing/userinterface))

This solves the exact problem heterogeneous-resource tab systems have: browsing (clicking through
search results, stack traces, a file tree) generates *many* transient opens, but only a few of
them are ever edited. Without a preview slot, every glance becomes a permanent tab. Browsers have
no equivalent primitive — a bookmark click or link click always creates a *durable* tab (see §8).

### Open Editors view

A dedicated tree, separate from the file explorer, listing every URI currently open across every
group — the closest thing either editor has to Emacs's buffer list, but scoped to editor inputs
specifically (it doesn't list terminals, output panels, or arbitrary views).

---

## 2. KDE Dolphin / KIO — one tab system, many transport protocols

KIO (KDE Input/Output, formerly "KIO slaves," now "KIO workers" since KDE Frameworks 6) is a
protocol-abstraction framework: every Dolphin location bar entry is a URL, and a plugin ("worker")
resolves the scheme to actual I/O.

- Confirmed backends: `file:`, `sftp:`/`fish:` (SSH), `smb:` (Windows shares), `webdav:`, `ftp:`,
  `trash:`, `mtp:` (Android/phone media transfer), `tar:`/`zip:` (browse inside archives as if
  they were folders), `man:`/`info:` (documentation as a browsable filesystem).
- Every KDE app that uses the standard file-open dialog inherits *all* of these transports for
  free — this isn't Dolphin-specific, it's a shared framework layer, so Kate, GIMP-via-KDE-integration,
  etc. all get sftp/mtp/trash access through the same URL scheme.
- Dolphin's tabs are literally a stack of independent navigation histories over
  arbitrary KIO URLs — a tab can point at `sftp://host/path` in one and `trash:/` in the
  next, and drag-and-drop between tabs invokes KIO's copy/move engine, which itself is
  scheme-aware (e.g. it knows `trash:` restore means "move back to origin," not a byte copy).
  ([KIO — Grokipedia](https://grokipedia.com/page/KIO), [KIO Wikipedia](https://en.wikipedia.org/wiki/KIO))

Compare to GNOME's equivalent, **GVFS**: same idea (SFTP, SMB, WebDAV, MTP/AFC-for-iPhones, trash,
archives — all mounted as URI-addressable virtual filesystems under `gvfsd`), consumed by Nautilus
(GNOME Files) tabs the same way. GVFS backends run as separate daemons per protocol
(`gvfsd-trash`, `gvfsd-sftp`, ...), a more service-oriented design than KIO's in-process worker
model, but the user-facing result — one tab strip, any protocol, uniform breadcrumb/URL bar — is
identical. ([Unix Memo: GVFS](https://unix-memo.readthedocs.io/en/latest/vfs.html), [ArchWiki: GNOME/Files](https://wiki.archlinux.org/title/GNOME/Files))

**What survives:** the URL-bar-as-command-line pattern. Because every resource *has* a URL, you
can type `sftp://host/etc` into the same bar you'd type `/home/user`, and the file manager doesn't
need special-case UI for "connect to remote server" — it's the same "go to location" action.

---

## 3. Windows File Explorer — tabs arrived late, but the "everything is a folder" idea is decades old

Two separate points here:

- **Shell namespace (1990s–present):** Windows Explorer's own resource model long predates its
  tabs. The Shell Namespace is a single tree where ordinary directories and **virtual folders**
  (My Computer, Control Panel, Recycle Bin, Printers, Network Neighborhood) are both just
  "folders" — Control Panel *items* are literally addressable as `shell:::{CLSID}` junctions, and
  third-party apps can inject "namespace extensions" that make an arbitrary data source (an FTP
  site, a database) appear as a browsable folder tree next to real directories.
  ([MS Learn: Introduction to the Shell Namespace](https://learn.microsoft.com/en-us/windows/win32/shell/namespace-intro),
  [Windows Shell namespace — Wikipedia](https://en.wikipedia.org/wiki/Windows_Shell_namespace))
  This is Windows's version of "everything is a file" — except the leaf/container uniformity is
  achieved by making *everything a folder* rather than a stream.
- **Tabs (2022):** actual tabbed browsing inside `explorer.exe` didn't ship until the Windows 11
  22H2 update (rolled out October 2022), after years of user requests and third-party tools
  (Clover, QTTabBar) filling the gap. Once shipped it was a straightforward browser-style tab
  strip (`Ctrl+T`, drag files between tabs) layered on top of the decades-old namespace model —
  notably it did **not** add anything like preview/pinned-tab distinctions; every opened folder is
  a full permanent tab. ([TechCrunch, Sept 2022](https://techcrunch.com/2022/09/20/microsoft-rolls-out-windows-11-update-with-file-explorer-tabs-system-wide-captions-and-more/),
  [XDA: File Explorer tabs in 22H2](https://www.xda-developers.com/windows-11-22h2-file-explorer-tabs-available/))

**Lesson:** a rich, uniform resource model can exist for 25+ years with *zero* tab UI on top of
it — tabs and "everything is addressable" are orthogonal capabilities, not a package deal.

---

## 4. Emacs — the ur-example of heterogeneous-resource-as-one-namespace

Emacs predates URIs as a mainstream concept but solved the identical problem with **buffers**:

- A buffer is created for every visited file, but also for a shell process, a `Dired` directory
  listing, a Magit status view, a compilation log, or a TRAMP remote connection — there is no
  type distinction at the navigation layer. `switch-to-buffer` (`C-x b`) works identically
  regardless of what's behind the buffer.
- **TRAMP** (Transparent Remote Access, Multiple Protocols) makes a *remote* file — over SSH,
  `sudo`, Docker exec, FTP, etc. — open as an ordinary buffer with a special-syntax path
  (`/ssh:user@host:/path/to/file`), fully editable with all the same commands as a local file.
  This is functionally identical to KIO's `sftp://` and GVFS's SFTP backend, but resolved at the
  *buffer-naming* layer instead of a URL scheme.
- **Ibuffer** (`M-x ibuffer`) is the buffer-list-as-power-tool: sortable, filterable, markable,
  and — relevantly — groupable by predicate, including grouping all TRAMP buffers under their
  remote host string via `file-remote-p`. This is the most mature "list every open resource,
  faceted by type/origin" UI surveyed here.
  ([Weekly Emacs tip #16: ibuffer](https://www.polyomica.com/weekly-emacs-tip-16-managing-your-buffers-more-easily-with-ibuffer/))
- **tab-bar-mode** (Emacs 27+, 2020) is a comparatively late addition — a tab holds a *window
  configuration* (an arrangement of split panes each showing a buffer), not a single buffer.
  `tab-line-mode` is the closer analog to a browser tab strip, showing one tab per buffer visible
  in a window. Community packages (`bufler.el`, `tabspaces`, `bufferlo`) layer "workspace" grouping
  on top, echoing Arc's Spaces or VS Code's multi-root workspaces, decades after the buffer model
  itself was solved.
  ([bufler.el](https://github.com/alphapapa/bufler.el), [Mastering Emacs: Demystifying Emacs's Window Manager](https://www.masteringemacs.org/article/demystifying-emacs-window-manager))

**Lesson:** the resource-abstraction layer (buffers) and the tab/window layer were built ~40 years
apart. Buffers alone, with a good list view (Ibuffer) and a minibuffer completion UI, were usable
for decades with no "tabs" at all — reinforcing the Windows Explorer lesson from §3.

---

## 5. Acme / Plan 9 — the file-server-as-window-manager extreme

Plan 9's "everything is a file" is more literal than any other system surveyed:

- Acme (Rob Pike's editor/shell) mounts *itself* as a file server at `/mnt/acme` — every open
  window, its selection, its tag line, and its body are exposed as read/write files. External
  programs manipulate the editor by doing filesystem I/O against it, not by calling an API.
  ([Acme (text editor) — Wikipedia](https://en.wikipedia.org/wiki/Acme_(text_editor)))
- No tabs at all. Windows are laid out in resizable **columns**; navigation is spatial, not
  list-based — there's no "switcher," you look at the screen.
- **The plumber**: a system-wide daemon that inspects arbitrary text (a file path, a URL, an
  error message with a line number) and routes it to the right handler/window based on rules —
  click a file path in a compiler error and the plumber opens that file at that line in a new
  Acme window. This is a *content-addressed, rule-based* open handler, conceptually the ancestor
  of Theia/VS Code's `OpenerService` picking a handler by URI scheme — except the plumber
  operates on unstructured text, discovering the "URI" via pattern matching rather than requiring
  the source to already emit a proper URI.
  ([the plumber (Plan 9 paper)](https://doc.cat-v.org/plan_9/4th_edition/papers/plumb))

**Lesson:** Acme proves the tab strip / buffer list is not a *necessary* consequence of a unified
resource model — it's a UI choice for when the number of open things exceeds what fits on screen
as spatial columns. Every other system here made that trade differently.

---

## 6. Browsers and the death of `ftp://`

Browsers used to be more Acme-like than they are now — a single navigable space where `file://`,
`ftp://`, and `http://` were interchangeable in the same address bar and tab. That's mostly gone:

- Chrome: FTP document-fetching stripped in Chrome 72 (2019); FTP proxy support removed in
  Chrome 76; flag to disable entirely landed in Chrome 86 (late 2020, deprioritized briefly during
  the pandemic due to IT teams' workload); fully removed by default in Chrome 88 (Jan 2021); code
  deleted entirely in Chrome 95 (Oct 2021).
- Firefox: dropped FTP in Firefox 90 (July 2021). Mozilla had announced the intent back in 2015,
  citing the protocol's lack of encryption. Mozilla's own telemetry showed only ~0.32% of monthly
  active users (700k of 220M) ever opened an FTP link.
- Edge inherited the Chromium removal; Safari has been FTP-free for years.
- Reasoning given by both vendors: FTP is plaintext (credentials and data both), doesn't fit the
  HTTPS-everywhere security model, and had negligible usage.
  ([The Register: Firefox ends FTP support](https://www.theregister.com/2021/07/21/firefox_ends_ftp_support/),
  [The Register: Chrome strips FTP code](https://www.theregister.com/2021/10/20/ftp_chrome_95/),
  [How-To Geek](https://www.howtogeek.com/744569/chrome-and-firefox-killed-ftp-support-heres-an-easy-alternative/))

`file://` survives in every browser (for local HTML/PDF preview) but is now second-class: no
directory-listing polish, aggressively sandboxed (no XHR/fetch to other `file://` paths, no
cross-origin access), and increasingly gated behind explicit flags for extensions. The trend line
is the opposite of Theia/VS Code/KIO/Emacs: those systems widened the set of protocols a single
navigation surface treats uniformly; browsers narrowed it, security-first, down to effectively
`https:` plus a crippled `file:`.

**Lesson:** protocol unification is not a one-way ratchet. When a scheme becomes a security
liability (plaintext creds) and usage is negligible, vendors will tear it back out of the unified
model rather than sandbox it — even though the "everything is a URL" abstraction had no technical
problem accommodating it.

---

## 7. Modern unified-workspace apps

### Arc (and its Spaces model)

- Replaces the horizontal tab strip with a vertical **sidebar**: Spaces (named, color-coded
  containers, each with its own pinned-tab section and its own tab-archive bucket) sit above
  Pinned Tabs (which behave like a cross between a bookmark and a site-locked app — clicking an
  internal link navigates in place, clicking an external link opens a "Peek" child window instead
  of hijacking the pin) and ordinary ephemeral tabs, which **auto-archive** after a timeout.
  ([Arc Help Center: Pinned Tabs](https://resources.arc.net/hc/en-us/articles/19231060187159-Pinned-Tabs-Tabs-you-want-to-stick-around),
  [SupaSidebar: What Is Arc's Sidebar](https://supasidebar.com/blog/what-is-arc-browser-sidebar-2026))
- Effectively: Spaces = Emacs's would-be "workspace" packages / VS Code's multi-root workspaces;
  Pinned Tabs = VS Code's sticky/pinned editors; auto-archived tabs = the preview-tab GC that
  Theia/VS Code do explicitly, done here by a timer instead of an explicit promote/demote action.
- Note: Arc's own team pivoted toward a new browser ("Dia") in 2025-2026, but the sidebar/Spaces
  vocabulary it popularized has been widely cloned (see `Arcify`, `Side Bar for Arc Users` browser
  extensions in the search results above) — it's now a genre, not one product.

### Vivaldi and Opera — browser as an all-in-one shell for non-web resources

- **Vivaldi**: ships a real IMAP/SMTP mail client (Vivaldi Mail), calendar, feed reader, notes,
  and to-do list, all as **Panels** — a second, independent tab-like strip pinned to the sidebar
  that doesn't consume main tab-bar space and persists across tab switches. Panels are explicitly
  positioned as "tabs that are always there," a second tier below the main tab strip rather than
  merged into it. ([Vivaldi: A deep dive into panels](https://vivaldi.com/blog/a-deep-dive-into-panels/),
  [Vivaldi Mail announcement](https://vivaldi.com/blog/vivaldi-mail-calendar-feed-reader-are-here/))
- **Opera**: pioneered this pattern decades earlier with the bundled M2 mail/newsgroup/IRC client
  (a "search-driven," unified-inbox client presented inside the same suite as the browser, long
  before Vivaldi existed) and now Opera/Opera GX ship **Workspaces** — named tab-group containers
  at the top of the sidebar, switchable, closer to Arc's Spaces than to Vivaldi's Panels.
  ([Opera Mail overview](https://appmus.com/software/opera-mail), [Opera Workspaces](https://www.opera.com/features/workspaces))
- Neither Vivaldi's Panels nor Opera's M2 make mail *URL-addressable* the way KIO/GVFS make a
  remote file addressable — mail lives in its own panel/app pane with its own navigation model,
  bolted alongside the tab strip rather than unified into it. This is the opposite design choice
  from Theia (where a webview and a file both become one URI-scoped "editor") and is worth
  flagging as a real architectural fork, not just a styling difference.

### Rambox / Ferdium (formerly Franz) — apps-as-tabs, no shared resource model at all

- These wrap dozens of independent web apps (Slack, WhatsApp, Gmail, Discord, Notion, Teams...)
  each in its own embedded browser instance (historically separate Electron webviews), presented
  as a tab-like sidebar list. ([Hongkiat: Unified Messaging Apps compared](https://www.hongkiat.com/blog/best-unified-messaging-apps/))
- Critically, this is *container-level* unification only — there is no shared URI space, no
  drag-and-drop of content between "tabs," no unified search across them. Each pane is an
  isolated web app; the "unification" is purely "one taskbar icon instead of six." This is the
  weakest form of "everything in one tab system" surveyed — a useful negative example: sticking
  heterogeneous apps in adjacent tabs isn't the same as giving them a shared addressing scheme.

### Notion / Obsidian — internal links as a navigation graph, not a tab system

- Obsidian's `[[wikilinks]]` create bidirectional links indexed by a metadata cache and rendered
  in a native Graph View (local graph = neighbors of current note, global graph = whole vault).
  Obsidian *tabs* exist (opened notes, pinnable, splittable into panes) layered on top of this
  link graph — so Obsidian actually combines a VS-Code-style tab strip with an Emacs/Acme-style
  content-addressed navigation graph.
- Notion's backlinks (`@mention` a page, or type `[[`) exist but there's no native graph
  visualization — third-party tools (NotionBL) bolt one on. Notion's own "tabs" (recently added,
  browser-tab-styled) are shallower: they track open pages, not a link graph.
  ([DeepWiki: Obsidian Internal Links and Backlinks](https://deepwiki.com/obsidianmd/obsidian-help/4.2-internal-links-and-backlinks),
  [Notion Backlinks explained](https://templates4notion.medium.com/notion-backlinks-explained-l-2024-tutorial-6a54fc18dac2))
- Relevant to the "everything is a URL" theme because a wikilink *is* a URI in miniature — a
  scheme-less relative reference resolved against the vault namespace, exactly analogous to how
  `file:` paths resolve against a workspace root in Theia/VS Code.

---

## 8. The save/stash problem across heterogeneous resource types

Every system above eventually needs an answer to: *how do you keep a resource around after you
close the thing that's currently showing it?* The answers cluster into four incompatible
mechanisms, and no surveyed system unifies more than two of them:

| Mechanism | What it preserves | What it loses | Seen in |
|---|---|---|---|
| **Bookmark** | A URL/path only | Scroll position, unsaved edits, tab-group structure, session state | Browsers, Windows Explorer "Quick Access", KDE Places panel |
| **History/MRU** | A URL/path + timestamp, auto-recorded | Same as bookmarks, plus it's unmanaged and decays (Chrome: 90-day default retention) | Browsers, VS Code "recently opened", Emacs `recentf` |
| **Pinned/sticky tab** | The *live* editor/tab itself — state, scroll, undo history — as long as the process stays alive | Nothing, until the app closes without session-restore; pins alone don't survive a full quit in most browsers | VS Code sticky editors, Arc Pinned Tabs, browser pinned tabs |
| **Session/workspace snapshot** | The *whole* window/group/pane layout, serialized to disk | Nothing structural, but heavyweight — usually one blob per whole window, not per-resource | Browser "Restore previous session", VS Code workspace `.code-workspace` state, Emacs `desktop-save-mode`, `tabspaces`/`bufferlo` |

The editors (Theia/VS Code) are the only systems surveyed that make the *third* row durable by
default without user action: an editor group's open/pinned editors are serialized into workspace
storage automatically, so quitting and reopening the IDE restores exactly what was open — no
explicit "save session" gesture required. Browsers require either enabling "continue where you
left off" (which is really row 4, a periodic snapshot) or manual bookmarking (row 1, lossy).
Emacs requires an opt-in mode (`desktop-save-mode`) to get row 4; nothing is automatic by default.

**This is the sharpest Theia/VS Code-vs-browser divergence**: IDEs treat "what's open" as durable
application *state* that should always survive a restart; browsers treat "what's open" as
ephemeral *session* data that survival is a feature to opt into, because for decades the assumed
default was that a browser window is disposable and content lives at its permanent URL, not in
the tab.

---

## 9. Answer to the key question

**What survives when one tab system must hold heterogeneous resources, and what breaks?**

Survives:

1. **A single addressing primitive (URI/URL/buffer-name) that every open thing implements**, even
   if the thing behind it isn't a file at all (a webview, a shell process, a virtual folder, a
   remote SSH path). Every successful system here has exactly one such primitive. KIO/GVFS resolve
   it via scheme + worker plugin; Theia/VS Code via scheme + OpenHandler/EditorInput; Emacs via
   buffer name (with TRAMP encoding remote-ness *inside* the name string); Acme via the filesystem
   itself.
2. **A cheap "just looking" state distinct from a durable "I opened this" state** (VS Code/Theia
   preview tabs, Arc's auto-archive, Emacs's `find-file` vs. explicit buffer creation being nearly
   free either way because buffers are cheap). Without this, browsing behavior floods the tab
   strip — this is the single biggest practical failure mode when heterogeneous browsing
   (following links, jumping to definitions, opening search hits) shares a tab system with
   heterogeneous editing.
3. **A list/graph view of everything open, independent of the tab strip's spatial layout**
   (Open Editors, Ibuffer, Obsidian's graph) — necessary once the resource types are varied enough
   that "look at the tab strip" stops being sufficient triage.
4. **Uniform drag/copy/move semantics that are scheme-aware underneath** (KIO/GVFS moving a file
   from `trash:` back to `file:` does a restore, not a byte copy; dragging a tab between Arc Spaces
   just re-parents state) — the uniformity is at the *gesture* level, not forcing every backend to
   behave identically.

Breaks:

1. **Treating heterogeneous *apps* (not resources) as tabs with no shared addressing** (Rambox/
   Ferdium) buys almost nothing beyond taskbar consolidation — no cross-resource search, no
   drag-and-drop, no unified history. This is the most common shallow imitation of the pattern.
2. **Protocols that carry an unacceptable security cost inside a security-hardened container.**
   `ftp://` is the clean counter-example: browsers didn't struggle to keep the plaintext protocol
   inside the same URL-bar abstraction technically — they removed it because HTTPS-everywhere and
   sandboxing made the abstraction's uniformity actively dangerous for that one scheme.
3. **Assuming "open" implies "durable" without an explicit save gesture, in a domain where users
   don't expect that.** Browsers get this "wrong" relative to IDEs on purpose — a browser tab is
   assumed disposable because the resource (a web page) has a permanent home elsewhere; an IDE
   editor's "resource" (an in-progress edit, an untitled scratch buffer) often has *no* other home,
   so IDEs default to preserving tab state and browsers default to not.
4. **Bolting a second, parallel navigation surface next to the tab strip instead of into it**
   (Vivaldi Panels, Opera M2) works as a pragmatic ship-it move, but it means the "everything is
   addressable" promise silently stops at the panel boundary — you can't drag a mail message into
   a browser tab's history, search across panels and tabs in one box, or address a specific email
   with the same kind of URI a bookmark uses.

### What Theia/VS Code do that browsers don't (and vice versa)

Theia/VS Code:
- Make genuinely non-document things (webviews, diffs, debug consoles) first-class tab citizens
  under the same `EditorInput`/`WidgetOpenHandler` contract as a text file — no separate "app
  panel" tier.
- Distinguish preview-vs-pinned at the *individual tab* level as a deliberate, named feature, not
  an emergent side effect of an archiving timer.
- Persist "what's open" as durable state by default, no user action required.

Browsers (and browser-descended tools like Arc) do that Theia/VS Code don't:
- Give every open thing a genuinely portable, shareable, permanent external address (a URL you can
  paste into Slack) — a VS Code preview tab's "address" is meaningless outside that specific
  workspace/machine.
- Layer a *social* graph atop navigation (bookmarks bar you show other people, shared workspace
  links) — IDEs have nothing equivalent; a `.code-workspace` file is not something you paste into
  a chat the way a URL is.
- Solve the "too many opens" problem partly through spatial/temporal grouping *of the container*
  (Spaces, Workspaces, tab groups) rather than only at the individual-tab level, because browser
  users routinely have 50-200 open resources — an order of magnitude more than a typical IDE
  session — so container-level triage (Arc Spaces, Opera Workspaces) matters more there than it
  does for Theia/VS Code, where editor-group splits are usually single digits.

---

## Sources

- [VS Code docs: User interface (preview tabs)](https://code.visualstudio.com/docs/editing/userinterface)
- [nicoespeon: Why files open over existing tabs in VS Code](https://www.nicoespeon.com/en/2020/01/change-vscode-preview-files/)
- [DeepWiki: VS Code Layout System and Editor Management](https://deepwiki.com/microsoft/vscode/3.2-editor-features-and-contributions)
- [Theia opener-service.ts](https://github.com/eclipse-theia/theia/blob/master/packages/core/src/browser/opener-service.ts)
- [Theia widget-open-handler.ts](https://github.com/eclipse-theia/theia/blob/master/packages/core/src/browser/widget-open-handler.ts)
- [DeepWiki: Theia Workspace and File Navigation](https://deepwiki.com/eclipse-theia/theia/3.5-workspace-and-file-navigation)
- [KIO — Grokipedia](https://grokipedia.com/page/KIO)
- [KIO — Wikipedia](https://en.wikipedia.org/wiki/KIO)
- [ArchWiki: GNOME/Files](https://wiki.archlinux.org/title/GNOME/Files)
- [Unix Memo: GVFS](https://unix-memo.readthedocs.io/en/latest/vfs.html)
- [MS Learn: Introduction to the Shell Namespace](https://learn.microsoft.com/en-us/windows/win32/shell/namespace-intro)
- [Windows Shell namespace — Wikipedia](https://en.wikipedia.org/wiki/Windows_Shell_namespace)
- [TechCrunch: Windows 11 File Explorer tabs](https://techcrunch.com/2022/09/20/microsoft-rolls-out-windows-11-update-with-file-explorer-tabs-system-wide-captions-and-more/)
- [XDA: File Explorer tabs in 22H2](https://www.xda-developers.com/windows-11-22h2-file-explorer-tabs-available/)
- [Weekly Emacs tip #16: ibuffer](https://www.polyomica.com/weekly-emacs-tip-16-managing-your-buffers-more-easily-with-ibuffer/)
- [Mastering Emacs: Demystifying Emacs's Window Manager](https://www.masteringemacs.org/article/demystifying-emacs-window-manager)
- [bufler.el](https://github.com/alphapapa/bufler.el)
- [Acme (text editor) — Wikipedia](https://en.wikipedia.org/wiki/Acme_(text_editor))
- [the plumber (Plan 9 paper)](https://doc.cat-v.org/plan_9/4th_edition/papers/plumb)
- [The Register: Firefox ends FTP support](https://www.theregister.com/2021/07/21/firefox_ends_ftp_support/)
- [The Register: Google strips FTP code from Chrome](https://www.theregister.com/2021/10/20/ftp_chrome_95/)
- [How-To Geek: Chrome and Firefox killed FTP](https://www.howtogeek.com/744569/chrome-and-firefox-killed-ftp-support-heres-an-easy-alternative/)
- [Arc Help Center: Pinned Tabs](https://resources.arc.net/hc/en-us/articles/19231060187159-Pinned-Tabs-Tabs-you-want-to-stick-around)
- [SupaSidebar: What Is Arc's Sidebar](https://supasidebar.com/blog/what-is-arc-browser-sidebar-2026)
- [Vivaldi: A deep dive into panels](https://vivaldi.com/blog/a-deep-dive-into-panels/)
- [Vivaldi Mail/Calendar/Feed Reader announcement](https://vivaldi.com/blog/vivaldi-mail-calendar-feed-reader-are-here/)
- [Opera Mail overview](https://appmus.com/software/opera-mail)
- [Opera Workspaces](https://www.opera.com/features/workspaces)
- [Hongkiat: Unified Messaging Apps compared (Rambox/Franz/Ferdium)](https://www.hongkiat.com/blog/best-unified-messaging-apps/)
- [DeepWiki: Obsidian Internal Links and Backlinks](https://deepwiki.com/obsidianmd/obsidian-help/4.2-internal-links-and-backlinks)
- [Notion Backlinks explained](https://templates4notion.medium.com/notion-backlinks-explained-l-2024-tutorial-6a54fc18dac2)
