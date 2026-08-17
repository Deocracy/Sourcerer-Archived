---
spike: 007
name: ingestion-study
type: standard
validates: "Given the approved requirements A-1…D-4, when the seven indexed competitor repos are mined, then each requirement has a build-vs-copy verdict with named source code, and the harness question has an evidence-based recommendation"
verdict: VALIDATED
related: [006]
tags: [ingestion, build-vs-copy, licenses, harness]
---

# Spike 007: Ingestion Study — Build vs Copy per Requirement

Source material: the seven competitor repos cloned at
`../Agnostic agentic coding tools-CLI for Sourcerer/competitors/` and indexed in the
codebase-memory graph, plus the spike-006 teardown inventories (file:line evidence).

## License gate (checked first)

| Repo | License | Can we lift code into PolyForm NC Sourcerer? |
|---|---|---|
| herdr | Apache-2.0 (since 0.8.0) | **Yes** |
| vibe-kanban | Apache-2.0 | **Yes** (sunsetting — take now) |
| crystal | MIT | **Yes** (frozen since Feb 2026 — take now) |
| zeroshot | MIT | **Yes** |
| claude-flow | MIT | Yes, but little worth taking |
| claude-squad | — (Go, tmux) | Designs only; nothing needed |
| cmux | GPL-3.0 + carve-out | **No code, ever. Designs only.** |

## Verdicts by requirement

### A-1 Cost ledger — COPY the shapes, build in Rust

- **zeroshot** `src/ledger.js:878-941` + `types.ts:502-519`: per-role rollup `{inputTokens, outputTokens, cacheRead, cacheCreation, totalCostUsd}` aggregated from provider result events. This is the normalized-row schema.
- **claude-flow**: the **user-global cost fuse** — home-dir-scoped, not workspace-scoped, which is the load-bearing insight (a runaway task must trip a *user* budget, not just its own). Its one genuinely copyable invention.
- **vibe-kanban** `NormalizedEntryType::TokenUsageInfo`: usage as a first-class entry in the conversation stream, so the UI shows cost inline.
- Integrate, don't rebuild: **ccusage** reads 15+ vendor CLIs' local logs with no proxy — the fallback when a harness under-reports (spike 005 in the other project: pi/goose/qwen/kimi report nothing).

### A-2 Policy boundary — BUILD, copying two patterns

- **zeroshot** fail-closed capability probing (`detectCliFeatures`, claude.ts:58-81): unprobed features default optimistic **except safety-relevant flags, which fail closed**. Copy the rule.
- **zeroshot** force-injected safety hooks (`--settings` + PreToolUse hard-stops on dangerous git). Proof that runtime enforcement survives what prompt rules do not.
- **vibe-kanban** `ExecutorApprovalService` (approvals.rs): three normalized policies + cancellation-aware async approval — the smallest policy set that maps onto every agent. Rust, Apache — liftable near-verbatim.
- Warning from the same repo: vibe-kanban **bypasses permissions by default** (`permission_mode()` returns bypass). We invert that default; the sandbox (C-2) is what makes safe-by-default not annoying.

### A-3 Session lock — BUILD (nobody has it)

Prior art is only partial: vibe-kanban pins vendor CLI versions as literal strings in
Rust source (`npx -y @anthropic-ai/claude-code@2.1.119`); zeroshot versions its adapter
contract and model catalog. Nobody locks the full run input set. We build it on Nix —
flake pin + model version + prompt/skill digests + MCP server versions in one
per-run lockfile. Differentiator confirmed.

### B-1 Claim store — BUILD (nobody has it), one precedent to copy

- **zeroshot's evidence-freshness rule**: quality-gate evidence with `completedAt` earlier than the last `IMPLEMENTATION_READY` is void. That is claim-staleness, already implemented — generalize it: evidence older than the source it cites is void.
- Ledger shape (append-only SQLite, replayable, `Orchestrator.resume` replays from it) is the right persistence model for claims too.

### B-2 Verifier loop — COPY zeroshot wholesale (the biggest single lift)

MIT-licensed and exactly the TDR machine:

- `criteriaResults[]` schema: `{id, status: PASS|FAIL|SKIPPED|CANNOT_VALIDATE, evidence: {command, exitCode, output}, reason}` — machine-readable evidence, deterministic ship gate ("everything an LLM produces is treated as a claim").
- `CANNOT_VALIDATE` as a first-class outcome requiring a stated reason — the anti-deadlock rule.
- **Context quarantine**: validators receive only ledger events (`ISSUE_OPENED`, `PLAN_READY`, `IMPLEMENTATION_READY`), never the worker's transcript. Independence is structural, not prompted.
- Validators forbidden from git; evidence freshness (above); failure terminalization policy per role.

### B-3 Deep Research — BUILD over B-1/B-2, copy the router

**zeroshot's conductor**: cheap junior classifier emits `{complexity, taskType}`,
escalates only `UNCERTAIN` to a senior pass; complexity scales validator count
(TRIVIAL 0 → CRITICAL 5). Same router, research-flavored: claim criticality scales
cross-checking depth.

### C-1 Daemon/client split — COPY herdr's architecture (Rust→Rust)

The single largest architectural lift, Apache-2.0:

