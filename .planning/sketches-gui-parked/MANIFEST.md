# Sketch Manifest

> **PARKED (2026-08-19):** GUI discussion is deferred until after v1.0.
> v1.0 GUI = default Theia, rebranded only. Nothing here is a decision.

## Design Direction
Developer-dark, dense, keyboard-first browser chrome. Web and IDE tabs are
peers in one strip; tab kind is signaled by accent color (blue = web,
teal = IDE) and URL scheme (`https://` vs `power://`). Chrome must feel like
a fast browser, not an IDE with a web view.

## Reference Points
Firefox (the substrate), Zen Browser / Arc (vertical-tab lineage and fork
templates), VS Code / Theia (IDE interior), Ctrl-K command palettes
(everything-is-a-URL research, docs/research/tab-management-research/).

## Sketches

| # | Name | Design Question | Winner | Tags |
|---|------|----------------|--------|------|
| 001 | window-layout | Where does the unified tab strip live? | — | layout, tab-strip, chrome |
