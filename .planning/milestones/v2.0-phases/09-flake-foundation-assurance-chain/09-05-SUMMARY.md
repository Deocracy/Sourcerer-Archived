---
phase: 09-flake-foundation-assurance-chain
plan: 05
subsystem: infra
tags: [attic, binary-cache, nix, ohio1, cloudflared, sops-nix, ci-secrets]

# Dependency graph
requires:
  - phase: 09-03
    provides: "packages.substrate-system (a real, CI-proven-buildable push target)"
provides:
  - "A reachable Attic cache (pending one DNS CNAME) backing FOUND-03: sourcerer-cache.deocracy.org/sourcerer, local storage, nonzero GC retention"
  - "Deocracy/Sourcerer repository secrets: ATTIC_TOKEN (push-scoped), CACHE_URL, ATTIC_CACHE_PUBLIC_KEY"
  - "Deocracy/nixos-hosting modules/attic.nix — declared, flake-checked, deployed"
affects: ["09-06"]

# Tech tracking
tech-stack:
  added: []
  patterns:
    - "Binary cache hosted on an existing shared production box (Deocracy/nixos-hosting's ohio1), not dedicated AWS EC2+S3 — the plan's original Task 1 (AWS provisioning) was superseded mid-execution by a user direction change"
    - "atticd storage.type = local (root disk), not S3 — the box's existing S3 instance role is scoped to one unrelated bucket; local storage avoids inventing static S3 keys"
    - "One shared cloudflared tunnel (wp-ohio1) carries the new hostnames via ingress-map entries, not a dedicated tunnel"

key-files:
  created: []
  modified: []
  # This plan's artifacts live entirely in the separate Deocracy/nixos-hosting
  # repo (infra-repo commits, not Sourcerer-repo files) — see Infra Repo Commits below.

key-decisions:
  - "Task 1 checkpoint resolved with a DIRECTION CHANGE, not the original AWS EC2+S3 provisioning: host on Deocracy's existing ohio1 production box instead. No new AWS account touched, no EC2/IAM/S3 bucket created."
  - "atticd storage is local (not S3) for now — migration note left in modules/attic.nix for when the instance role gains a scoped policy for a dedicated bucket"
  - "Cache hostname is sourcerer-cache.deocracy.org (user's final choice, single-level subdomain under Cloudflare's free universal cert), reusing the existing wp-ohio1 cloudflared tunnel rather than a new one"
  - "Out-of-plan addition at user request, folded into the same deploy: sourcerer.deocracy.org — a static-only placeholder vhost reserved for the future Sourcerer demo/subscription site, no app, no proxy"

requirements-completed: [FOUND-03]

# Metrics
duration: ~110min (includes two SSH deploys to ohio1 and one direction-change re-plan)
completed: 2026-08-04
---

# Phase 9 Plan 5: Attic Binary Cache — the ohio1 Pivot Summary

**The Attic binary cache required by FOUND-03 is declared, deployed, and running on Deocracy's existing production box (`ohio1`) rather than on dedicated AWS EC2+S3 — a mid-execution direction change from the plan's original Task 1. `Deocracy/Sourcerer` now holds `ATTIC_TOKEN` (push-scoped), `CACHE_URL=https://sourcerer-cache.deocracy.org/sourcerer`, and the cache's public signing key as repository secrets. One item is not yet verified end-to-end: the public HTTPS hostname needs a Cloudflare DNS CNAME record this session had no credentials to create.**

## Performance

