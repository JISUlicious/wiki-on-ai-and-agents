---
title: "What a task costs on Opus 5.5"
type: source
created: 2026-10-02
updated: 2026-10-02
sources:
  - what-a-task-costs-on-opus-5-5-osmani-2026.md
authors:
  - Addy Osmani
year: 2026
venue: "claude.dev (Anthropic developer blog)"
publisher: Anthropic
status: complete
importance: high
tags:
  - 2026
---

# What a task costs on Opus 5.5

**Source**: `sources/what-a-task-costs-on-opus-5-5-osmani-2026.md` (official [[anthropic|Anthropic]] developer blog)
**Author**: Addy Osmani (Anthropic)
**Date**: 2026-09-25
**URL**: https://claude.dev/blog/what-a-task-costs-on-opus-5-5/

## Summary

A worked cost model for agentic sessions in [[claude-code|Claude Code]], priced at **Claude Opus 5.5** list prices. Its thesis is that cost per *task*, not per token, is what matters, and that "a retry costs more than those savings". The cost is driven by turns (each one resends the conversation), cache-hit rate, output tokens (thinking included) and model choice. The prices are Anthropic's list prices. Most of the dollar figures are **illustrations the author built from those prices**, and the post says to check "your own math". The one empirical result is an internal benchmark.

## Key points

- **List prices (per 1M tokens)**: Opus 5.5 costs $4 input, $20 output and $0.20 for a cache read. Input and output are "20% cheaper than on Opus 5" and cache reads "60% cheaper". The cache-read rate falls from a tenth of input to a twentieth. Cache writes cost 1.25× input (5-minute cache) or 2× input (1-hour cache). Fable 5.1 costs $10 / $50 / $0.25.
- **Turns dominate (illustration)**: a task that grows from 20K to 120K of context over 40 turns processes about 2.8M input tokens. At 90% cache hits that costs about $1.62. The same task in 25 turns costs about $1.02. With no cache it costs $11.20, and at 96% hits about $0.99.
- **Output is expensive**: an output token costs 100× a cache read. Thinking is billed as output "even when Claude Code only shows you a summary".
- **"40% less" decoded**: the headline is "our estimate for typical workloads billed by token, at default settings". It assumes fewer tokens per task at the `medium` default on top of the price cut, so "it isn't a 40% cut to the price of a token". On identical tokens, the illustrative session costs $3.50 on Opus 5 and $2.40 on Opus 5.5 (−31%).
- **Effort**: Opus 5.5 has "four levels (low, medium, high and xhigh), plus max for a single session". The default is `medium`, against `high` for Opus 5. At a given level Opus 5.5 "thinks more per turn than Opus 5", so old settings shouldn't be carried over. Changing effort keeps the cache on the API and subscriptions, but clears it on Bedrock, Google Cloud and gateways.
- **Internal benchmark (Anthropic-reported)**: a migration from Opus 4.8 to Opus 5.5 at low effort cut cost by about 18% on a 44-ticket customer-support benchmark. A `/claude-api prompt-audit` pass cut a further 9%, to about 25% below the starting point. The post itself says "treat it as an example".
- **Cache economics**: at 120K context, a 5-minute write costs about $0.60 and a read about $0.02, so "a six-minute coffee break" turns a read into a write. Compacting at 150K costs about $0.25 and pays back in about 10 turns, but only if the cache is warm.
- **Fast mode**: "up to 2.5 times faster, at twice the standard price" ($8 / $40). Agent teams in plan mode use about 7× the tokens of a standard session. A Claude Code docs baseline puts typical spend at about $13 per developer per active day, and under $30 for 90% of users.

## Concepts & entities

- [[the-harness-effect-token-economics-writer-2026]] is the academic counterpart: cost is price × tokens, and the harness sets the token term. That paper models cached input at "roughly a tenth" of list price, while Opus 5.5's read rate is a twentieth.
- [[prefix-caching]] · [[kv-cache]]: prefix matching, and what breaks the cache (tool, system-prompt, model or MCP changes, compaction, pauses).
- [[reasoning-effort]] · [[adaptive-thinking]] · [[inference-time-compute]]: effort as the main cost lever after model choice.
- [[multi-agent]]: subagent model inheritance and agent-team token multiples. [[context-assembly]]: `CLAUDE.md` under 200 lines, deferred MCP tool loading.
- [[claude]] · [[claude-code]] · [[anthropic]] · [[same-task-different-work-weinberger-2026]] (measured cost-per-success spread under Claude Code).

## What it does not show

- No measured Opus 5 vs Opus 5.5 work-per-task delta. The calculator's "token change" slider defaults to 0%, and readers are told to measure it themselves.
- The 18% and 9% savings come from one internal benchmark that has not been released.
- Subscription "about 25% further" limits are asserted, not shown.

## References

- https://claude.dev/blog/what-a-task-costs-on-opus-5-5/
- Companions: [[getting-the-most-out-of-opus-5-5-osmani-2026]] · [[building-with-claude-sonnet-5-5-osmani-2026]]
- _Original source: `sources/what-a-task-costs-on-opus-5-5-osmani-2026.md`_