- Server owns every PTY/session; clients attach over a **versioned** wire protocol (PROTOCOL_VERSION + documented bump policy) and detach without killing anything.
- **~90-method NDJSON socket API with a schemars-generated published schema and CLI parity** — every CLI subcommand is a thin wrapper over one API method. This is exactly what the Tauri window needs as a client.
- **Race-free waits**: `agent.wait` pins the resolved pane occupant so a replacement can't satisfy the wait; a 5-second no-lifecycle-change guard returns `stalled` instead of hanging. These two details are what separate a usable wait primitive from 3am deadlocks.
- `session.snapshot` + event stream = the snapshot/delta cache contract for the UI.
- **herdr's skill-in-binary trick**: SKILL.md compiled into the binary, printable via `--skill`, always version-matched — adopt for Sourcerer's own agent-facing surface (D-2).

### C-2 Workspace isolation — COPY a worktree manager, keep the substrate's sandbox

- **vibe-kanban** `worktree_manager.rs` (Rust, Apache): per-path async mutex against races, validity re-check (dir exists AND registered in git metadata), orphan/expired cleanup. Lift near-verbatim.
- **zeroshot** `isolation-manager.js` adds the hardening details: base-ref allowlist regex, disk-space guard, orphan auto-GC — copy the checks into the Rust manager.
- Sandbox tier is *not* copied from anyone: Landlock/bubblewrap are already the substrate's native world (Nix advantage; herdr has no sandbox at all, cmux none either).
- Demand-side additions no competitor ships: port/env/DB allocation per workspace, and absolute-path leak verification at dispatch.

### C-3 Op-log undo — BUILD small, copy the UX

- **crystal's execution-scoped diff timeline** (MIT): "show me the diff for prompt #4, or prompts 4–7 combined" — the best UX idea in the survey; checkpoint-commit per turn makes every agent turn revertible.
- Mechanism: vibe-kanban's auto-commit-per-turn + jj's op-log concept for whole-turn undo across documents and code.

### D-1 MCP client — BUILD on the official Rust SDK, COPY one table

The protocol work is ours (2025-11-25 + 2026-07-28 straddle, OAuth 2.1). The thing to
lift: **vibe-kanban `mcp_config.rs`** — the adapter table over six incompatible vendor
MCP-config dialects with comment-preserving JSON editing. Only needed if/when we write
into vendor configs; Agent Plugins 1.0's portable mcp.json (D-3) may obsolete it.

### D-2 AGENTS.md + SKILL.md — BUILD (hours), copy two scans

vibe-kanban `slash_commands.rs` discovery scan (project `.claude/commands`, global,
plugins, `skills/*/SKILL.md` merged into one descriptor list) + herdr's version-matched
skill-in-binary. The Assistant's interactive-GUI behaviour ships as a skill here.

### D-3 Agent Plugins 1.0 — BUILD (new standard, no prior art to copy)

### D-4 / Harness — the evidence-based recommendation

Three proven patterns, with measured costs:

| Pattern | Who proved it | Cost |
|---|---|---|
| Headless piped subprocess + JSON stream | zeroshot, vibe-kanban, claude-flow | Bespoke protocol: thousands of lines per vendor. **Via ACP: 545 shared lines + ~200/agent** (vibe-kanban's own numbers) |
| PTY host + screen-scrape manifests | herdr, cmux | 1,514-line detection engine + 19 TOML manifests + perpetual vendor-TUI churn |
| Embedded engine | Sourcerer v1 (Pi) | Full control, no user-subscription reuse |

**Recommendation:** for the TDR engine, Sessions are headless workers under Tasks — the
**piped-subprocess/JSON pattern is primary**, and **ACP is the wire format to prefer**
when vendor support exists (it is the only path whose per-vendor cost is ~200 lines).
PTY hosting is needed **only** if we later surface interactive vendor TUIs in the shell —
defer it; herdr's manifests are the known fallback. The embedded engine remains the
Nix-pinned default until the rebuild replaces it.

One more zeroshot pattern to adopt regardless of harness choice: **the provider layer as
a standalone JSON stdin/stdout executable** (versioned envelope, shadow-first rollout
with parity fixtures). A hard process boundary means harness adapters can evolve — or be
swapped wholesale — without touching the Rust core. That is the seam that keeps the
"maybe we remake the entire stack" option cheap.

## Summary table

| Req | Verdict | Primary source |
|---|---|---|
| A-1 | Copy shapes, build Rust | zeroshot ledger + claude-flow fuse + ccusage |
| A-2 | Build, copy 2 patterns | zeroshot fail-closed probe · vibe-kanban approvals |
| A-3 | Build (novel) | Nix; partial prior art only |
| B-1 | Build (novel) | zeroshot evidence-freshness as staleness precedent |
| B-2 | **Copy wholesale** | zeroshot verifier contract (MIT) |
| B-3 | Build over B-1/2 | zeroshot conductor router |
| C-1 | **Copy architecture** | herdr daemon/protocol/API (Apache) |
| C-2 | Copy manager, keep sandbox | vibe-kanban worktree_manager + zeroshot hardening |
| C-3 | Build small, copy UX | crystal diff timeline (MIT) |
| D-1 | Build on official SDK | vibe-kanban mcp_config table if needed |
| D-2 | Build (hours) | vibe-kanban discovery scan · herdr skill-in-binary |
| D-3 | Build (new standard) | — |
| D-4 | Piped+ACP primary, PTY deferred | vibe-kanban's 545+200 measurement |

## Results

**VALIDATED.** Every requirement has a verdict and a named source. Two wholesale lifts
(zeroshot's verifier contract, herdr's daemon architecture), two novel builds that are
the differentiators (session lock, claim store), and an evidence-based harness
recommendation that keeps the full-stack-remake option open via a process-boundary seam.
