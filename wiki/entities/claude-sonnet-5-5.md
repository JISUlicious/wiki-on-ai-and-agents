---
title: Claude Sonnet 5.5
type: entity
created: 2026-10-02
updated: 2026-10-02
sources:
  - building-with-claude-sonnet-5-5-osmani-2026.md
family: claude
predecessor: [[claude]]
successors: []
release_date: 2026-09-28
status: draft
importance: medium
tags:
  - model
---

# Claude Sonnet 5.5

The second model in [[anthropic|Anthropic]]'s Claude 5.5 family, released 2026-09-28 after [[claude-opus-5-5|Opus 5.5]]. Claude Haiku 5.5 is announced for "the coming weeks". Every figure below is Anthropic-reported, from the claude.dev launch post.

## Spec, as published

| | Sonnet 5.5 |
|---|---|
| Model ID | `claude-sonnet-5-5` (Claude API, AWS, Google Cloud, Microsoft Foundry); `anthropic.claude-sonnet-5-5` on Bedrock |
| Price per M tokens | $2 input · $10 output · $0.20 cache read · $2.50 / $4 cache write (5 min / 1 h), unchanged from Sonnet 5 |
| Context window | 1M tokens, native, no beta header |
| Max output | 128k (300k on the Batches API with a beta header) |
| Knowledge cutoff | June 2026 |
| Minimum cacheable prompt | 512 tokens (1,024 on Sonnet 5) |
| Effort levels | low, medium, high, xhigh, max; default `high` on the API, `medium` in Claude Code |
| Thinking | adaptive and on by default; cannot be disabled (`disabled` returns a 400) |

The headline claims are "30% faster" and "costs up to 30% less", both at an unchanged per-token price. The saving comes from using fewer tokens. No methodology is published; the only outside evidence is a customer quote.

## Migration from Sonnet 5

Six breaking changes:
- `thinking: disabled` returns a 400; the replacement is `between_tools`, which only works at low to high effort.
- Forced `tool_choice` returns a 400.
- Thinking blocks are bound to the model that produced them.
- Computer use moves into the toolset.
- The advisor tool rejects some older models as advisors.
- Notes between tool calls arrive in `thinking` blocks.

It is the first Sonnet model with cybersecurity safeguards like those on the most capable models. It has five refusal categories, and some declines fall back to Sonnet 5 server-side.

In [[claude-code|Claude Code]], from v2.1.284 the `sonnet` alias resolves to Sonnet 5.5, at `medium` effort. It has no fast mode, and the `default` model stays Opus 5.5.

## Lineage

Successor to Claude Sonnet 5 (no page yet), with the same tokenizer and prices. The changes are speed, efficiency, recalibrated effort levels (old settings do not carry over), a lower minimum cacheable prompt, and stricter API semantics around thinking and tool choice. Earlier in the Sonnet line: [[claude-3-5-sonnet]].

## Related

- [[claude]] · [[claude-opus-5-5]] · [[claude-code]] · [[reasoning-effort]] · [[prefix-caching]] · [[anthropic]]

## References

- [[building-with-claude-sonnet-5-5-osmani-2026]]
