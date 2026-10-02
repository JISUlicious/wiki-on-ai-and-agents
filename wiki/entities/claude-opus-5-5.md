---
title: Claude Opus 5.5
type: entity
created: 2026-10-02
updated: 2026-10-02
sources:
  - getting-the-most-out-of-opus-5-5-osmani-2026.md
  - what-a-task-costs-on-opus-5-5-osmani-2026.md
  - building-with-claude-sonnet-5-5-osmani-2026.md
family: claude
predecessor: [[claude]]
successors: []
release_date: 2026-09
status: draft
importance: medium
tags:
  - model
---

# Claude Opus 5.5

[[anthropic|Anthropic]]'s flagship model in the Claude 5.5 family, and the first model of that family, released in September 2026 ahead of [[claude-sonnet-5-5|Sonnet 5.5]]. It is the `default` model in [[claude-code|Claude Code]], which needs v2.1.280 or later to run it. Every figure below is Anthropic-reported, from its claude.dev developer posts.

## Spec, as published

| | Opus 5.5 | Change vs Opus 5 |
|---|---|---|
| Input | $4 / M tokens | −20% |
| Output | $20 / M tokens | −20% |
| Cache read | $0.20 / M tokens | −60%: a twentieth of input price, down from a tenth |
| Cache write (5 min / 1 h) | $5 / $8 per M | — |
| Fast mode (research preview) | up to 2.5× faster at 2× price ($8 / $40) | new at launch |

The posts do not state its context window, maximum output, knowledge cutoff or API model string.

**Thinking is always on**: the model decides how much to think before every reply, and there is no setting to disable it. **[[reasoning-effort|Effort]]** has four levels (low, medium, high, xhigh) plus `max` for a single session. **The default is `medium`, one level below Opus 5's `high`**, and at a given level Opus 5.5 thinks more per turn than Opus 5. Changing effort keeps the prompt cache on the API but clears it on Bedrock, Google Cloud and gateways.

It is described as the first Opus model to launch with Fable-level biology and cyber safeguards; flagged messages are moved to an older model.

## What "40% cheaper" means

Anthropic says Opus 5.5 "costs 40% less to run than Opus 5", and the cost post decodes that carefully: it is "our estimate for typical workloads billed by token, at default settings", not a 40% cut in the price of a token. The per-token price cut alone is worth −31% on the post's illustrative session. The rest comes from the lower default effort and cheaper cache reads, which dominate agentic sessions. The only empirical datapoint is one internal 44-ticket benchmark: −18% from moving Opus 4.8 to Opus 5.5 at low effort. See [[what-a-task-costs-on-opus-5-5-osmani-2026]].

## How Anthropic says to prompt it

Say what "done" looks like and let it run. Delete "think carefully" instructions, since thinking is automatic. Put stop rules in `CLAUDE.md`, and fan work out to subagents. Start at `medium` effort and keep `xhigh`/`max` for work where a gain has been measured. All of this is advice, not measurement. See [[getting-the-most-out-of-opus-5-5-osmani-2026]].

## Lineage

Successor to Claude Opus 5 (no page yet), which every comparison in the posts uses as the baseline. Earlier in the Opus line: [[claude-opus-4-8]]. The change from Opus 5 is mostly economic: cheaper tokens, much cheaper cache reads and a lower default effort. Thinking is now always on, and capability claims are qualitative ("caught more bugs than Opus 5 at high effort" even at its lowest effort, per early testers).

> [!note] Effort guidance has changed since the 4.x pages
> [[reasoning-effort]] records Opus 4.8-era defaults (`high` everywhere, start coding at `xhigh`). For the 5.5 models the default is `medium`, and Anthropic now recommends reserving `xhigh` for measured gains.

## Related

- [[claude]] · [[claude-sonnet-5-5]] · [[claude-code]] · [[reasoning-effort]] · [[prefix-caching]] · [[claude-opus-4-8]] · [[anthropic]]

## References

- [[getting-the-most-out-of-opus-5-5-osmani-2026]] · [[what-a-task-costs-on-opus-5-5-osmani-2026]] · [[building-with-claude-sonnet-5-5-osmani-2026]]
