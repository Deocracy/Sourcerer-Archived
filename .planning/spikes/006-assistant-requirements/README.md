---
spike: 006
name: assistant-requirements
type: standard
validates: "Given the Scholar-of-Tomorrow thesis, when the agentic-tooling landscape is torn down and the demand side researched, then a ranked feature requirement set for the Dashboard Assistant exists with a defensible v-next cut"
verdict: VALIDATED
related: [001, 002, 003, 004, 005]
tags: [requirements, assistant, tdr, rag, deep-research, agnostic, brainstorm]
---

# Spike 006: Dashboard Assistant — Feature Requirements

## The Thesis (noted properly, per user 2026-08-11)

> **Coding ⊂ Test-Driven Research.**
> A scholar for today needs to be able to code. The Assistant is *all* of code and more.
> We are building for the **Scholar of Tomorrow** — which is why the most advanced RAG and
> Deep Research are core, not adjacent. **Code is needed at every level.**

This is the governing frame for every requirement below, and it resolves the question that
blocked this spike: whether coding-agent orchestration is even relevant to a scholar's
workbench. It is — but not as a separate "coding mode."

**Test-Driven Research (TDR)** is the generalization of TDD:

| TDD | TDR |
|---|---|
| Write a failing test | State a claim that is not yet evidenced |
| Implement until green | Gather evidence until the claim is supported *or* refuted |
| The test is the contract | The evidence chain is the contract |
| CI re-runs tests on change | Re-verification when sources, corpus, or claim change |
| Red build blocks merge | Unverified claim blocks publication |
| Refactor safely under green | Rewrite prose safely under a stable evidence chain |

