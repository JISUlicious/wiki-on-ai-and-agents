---
title: "Building with Claude Sonnet 5.5"
type: source
created: 2026-10-02
updated: 2026-10-02
sources:
  - building-with-claude-sonnet-5-5-osmani-2026.md
authors:
  - Addy Osmani
year: 2026
venue: "claude.dev (Anthropic developer blog)"
publisher: Anthropic
status: complete
importance: medium
tags:
  - 2026
---

# Building with Claude Sonnet 5.5

**Source**: `sources/building-with-claude-sonnet-5-5-osmani-2026.md` (official [[anthropic|Anthropic]] developer blog)
**Author**: Addy Osmani (Anthropic)
**Date**: 2026-09-28
**URL**: https://claude.dev/blog/building-with-claude-sonnet-5-5/

## Summary

The developer launch guide for **Claude Sonnet 5.5**, the second [[claude|Claude]] 5.5 model after Opus 5.5. It covers when to pick Sonnet over Opus, pricing, the model spec sheet, six migration changes from Sonnet 5, effort tuning, and refusal handling. It is a vendor post about the vendor's own model. Every speed, cost and quality claim below is **Anthropic-reported** and has no independent measurement behind it.

## Key points

- **Positioning (Anthropic's claims)**: "smarter, more efficient and 30% faster" than Sonnet 5. Per-token price is unchanged, and the post says it "costs up to 30% less for most work" because it uses fewer tokens per task. Sonnet is for "well-scoped everyday tasks", while Opus 5.5 is for "complex work requiring careful judgment". Claude Haiku 5.5 is announced for "the coming weeks".
- **Pricing (list, per 1M tokens)**: Sonnet 5.5 costs $2 input, $10 output, $2.50 for a 5-minute cache write, $4 for a 1-hour cache write and $0.20 for a cache read. The same table lists Opus 5.5 at $4 / $20 / $5 / $8 / $0.20. US-only inference costs 1.1× standard.
- **Spec**: 1M-token context window ("native, no beta header"). Max output is 128k, or 300k on the Batches API with a beta header. Knowledge cutoff is June 2026. Thinking is on by default ([[adaptive-thinking]]). Effort levels are `low`/`medium`/`high`/`xhigh`/`max`, defaulting to `high` on the API and `medium` in [[claude-code|Claude Code]]. The minimum cacheable prompt is 512 tokens, down from 1,024 on Sonnet 5.
- **Breaking changes**:
  - `thinking: disabled` now returns a 400 error. It is replaced by a new `between_tools` mode, which thinks only between tool calls and works only at low, medium or high effort.
  - Forced `tool_choice` (`any`/`tool`) now returns a 400. Use `auto` plus `strict: true` tools instead.
  - Thinking blocks are model-bound, and only Sonnet 5.5 can read its own.
  - Computer use moves to `computer_toolset_20260801`.
  - The advisor tool rejects Opus 4.8, Opus 4.7 and Sonnet 5 as advisors.
- **Tuning (advice)**: effort levels are "recalibrated", so old settings don't carry over. The post recommends starting at `medium` for well-specified agentic coding and keeping `xhigh`/`max` for cases where evals show a gain. Asking in the prompt for less thinking "doesn't reliably reduce it". Changing top-level effort invalidates the cache, but per-message effort (beta) does not. Old "do not be lazy" style workarounds should be removed.
- **Safeguards**: refusals return HTTP 200 with `stop_reason: "refusal"` and one of five categories (`cyber`, `bio`, `frontier_llm`, `reasoning_extraction`, `general_harms`). Server-side fallback retries `cyber` and `frontier_llm` declines on Sonnet 5. The post calls it "the first Sonnet model with cybersecurity safeguards similar to those on our most capable models".
- **Image cost**: a 2000×1500 image costs "about 2.5 times as many tokens" as on Sonnet 4.6, Sonnet 4.5 or Haiku 4.5, because of the high-resolution image tier.

## Concepts & entities

- [[claude]] (family) · [[anthropic]] (publisher) · [[claude-code]] (`sonnet` alias, `medium` default, no fast mode).
- [[reasoning-effort]] · [[adaptive-thinking]] · [[inference-time-compute]]: effort is the thinking-depth control, and `between_tools` is a new thinking mode.
- [[prefix-caching]] · [[kv-cache]]: the 512-token cache floor, cache reads at a tenth of input, and effort changes breaking the cache.
- [[tool-use]] · [[structured-outputs]] (strict tools) · [[computer-use-agent]] · [[long-context-llm]] (1M native).
- [[ai-enabled-cyber-operations]] · [[illicit-distillation]]: the `cyber`, `frontier_llm` and `reasoning_extraction` refusal categories.

## What it does not show

- No benchmark tables. The "30% faster", "up to 30% less" and "fewer tokens" claims come with no methodology or task set. The only third-party evidence is a customer quote from Epic Games.
- No comparison with Opus 5.5 beyond a routing table and the code-to-painting demo images.
- The post gives no alignment-audit numbers. It only says Sonnet 5.5 "improves on or matches Sonnet 5 on most measures".

## References

- https://claude.dev/blog/building-with-claude-sonnet-5-5/
- Companions: [[getting-the-most-out-of-opus-5-5-osmani-2026]] · [[what-a-task-costs-on-opus-5-5-osmani-2026]]
- _Original source: `sources/building-with-claude-sonnet-5-5-osmani-2026.md`_