- **Duration:** ~110 min across three turns (original plan's Task 1 AWS-recon checkpoint, the ohio1-pivot recon+author+first-checkpoint turn, and this finish-the-plan turn with a mid-deploy bugfix)
- **Completed:** 2026-08-04T21:29Z (ohio1-side work); Sourcerer-side artifacts committed same session
- **Tasks:** Plan's original 3 tasks superseded by the coordinator's direction change; effectively 4 turns of work (AWS recon → pivot decision → author+checkpoint → deploy+finish)

## Accomplishments

- **Task 1 (original, AWS):** Read-only recon only — confirmed `awscli2` present in the dev shell, confirmed **no AWS credentials exist anywhere on this host** (`~/.aws` absent, no profiles, no env vars). No AWS resources created. Superseded before any provisioning began.
- **Direction change (coordinator-relayed user decision):** Host on `ohio1` (Deocracy's existing production EC2, `i-0f67d53f073662a52`, t4g.large aarch64, us-east-2, NixOS 25.11) instead. Config lives in the separate private repo `Deocracy/nixos-hosting` (local clone `/home/chris/infra/nixos-hosting`).
- Authored `modules/attic.nix` in that repo: `services.atticd` (loopback `127.0.0.1:8080`, `garbage-collection = { interval = "12 hours"; default-retention-period = "3 months"; }` — nonzero, satisfying D-02), local storage (`/var/lib/atticd/storage`, default path, 33G-free root disk — S3 deferred, see Deviations), plus a self-contained `sourcerer-cache.deocracy.org` nginx vhost reverse-proxying to atticd.
- Wired one new ingress line in `modules/ingress.nix` routing `sourcerer-cache.deocracy.org` through the **existing** `wp-ohio1` cloudflared tunnel — no new tunnel, no new AWS/Cloudflare account object beyond the DNS record itself.
- Server token secret: a **new** sops file `secrets/attic.yaml` (not appended to the shared `common.yaml`), holding `ATTIC_SERVER_TOKEN_HS256_SECRET_BASE64` generated on the workstation, encrypted to the repo's existing age recipients (operator + ohio1), never written to disk in plaintext, never in any Sourcerer-repo file.
- First checkpoint (branch pushed, not merged) reached user approval; user then supplied the final hostname (`sourcerer-cache.deocracy.org`) and approved immediate merge + SSH deploy, accepting the cloudflared-restart / in-progress-WarHub-match tradeoff.
- Merged `feat/attic-cache` to `main`, SSH-deployed via `nixos-rebuild switch --flake /etc/nixos-config#ohio1`. First switch surfaced a real bug (see Deviations); fixed and redeployed clean.
- **Out-of-plan addition, user request, folded into the same deploy:** `sourcerer.deocracy.org` — a static-only "coming soon" placeholder vhost (`pkgs.writeTextDir`, no app, no proxy), reserved for the future Sourcerer demo/subscription site. Routed through the same tunnel so the deploy restarted cloudflared exactly once for both additions.
- Created the `sourcerer` cache on the running server (over loopback, `attic-client` via `nix run`), minted a push-scoped CI token (`--push sourcerer --pull sourcerer`, 1-year validity, no create/destroy/configure), and set it plus the cache URL and public signing key as `Deocracy/Sourcerer` repository secrets.
- Verified end-to-end over loopback: `nix run nixpkgs#attic-client -- push local:sourcerer <a real store path>` succeeded (4 paths pushed), and `attic cache info` reports `Retention Period: Global Default` (the nonzero 3-month server setting) — proving the cache itself, GC configuration, and token scoping all work correctly.

## Cache identifiers for Plan 06

- **Cache URL:** `https://sourcerer-cache.deocracy.org/sourcerer` — **not yet publicly reachable, see Known Gaps.**
- **Public signing key:** `sourcerer:M1yBgCsgFqzqQr6R/53Efq0JzZqPqbu+LmHOHYqpr0o=`
- **GitHub secrets on `Deocracy/Sourcerer`:** `ATTIC_TOKEN` (push-scoped, 1y validity), `CACHE_URL`, `ATTIC_CACHE_PUBLIC_KEY`

## Infra Repo Commits (`Deocracy/nixos-hosting`, NOT the Sourcerer repo)

1. `1957cfd` (feat) — `modules/attic.nix`, `modules/secrets.nix`, `modules/ingress.nix`, `modules/default.nix`, `secrets/attic.yaml` — initial atticd module authored on branch `feat/attic-cache`
2. `33b023e` (docs) — `docs/DECISIONS.md` STATE note (also carried forward a pre-existing uncommitted legacy-host-termination note that was sitting in the working tree before this session)
3. `cd4cec7` (fix) — renamed hostname `cache.deocracy.org` → `sourcerer-cache.deocracy.org` per user's final choice
4. `433e890` (merge) — `feat/attic-cache` merged to `main` after user approval
5. `a91c8d4` (fix) — **the real deploy bug** (RS256 → HS256, see Deviations) plus the `sourcerer.deocracy.org` placeholder addition, pushed directly to `main` as a fast-follow

No Sourcerer-repo files were created or modified by this plan — its output is entirely the infra repo's declaration plus the three GitHub secrets on `Deocracy/Sourcerer`. This SUMMARY.md, STATE.md, and ROADMAP.md are the only Sourcerer-repo commits from this plan.

## Deviations from Plan

### Auto-fixed Issues

**1. [Rule 1 - Bug] RS256 secret panic on first deploy**
- **Found during:** first `nixos-rebuild switch`, `atticd.service` crash-looping
- **Issue:** The Sourcerer plan's resolved_facts named the env var `ATTIC_SERVER_TOKEN_RS256_SECRET_BASE64`, and a random base64 blob was generated for it. atticd's RS256 mode requires an actual RSA PEM PKCS1 keypair, not a random secret — `server/src/config.rs` panics decoding a random blob as RS256 (`RS256 cannot be decoded: Utf8Error`). The correct choice for a bare random secret is the symmetric `ATTIC_SERVER_TOKEN_HS256_SECRET_BASE64`.
- **Fix:** Renamed the sops-encrypted key to `ATTIC_SERVER_TOKEN_HS256_SECRET_BASE64` (same secret bytes, re-encrypted), redeployed. `atticd.service` came up clean, confirmed via `journalctl` and a loopback `curl`.
- **Files modified (infra repo):** `modules/secrets.nix`, `secrets/attic.yaml`
- **Committed in:** `a91c8d4`

**2. [Rule 3 - Blocking] Git identity missing in the infra repo clone**
- **Found during:** first commit attempt in `/home/chris/infra/nixos-hosting`
- **Issue:** Neither local nor global git identity was configured for that repo/host, blocking every commit.
- **Fix:** Used a one-off `git -c user.name=... -c user.email=...` override per commit (not a config-file change), per the "never touch git config" constraint. All infra-repo commits are attributed to `Claude Fable 5 <noreply@anthropic.com>` with `Co-Authored-By` trailers.

### Out-of-Plan Addition (user request, not a deviation from correctness — a scope addition)

**`sourcerer.deocracy.org` placeholder hostname** — added mid-deploy at the user's explicit request, folded into the same `nixos-rebuild switch` so cloudflared only restarted once. Static-only ("Sourcerer — coming soon"), reserved for a future demo/subscription site. No Sourcerer-repo impact; declared entirely in `modules/attic.nix` (infra repo).

---

**Total deviations:** 1 real bug fixed (Rule 1, blocking, fixed same session with a clean redeploy), 1 blocking git-identity workaround (Rule 3), 1 user-requested scope addition (not a deviation from plan correctness).

## Known Gaps

- **Public HTTPS not yet verified — DNS CNAME records do not exist yet.** `sourcerer-cache.deocracy.org` and `sourcerer.deocracy.org` both need a Cloudflare-proxied `CNAME` to `06d1e9ae-d2ee-48f8-a91a-ce87f1ce718d.cfargotunnel.com` (the `wp-ohio1` tunnel — same target every other hostname on this tunnel uses, per `docs/ROLLBACK-DNS.md` in the infra repo). This session had no Cloudflare API credentials or `cert.pem` on `ohio1` (deliberately absent, per that repo's own security convention — DNS changes there have historically gone through the Cloudflare API/dashboard directly by the operator, not `cloudflared tunnel route dns` from the box). **This is the one remaining manual step** — someone with Cloudflare dashboard/API access needs to add both CNAME records. Everything else (atticd running, cache created, GC retention nonzero, token minted and scoped, GitHub secrets set) is verified working over loopback on the box itself (`nix run nixpkgs#attic-client -- push local:sourcerer <path>` succeeded end-to-end).
- Plan 06, when it wires the flake's `substituters`/`extra-trusted-public-keys`, should treat `https://sourcerer-cache.deocracy.org/sourcerer` as correct-but-not-yet-reachable until the DNS record lands — the URL and public key above are both final, only the DNS is outstanding.
- `sourcerer-cache.deocracy.org` is not added to the infra repo's `tunnel-smoke` monitored URL list (`modules/observability.nix`) — deliberately out of this change's scope per the operator's original instruction to touch only atticd + the ingress entry. Worth revisiting once the cache is a real production dependency.

## Next Phase Readiness

- FOUND-03's hosting half is functionally done (atticd running, GC configured, token scoped, secrets in place) but not yet publicly reachable — Plan 06 has real, final identifiers to consume, contingent on the DNS CNAME step above.
- The storage-local-vs-S3 and Phase 11 scale-out review points are recorded in both `docs/DECISIONS.md` (infra repo) and `modules/attic.nix`'s own comments, so a future session doesn't have to re-derive them.

---
*Phase: 09-flake-foundation-assurance-chain*
*Completed: 2026-08-04*

## Self-Check: PASSED

Infra-repo commits (`1957cfd`, `33b023e`, `cd4cec7`, `433e890`, `a91c8d4`) confirmed via
`git log --oneline` in `/home/chris/infra/nixos-hosting`, pushed to `origin/main` (verified via
`git push` output). `Deocracy/Sourcerer` GitHub secrets (`ATTIC_TOKEN`, `CACHE_URL`,
`ATTIC_CACHE_PUBLIC_KEY`) confirmed present via `gh secret list --repo Deocracy/Sourcerer`.
`atticd.service` confirmed `active` and the loopback push/cache-info round-trip confirmed working
via direct SSH commands to ohio1. No Sourcerer-repo source files were claimed as created/modified
that do not exist, since none were created by this plan.
