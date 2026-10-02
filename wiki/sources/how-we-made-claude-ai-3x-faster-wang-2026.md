---
title: "How we made claude.ai 3x faster in two weeks"
type: source
created: 2026-10-02
updated: 2026-10-02
sources:
  - how-we-made-claude-ai-3x-faster-wang-2026.md
authors:
  - Raymond Wang
  - Sam Attard
  - Issac G.
year: 2026
venue: "claude.dev (Anthropic developer blog)"
status: complete
importance: medium
tags:
  - 2026
---

# How we made claude.ai 3x faster in two weeks

**Raymond Wang, Sam Attard, and Issac G.** ([[anthropic|Anthropic]]), with contributions from eight other engineers. Published 2026-09-23 on claude.dev.
**URL**: https://claude.dev/blog/how-we-made-claude-ai-faster/

## Summary

A case study of a two-week sprint in August 2026 to speed up the **client side** of claude.ai and the Claude desktop app. The work was done mostly by Claude, running in a single Slack channel through Claude Tag (beta) on an internal research model the authors call "roughly comparable to Opus 5.5". Humans set goals, made tradeoffs and approved every change. The central lesson is that **measuring something makes it tractable for Claude**. Once there is a number to beat, Claude can hill-climb it, so the most useful human work was finding new things to measure. This is a frontend and web-performance post. It says nothing about model serving or inference latency.

## Key points

All figures are **Anthropic-reported** and come from Anthropic's own real-user monitoring. There is no external check.

- **Headline (p75, Aug 13 vs Aug 27):** time to a typeable page on a fresh claude.ai load went from 3.1 s to 0.55 s. A new [[claude-code|Claude Code]] session went from 0.8 s to 0.3 s, and a Cowork cloud session from 2.6 s to 0.73 s. These four journeys cover 95% of user activity (13 measurements across platforms). Anthropic estimates "tens of thousands of user-hours" of waiting saved per day.
- **Throughput:** more than 3,000 changes merged with no customer-facing incident or rollback. Up to ~150 threads ran at once and more than 200 changes landed on the busiest days. Twelve of the thirteen initial targets were hit by day three.
- **Deterministic proxies over wall-clock time.** Five benchmark threads were started within 11 minutes: Valgrind instruction counts (`node --predictable`), V8 call counts, React commits, style recalculations and DOM mutations. Each proxy had to prove it tracked wall-clock time before it was kept. On two hot paths, instructions fell 48% and 31% and wall-clock time fell 78% and 44%.
- **Ratchets.** Each validated benchmark became a CI gate that can only go down. A PR that raises the count fails, and a daily job lowers the ceiling.
- **The loop:** a human opens a thread, Claude builds a benchmark that shows the problem, ships PRs behind feature flags, watches the deploy and field data, then ratchets the win or rolls the flag back. About a third of PRs added telemetry or guardrails.
- **Example finds:** a Layout Instability event showed that 31% of page loads shifted after the page was usable, though CLS scored only ~0.008. Claude also found 6,900 hooks on the typing path, a `:root:has()` selector adding 24 ms per DOM change, and half a million hidden reloads a day. Em dashes forced V8 strings into UTF-16, which slowed regex highlighting.
- **120 Hz streaming:** about 60 PRs in one thread cut main-thread blocking on long replies from ~750 ms to ~200 ms. Long replies now hold 120 fps on a 120 Hz MacBook.
- **Guardrails:** automated review plus at least one human approval, tests before optimizations, short-lived flags (~200 created, more than half retired), and staged rollouts.
- **Advice, not measurement:** steering needs *ambition* (Claude pads estimates and hedges by default), *taste* (humans rule on visible tradeoffs) and *direction* (narrow threads, sequencing).

## Concepts & entities

- [[claude-code]] · [[claude]] · [[anthropic]] — the products and publisher.
- [[agent-evaluation]] / [[benchmark]] — the "validate the proxy before you climb it" rule applies to evals in general.
- [[reward-hacking]] — the stated reason for dropping proxies that do not correlate with user latency.
- [[multi-agent]] · [[mcp]] — parallel threads, and Datadog accessed through MCP.

## What it does not show

- No control: the speedup cannot be split between what Claude did and what the engineers would have done anyway.
- The model is unreleased ("roughly comparable to Opus 5.5"), so the result cannot be reproduced.
- Only p75 is reported. The authors say p95 and very long conversations still need work.
- Nothing about server-side or model-inference latency. It is not evidence about [[kv-cache]] or [[prefix-caching]].

## References

- https://claude.dev/blog/how-we-made-claude-ai-faster/
- _Original source: `sources/how-we-made-claude-ai-3x-faster-wang-2026.md`_
