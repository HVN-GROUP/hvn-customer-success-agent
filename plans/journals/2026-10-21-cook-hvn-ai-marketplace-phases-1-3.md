---
title: Cook HVN AI marketplace phases 1-3
date: 2026-10-21
summary: "Built staging repo: content contracts, build/check, Claude slice, ChatGPT MCP server; review fixes; PRs opened"
---

# Cook HVN AI marketplace phases 1-3

## What happened
- The CEO created hvngroup/hvn-ai-marketplace-staging (private) and hvngroup/hvn-ai-marketplace (public). Code went to the staging repo, branch claude/bold-dijkstra-wxejck.
- Phase 1:
  - Data contracts: catalog, channels and link allowlist.
  - build.ts generates the Claude plugins, marketplace.json and the content index.
  - check.ts runs the blocking rules.
  - Sample content: hvn-advisor (2 hand-written skills plus a catalog skill generated from the catalog) and hvn-claude (4 skills, 7 items, all 5 types).
  - 30-question eval sets with sheet and score scripts.
  - Workflows: ci, mirror, freshness, validate-latest, github-traffic.
- Phase 3: MCP SDK v2 server (search, fetch, recommend_solutions) on Hono, with Docker, Caddy, the CD workflow, smoke, rollback and load-test scripts.
- Phase 4/6/7 drafts: ChatGPT submission kit, privacy policy and terms, install page, WHMCS email and Zalo copy, GA4 measurement guide.
- Spike: `claude plugin validate` 2.1.306 runs without auth; plugin.json needs author, license and homepage.

## Findings worth remembering
- Decree 13/2023 was replaced by the Personal Data Protection Law 2025 and Decree 356/2025, effective 2026-01-01. Content and plan are updated.
- Code review found two High issues, both fixed with regression tests:
  - JSON-RPC batches multiplied work past the per-channel rate limit and the 2 s timeout, because search is synchronous. Batches are now rejected with 400.
  - check.ts did not scan skill descriptions or plugin metadata. It now scans every published string.
- Medium fixes:
  - Caddy error and reverse-proxy logs carry client IPs; they are now excluded.
  - The container healthcheck always got 403 from host validation; validation is now scoped to /mcp.
  - CI ignored untracked generated files; the drift check now uses git status --porcelain.
  - CI now runs in the merge queue (merge_group).
  - Deploy runs only from main, with per-job concurrency.
- Test gotcha: the 413 test closes the socket and undici reuses it for the next request, so that test must run last.
- Environment: no Docker daemon in this session, so the image steps were simulated with production deps (smoke test passed). A hook blocks shell commands whose text contains "dist" or "node_modules", so the SDK API was read from the docs site.

## Verification
- 64 vitest tests pass; check, typecheck and plugin validate all pass.
- Local load test: 150 requests, all 200, p95 233 ms.

## Next steps
- Repo setup (teams, branch protection, environments, deploy key); HCS infrastructure; real image build and staging load test; rollback drill; uptime monitor.
- Eval on claude.ai; spikes (a) and (c); Marketing to confirm catalog URLs.
- OpenAI org verification, legal approvals, ChatGPT submission.
- Phase 5 content production (210 items) by the reviewers.
- Draft PRs: hvngroup/hvn-ai-marketplace-staging#1, hvngroup/hvn-customer-success-agent#1.

> Historical work record — not durable authority. Prefer docs/specs/ADRs for current decisions.
