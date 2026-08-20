---
sketch: 001
name: window-layout
question: "Where does the unified tab strip live, and what does one window of Power Browser look like?"
winner: null
tags: [layout, tab-strip, chrome]
---

# Sketch 001: Window Layout

## Design Question
The single highest-risk visual decision: where the unified (web + IDE) tab
strip lives. Everything else in the shell hangs off this.

## How to View
open .planning/sketches/001-window-layout/index.html

## Variants
- **A: Horizontal strip** — classic browser row; kind shown by accent color
  (blue = web, teal = IDE). Path of least resistance for a Firefox fork.
- **B: Vertical sidebar** — Zen/Arc lineage; tabs grouped by kind with room
  for full titles and future workspaces.
- **C: Command deck** — no persistent strip; one bar, URL bar doubles as a
  Ctrl-K switcher over all tabs (everything-is-a-URL taken literally).

## What to Look For
- Does an IDE tab sitting next to a YouTube tab read as *normal* or jarring?
- The dirty-dot + vetoed close on `power.xhtml` — the optimistic-close
  reconcile path made visible (try closing it).
- URL bar switching between `https://` and `power://` schemes per tab kind.
- In B: does kind-grouping (WEB / IDE sections) help or fight the "one strip,
  drag anywhere" thesis?
- In C: is losing always-visible tabs worth the calm? (Ctrl-K works.)

All three render from the same shared TabModel array — deliberately, since
that mirrors the real architecture (strip renders from the chrome-side mirror).
