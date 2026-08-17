# Assistant Framework Model & Requirements — Design (approved 2026-08-11)

**Status:** APPROVED (user, 2026-08-11). Produced by spike 006; consumed by the
ingestion study and the next assistant milestone's requirements/roadmap work.

**Standing decisions from the brainstorm:**

- Pi and the current Assistant GUI both get **rebuilt/replaced**; the interactive-GUI
  behaviour survives as a **skill** (D-2), not as bespoke panel code.
- The harness choice (embedded vs vendor CLIs vs ACP) is **deliberately deferred** —
  the layered model defers it by construction; the ingestion study decides it with evidence.
- The entire multi-coding-agent stack may be remade; nothing below assumes any
  competitor's runtime, only their proven shapes.

## Governing thesis: Test-Driven Research

**Coding ⊂ Test-Driven Research.** Built for the Scholar of Tomorrow: code is needed at
every level; the most advanced RAG and Deep Research are core, not adjacent. TDD's loop
generalizes — a claim is a failing test, evidence gathering is implementation, the
evidence chain is the contract, re-verification is CI, and an unverified claim blocks
publication the way a red build blocks merge. One verification machine serves both.

## The framework model — four layers

```
Claim    — durable research record. statement · status (unevidenced/supported/
           refuted/stale) · evidence chain · deterministic verifier · DAG of
           dependencies. Outlives every agent. Re-verifies when sources change.
           Red blocks publication.
  ▲ evidenced by
Task     — unit of isolated work. goal · isolated workspace · budget · status ·
           reviewable artifact (diff | document | report). Maps to the Kanban applet.
  ▲ executed by
Session  — transient. one agent · one context · one cost-ledger row per turn ·
           fork/resume/rewind. Disposable by design.
  ▲ runs on
Harness  — swappable, capability-declared. Pi today; replacement chosen by the
           ingestion study. Capability is data, not code — UI degrades per flag.
```

`Evidence` is one type with many kinds: cited passage · code + exit code · reproduced
figure · dataset digest. Each layer has its own lifetime and store; each is swappable
without touching the layer above.

## Requirements

### A. Control plane (the demand-side winner — not more parallelism)

| ID | P | Requirement |
|----|---|-------------|
| A-1 | P0 | **Cost ledger.** One normalized usage row per turn (harness, model, session, task, tokens in/out/cache, cost). Per-task budgets. A user-global cost fuse. Nothing ships that can spend without showing. |
| A-2 | P0 | **Policy boundary.** A runtime layer between agent reasoning and the shell — allow/deny/ask evaluated in the runtime, never in the prompt (immune to compaction "governance decay"). Safe path = fast path: sandboxed workspace where most operations need no prompt. |
| A-3 | P1 | **Session lock.** Nix-style lockfile per run: model version, system prompt, skills, MCP server versions, tool closure. Reproducible research runs — a differentiator nobody in the survey has. |

### B. TDR engine

| ID | P | Requirement |
|----|---|-------------|
| B-1 | P0 | **Claim store with re-verification.** Claims-as-CI: source/corpus changes mark dependent claims stale; deterministic evidence gates; unverified claims block export/publication. |
| B-2 | P0 | **One verifier loop.** plan → execute → independently verify, with machine-readable evidence (zeroshot's command/exitCode/output contract), identical for code and prose. |
| B-3 | P1 | **Deep Research as a Task type.** Fan-out readers → cross-checked cited synthesis; every claim enters the store through B-1. |

### C. Execution substrate

| ID | P | Requirement |
|----|---|-------------|
| C-1 | P0 | **Daemon/client split.** The substrate-side harness owns sessions; the Tauri window is a client. Closing the window kills nothing. (Aligned with Phase 14 relocation; `host.ai()` seam unchanged applet-side.) |
| C-2 | P1 | **Workspace isolation.** Worktree-per-task default; Landlock/bubblewrap tier for untrusted work — native to the substrate already. |
| C-3 | P2 | **Op-log undo.** jj-style operation log: "undo the agent's whole turn" for documents and code both. |

### D. Interop (parsers and one protocol — days, not quarters)

| ID | P | Requirement |
|----|---|-------------|
| D-1 | P0 | **MCP client/host**, supporting both the 2025-11-25 and 2026-07-28 spec revisions through the deprecation straddle. stdio + Streamable HTTP; OAuth 2.1 + PKCE. Never build on Sampling/Roots/Logging (deprecated) or session IDs (removed). |
| D-2 | P0 | **AGENTS.md + SKILL.md.** Read-and-prepend + frontmatter parser. The Assistant's interactive-GUI behaviour is delivered as a skill here. |
| D-3 | P1 | **Agent Plugins 1.0.** plugin.json + skills/ + its portable mcp.json (kills the per-client MCP-config translation problem). |
| D-4 | P2 | **ACP** — deferred until the harness rebuild decision; version-negotiated v1-default when it lands. |

### E. Anti-abandonment invariants (constraints on everything, not features)

Local-first · no required server · full state export · single self-contained install.
These are the 2026 purchase criteria the abandoned incumbents (Bloop, Crystal,
vibe-kanban) violated.

### Explicitly out of scope

Swarms (measured 9× wall time, 2× cost, +8% quality) · A2A · ai-catalog.json/ARD ·
per-vendor CLI adapter tables (until the ingestion study says otherwise) ·
OTel gen_ai.* attribute names in the product surface (emit, don't surface).

## Next step

**Ingestion study:** mine the seven indexed competitor repos
(`../Agnostic agentic coding tools-CLI for Sourcerer/competitors/`, all in the
codebase-memory graph) against requirements A-1…D-4 to decide build-vs-copy per
requirement — licenses allow lifting from herdr/vibe-kanban/crystal/zeroshot
(Apache-2.0/MIT), never cmux (GPL). The harness decision then falls out of C-1 + D-4
with evidence instead of taste.