The consequence for architecture: the executor→verifier loop that coding orchestrators
use (zeroshot's plan → implement → *independently validate*, with a deterministic
evidence gate rather than an LLM's opinion) is **the same machine** as claim → evidence →
verify. Sourcerer should build one loop and use it for both. Code is one kind of evidence;
a cited passage is another; a reproduced figure is a third.

**What this rules in that a "research copilot" framing would have ruled out:** sandboxing,
worktree-style isolation, deterministic verification gates, per-turn cost accounting,
multi-vendor agent execution, session fork/rewind, reproducible-run locking.

## Research

11 agents. 7 competitor repos cloned into
`../Agnostic agentic coding tools-CLI for Sourcerer/competitors/` and indexed into the
codebase-memory knowledge graph; 4 discovery sweeps (adjacent categories, steal-from-outside,
user pain, standards). Full agent output in the session workflow journal.

### Competitor teardown — corrections to the public picture

| Repo | Reality found in source | License | Status for us |
|---|---|---|---|
| **herdr** | Rust background server owning PTYs; agent state via hot-reloadable TOML **screen-scrape manifests**; ~90-method JSON socket API with published schema | Apache-2.0 (relicensed from AGPL in 0.8.0) | **Closest architectural sibling.** Rust, daemon+client split |
| **cmux** | **Pivoted** — now a native macOS Swift/libghostty terminal emulator, 8,299 Swift files. **No isolation model at all**; worktrees are a dead 229-line prototype | GPL-3.0 + carve-out | **Copy designs, never code** (GPL blocks PolyForm NC) |
| **vibe-kanban** | Rust/Axum + React, 9 vendor CLIs as **piped subprocesses** in per-workspace worktrees, one executor trait, ACP for 3 of them | Apache-2.0 | **Sunsetting** → free reference implementation |
| **claude-flow** | Claude-Code-centric, not agnostic (Codex = one `codex exec` branch). "Queen hive-mind swarm" is a generated prompt + in-process EventEmitter | MIT | Swarm is theater; the **cost fuse** is real |
| **crystal** | Electron, worktree-isolated parallel sessions, **execution-scoped diff timeline** | MIT | **Deprecated Feb 2026** → frozen design spec |
| **zeroshot** | Executor→verifier over 9 vendor CLIs; ship gate is a **deterministic JS check on command/exitCode/output evidence**, not an LLM opinion | MIT | **The TDR machine already built** |
| **claude-squad** | Go, tmux-based, cheapest possible isolation | — | Baseline to beat |

### Convergent findings across all seven

1. **Nobody normalizes config; everybody writes per-vendor adapters** into each vendor's native hook/config system. herdr and vibe-kanban both maintain adapter tables for ~6 incompatible MCP config dialects. This is hard-won knowledge, and it is the asset.
2. **Session resume = store the vendor's session handle, relaunch with the vendor's own `--resume` flag.** Never persist the conversation yourself. Unanimous across herdr, cmux, vibe-kanban, crystal.
3. **Piped subprocess beats PTY when the vendor has a JSON stream mode** (vibe-kanban); PTY is required only for TUI-only agents (herdr, cmux). Sourcerer needs both paths.
4. **Git worktrees are the correct default isolation** — ~100–150 lines of shelling out to git, no dependencies. Every survivor uses them; cmux's lack of them is its central unsolved problem.
5. **The daemon/client split is load-bearing for a desktop app.** If the Tauri window owns the PTYs, closing the window kills the agents.
6. **Capability must be data, not code** — a declared capability flag per adapter, with UI degrading gracefully, rather than a lowest-common-denominator feature set.

### Demand-side research — what users actually complain about

Primary sources: GitHub issues on the orchestrators themselves, HN 47218318, 2026 post-mortems, Docker incident write-ups. Seven clusters, in order of recurrence:

1. **Parallelism does not pay off at small/medium task size.** One controlled comparison: **9× wall time, 2× cost, +8% quality.** Worktrees fix file conflicts and leave semantic conflicts, port/DB collisions, and the integration tax untouched.
2. **Cost is the #1 operational complaint and no tool shows it.** claude-flow P0 #2661: subscription quota gone in ~10 minutes. No per-session attribution, no budget, no fuse.
3. **Trust incidents are architectural, not bugs** — production DB deleted in 9 seconds, 717 GB wiped, three separate `rm -rf ~/` incidents, secrets leaking at ~2× baseline. Root cause everyone names: *nothing sits between the agent's reasoning and the shell.* Meanwhile permission prompts fire every 2–3 minutes, so users alias `--dangerously-skip-permissions` and delete the only safeguard.
4. **Switching agents loses everything** — MCP config vanishes in worktrees, transcripts don't port, instruction files fragmented across six filenames.
5. **Onboarding friction kills adoption** — cloud/OAuth bolted onto local tools, Python/npm dependency hell, npx hangs, no Windows support.
6. **Abandonment is the category's defining 2026 event** — Bloop shut down (April), Crystal deprecated (February), users pinning `vibe-kanban@0.1.43` and forking. *Local-first, full export, no server dependency are now explicit purchase criteria.*
7. **Review capacity, not generation, is the binding constraint.** Anthropic's own report: AI in ~60% of work, only 0–20% fully delegated.

> **Strategic read:** the winning feature is not more parallelism. It is a **cost / permission / state control plane** under agents the user already runs — local-first, exportable, no server required. Sourcerer is unusually well-positioned: it is already local-first, already Nix-reproducible, already a desktop app, and PolyForm-NC licensed rather than VC-funded-and-abandonable.

### Standards verdict (Aug 2026)

If only two protocols can be done well:

1. **MCP, client/host side** — under Linux Foundation AAIF; spec **2026-07-28** is the largest breaking change since launch (stateless core, no `initialize` handshake, no `Mcp-Session-Id`, Roots/Sampling/Logging **deprecated**, elicitation folded into **MRTR**). Support both 2025-11-25 and 2026-07-28 through the 12-month straddle. ~9,650 servers in the official registry.
2. **ACP, agent side** — this is **distribution, not plumbing**: implementing it puts Sourcerer inside Zed, every JetBrains IDE, VS Code, Neovim, Emacs and Obsidian with no editor plugins, via a registry you join with one PR. v1 stable, v2 draft — negotiate, default v1.

Ship in v1 but costing hours, not quarters: **AGENTS.md** (no formal schema — read and prepend), **SKILL.md** (frontmatter + progressive disclosure), **Agent Plugins 1.0** (Aug 6 2026; its `mcp.json` is the first portable MCP-config format and kills a translation problem you would otherwise own forever).

**Bet against:** Agentic Resource Discovery / `ai-catalog.json` (39-site census found *zero* serving one), A2A for coding workflows, MCP Sampling/Roots as architecture, Codex App Server Protocol as anything but a bridge target.

**Emit but don't surface:** OTel `gen_ai.*` — every attribute still marked Development, conventions moved repos mid-2026 with no tagged release.

### Highest-regret integrations found (don't rebuild these)

- **Serena** — LSP-over-MCP symbol tools; upgrades *every* wrapped harness at once because it attaches at the MCP layer
- **Jujutsu (jj)** — its operation log is a native "undo the agent's last turn" primitive git does not have, while still writing real git commits
- **ccusage** — unified cost across 15+ agent CLIs read from local logs, no proxy in the request path
- **Aider's repo-map algorithm** (tree-sitter + PageRank) — best quality-per-token repo summarization; the algorithm is the asset
- **mergiraf** — semantic merge for the conflicts N parallel agents generate
- **difftastic** — structural diff so review spends attention on substance
- **Landlock / bubblewrap** — cheapest real isolation on the Nix/Linux target; already the substrate's world
- **systemd user units** — supervision, restart backoff, socket activation, readiness, structured logs, all free on our target
- **Letta / Zep-Graphiti / ReasoningBank** — memory-as-a-service shapes; ReasoningBank is the one a meta-layer is uniquely positioned to exploit (learn from every harness's failures)

### Patterns to steal from outside the AI space

- **LSP capability negotiation** — declare, don't assume; ignore what you don't understand. ACP copies it method-for-method.
- **tmux control mode (`-CC`)** — headless server + structured control channel. Our desktop-app-plus-CLI architecture, shipped and debugged in 2011.
- **Self-hosted CI runners** — "register this machine as an agent host" is exactly JIT registration + outbound long-poll + ephemeral one-shot runners + tag-based targeting.
- **Kubernetes reconciliation** — level-triggered spec/status/conditions, not edge-triggered "on agent exit do X". Edge-triggered handlers are why orchestrators wedge.
- **Bazel action cache** — content-address the *complete* input digest of an agent step; identical inputs shouldn't pay twice.
- **Lockfiles** — an agent run's inputs (model version, system prompt, skills, MCP versions, tool versions) are *more* volatile than a build's. We get this nearly free from Nix. **A session lock is a differentiator nobody has.**
- **OTP supervision trees** — restart intensity limits = a crash budget; per-provider circuit breakers and bulkheads; bounded queues for backpressure.
- **Jujutsu op log over Git reflog** — append-only whole-state snapshots with real undo.

## Constraints inherited (must not be re-litigated)

From `PROJECT.md`, `REQUIREMENTS.md`, and `research/AI-PROVIDER-ARCHITECTURE.md`:

- **Nix-native rule** — if it's not in nixpkgs, we package it. OCI is never a submission format. Sole exception: the time-boxed Phase 13 engine escape hatch.
- **AI provider architecture is DECIDED (2026-08-04)** — two-tier hosted-default / local-opt-in; hosted and local **pin the same embedding model**; weights are data, not code (download to `StateDirectory`, digest-verified, never in the Nix store).
- **`host.ai()` seam stays unchanged applet-side** (ENG-02) while the harness relocates substrate-side (Phase 14).
- **License: PolyForm NC 1.0.0** — GPL competitor code (cmux) cannot be copied, only studied.
- **Two-host reality** — NixOS dev host + Windows box; every phase declares its execution host.
- **Current position:** Phase 9 of 8–19 (v2.0 Container Platform), plan 7 of 7.

## Prior spikes not to re-derive

| # | Finding |
|---|---|
| 001 | pi-headless-embed — Pi runs headless behind the seam |
| 002 | databasise-tools-over-pi — **VALIDATED**, Databasise MCP/REST tools resolve real wiki queries in a chat turn |
| 003 | lean-prompt-modes — prompt modes shrink the baseline |
| 004 | omp-component-harvest — **VALIDATED**, liftable components list with license/coupling notes |
| 005 | token-baseline-benchmark — **VALIDATED**, lean-Pi vs OMP vs opencode; the "22k is overkill" claim confirmed |

## Requirements

**APPROVED (user, 2026-08-11)** — see [DESIGN.md](DESIGN.md) in this directory for the
full requirement set (A-1…E) and the four-layer framework model
(Claim → Task → Session → Harness). Brainstorm outcomes fixed there:

- Pi and the Assistant GUI are rebuilt; interactive-GUI behaviour becomes a **skill**
- Harness choice deferred by construction; the **ingestion study** decides it
- P0 set: cost ledger, policy boundary, claim store + verifier loop, daemon/client
  split, MCP client, AGENTS.md/SKILL.md

## Investigation Trail

1. Started assuming Sourcerer was a coding CLI competing with omnigent (working-directory name misleads). Reading `.planning/` corrected it: Sourcerer is a scholar's research workbench whose *Assistant* is the subject.
2. Assumed the coding-orchestrator feature set was therefore mostly irrelevant. **User overturned this** with the TDR thesis — coding is a subset of test-driven research, code is needed at every level. That inverted the relevance judgment on roughly half the competitor capabilities.
3. Two discovery agents mis-read the brief and analyzed as if "Sourcerer = the Omnigent meta-harness." Their *findings* survive (the categories and tools are real and independently sourced); their *framing* about what Sourcerer already has does not.
4. Expected the pain research to argue for better orchestration. It argues the opposite — parallelism measurably loses at small/medium task size, and the unmet need is a control plane. This is the single most decision-relevant finding in the spike.

## Results

**VALIDATED.** Ranked requirement set with a defensible cut exists and is approved:
11 requirements across 4 groups plus 4 standing invariants, P0/P1/P2-ranked, each
traceable to teardown or demand-side evidence. The framework model (Claim → Task →
Session → Harness) makes TDR structural and defers the harness decision to the
ingestion study, which is the named next step.
