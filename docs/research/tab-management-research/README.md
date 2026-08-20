# Tab Management Research — Index

All tab-management findings for the Power Browser project. Read 00 first; it
synthesizes 01–04. 05 is the implementation side (Theia mechanics).

| File | What it is |
|---|---|
| `00-SYNTHESIS-chart.md` | The synthesis: paradigm chart, user segmentation, customization matrix, implications for the unified everything-is-a-URL tab system, unverified-figures list |
| `01-paradigm-history-taxonomy.md` | Full history and taxonomy of tab/navigation paradigms |
| `02-academic-studies.md` | Academic and industry research on tab, bookmark, session, and saved-page management (Dubroy & Balakrishnan 2010, Chang et al. CHI 2021, etc.) |
| `03-user-customization-preferences.md` | What users actually install and complain about — OneTab, Tree Style Tab, Sidebery, Session Buddy, Toby, Workona, Tabs Outliner |
| `04-everything-is-a-url.md` | Unified navigation systems across heterogeneous tab managers; the URI-as-universal-handle model in Theia/VS Code |
| `05-theia-shell-customization.md` | Implementation: Theia URL bar mechanics, tab↔URI mapping (`Navigatable`), chrome rebuild seams, build cost, session decisions D1–D4 |

Related notes one level up (`notes/`):

- `power-browser-architecture.md` — where the tab system lands in the overall Firefox-fork + Theia design (unified tab strip, TabModel, bridge)
- `extension-model-decisions.md` — plugin API decisions that constrain what extensions can do to tabs
- `firefox-fork-browser-ide-research.md` — why Firefox fork over Electron
