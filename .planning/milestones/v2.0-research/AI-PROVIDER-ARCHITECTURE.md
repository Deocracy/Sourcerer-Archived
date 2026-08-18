# AI Provider Architecture — decision set (2026-08-04)

**Status:** DECIDED (user, 2026-08-04, Phase 8 discussion follow-on). Consumed by Phases 13/14/15 planning; macOS leg by Platform v2 (P9/P14).
**Supplements:** `CONTAINER-PLATFORM.md` §6 (compute routing) · `08-CONTEXT.md` (Nix-native rule these decisions build on).

## The two-tier model

| Tier | Embeddings | Generation | Setup |
|------|-----------|------------|-------|
| **Default** | Sourcerer-hosted server, open-source model, on its own box (separate from any tenant hosting) | Cloud API via existing Pi sidecar path (unchanged) | Zero — works at install |
| **Local (opt-in)** | Same pinned model, served locally | Ollama in the substrate | One click: Ollama is a **catalog app** (Phase 15) |

When a local endpoint is installed, `host.ai()` routes to it; otherwise hosted. Per-capability (embed/generate), invisible to applets.

## Binding rules

1. **Open-source embedding models only, self-hosted.** Never a proprietary embedding API.
2. **Hosted and local pin the SAME embedding model.** Vectors from different models are not comparable — same model means switching to local is a config change and the user's existing index keeps working; different models would silently break retrieval or force full re-embedding. This rule is what makes hosted→local migration possible at all.
3. **Weights are data, not code.** Runtimes are Nix-packaged (`services.ollama`, `llama-cpp`, `onnxruntime` — all in nixpkgs, cached; Nix-native rule satisfied); weights download on first use into `StateDirectory`, digest-verified. Never in the Nix store / binary cache.

## Per-platform plumbing (same UX, different route)

- **Windows/WSL2:** Ollama runs inside the substrate VM as a hardened unit. GPU via `/dev/dxg` (WSL GPU-PV) — **spike B resolved: user reports GPU-in-WSL2 proven in a separate test (2026-08-04; evidence location not yet recorded — ask before Phase 15 planning)**. Catalog app selects `ollama-cuda`/`-rocm`/`-vulkan`/`-cpu` variant; `DeviceAllow=/dev/dxg` vs `PrivateDevices` is a compile-time hardening detail, handled by Phase 8's knob-by-knob method, not a new spike.
- **macOS (Platform v2):** the fast path (Metal/MLX) is **host-native, not in-VM** — Virtualization.framework does not pass the ML compute path into a Linux guest. Run Ollama/MLX natively on the host; substrate points at it. Matches §6's "host-native Metal helper".

## Consequences owned

- **The hosted embedding server is a second standing production service** (after the binary cache): uptime, model-version management, re-embed policy on model change. Cost/runbook treatment like FOUND-03.
- **Privacy wording must be honest:** default tier sends documents to Sourcerer's embedding server; installing local AI keeps everything on-machine — that is the *point* of the local tier. Phase 19's user-facing security notes own the wording.
- **Phase 15 manifest schema needs a runtime-download concept:** declared size budget + fetch-and-verify into `StateDirectory` (the Ollama app pulls multi-GB weights). Decide at schema design, not after the compiler exists.
- **`host.ai()` grows a provider concept** (hosted-default / local-override, per capability) — Phase 13/14 seam design.

## Explicitly rejected

- Per-tenant model instances at scale (200 active × ~2 GB per 3B — not viable; shared services or API only).
- Proprietary embedding APIs (lock-in: model deprecation forces corpus re-embedding).
- API-only AI (breaks offline ingest, breaks the privacy story, and the local tier is a product point, not an optimization).
