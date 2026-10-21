---
title: HVN AI plugin marketplace wave 1 plan
date: 2026-10-21
summary: Advise + hard-mode plan for a public Claude marketplace and ChatGPT plugin; red team reshaped the repo and data contracts
---

# HVN AI plugin marketplace wave 1 plan

## What happened
- Advisory interview with the CEO locked scope: public content, no login, no prices, UTM links to brand info pages, upsell strong on Claude / relevant on ChatGPT, 7 plugins (hvn-advisor + 6 specialist), AI drafts with human review, Vietnamese first, 6–8 weeks.
- Two researchers verified platform facts: Claude marketplace needs a public GitHub repo; plugin dependencies and symlinks are Claude Code-only (shared files must be copied at build); OpenAI renamed Apps to Plugins and bans promoting subscriptions or linking to checkout; Custom GPTs retire 2026-12-11; MCP TS SDK v2 is stateless; WHMCS has no UTM capture and its GA4 addon misses bank-transfer orders.
- Plan written at plans/261021-1121-hvn-ai-plugin-marketplace (7 phases).

## Red team (15 accepted of 39 raw findings)
- Drafting in a public repo would leak ticket data on PR open, so authoring and review moved to a private staging repo mirrored to the public repo after merge.
- Content source is now separated from generated plugins: the suggestion block is embedded per item at build time, not a "read this file and follow it" instruction.
- The index unit is now the item, not the skill.
- UTM and needs vocabulary now come from config/channels.yaml and catalog.
- Added a file and link allowlist, dangerous-command checks, a yaml parser instead of gray-matter, and least-privilege CI.
- MCP server: per-channel rate limits, IP-free logs, staging, CD with rollback, uptime monitoring.
- WHMCS: cart redirect spike in week 1–2, 2 cart-only hooks with a kill switch.
- Every 90-day metric now has a defined data source.

## Decisions (Validation Log)
- N = 35 items per specialist plugin (210 total).
- Reviewers: Content #Valkyrie and Service #Witch.
- Code is MIT; content is © HVN GROUP with use and internal-mirror permission.
- ChatGPT submission keeps recommend_solutions and excludes chon-goi.
- No Anthropic directory submission in wave 1.
- This internal repo is private.

## Next steps
- Week 1: platform spikes (claude.ai account tiers, CI validate without auth, GitHub org ownership); start OpenAI org verification; WHMCS cart spike; name the reviewers.
- Execute with /ak:cook plans/261021-1121-hvn-ai-plugin-marketplace/plan.md.

> Historical work record — not durable authority. Prefer docs/specs/ADRs for current decisions.
