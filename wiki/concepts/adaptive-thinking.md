---
title: Adaptive Thinking
type: concept
created: 2026-07-28
updated: 2026-07-28
sources:
  - claude-effort-and-ultracode-docs-2026.md
status: draft
importance: medium
---

# Adaptive Thinking

**Adaptive thinking** is a model deciding *how much* to think per request, rather than spending a fixed budget on everything. Easy prompts should return quickly; hard ones should get extended reasoning — and ideally the model makes that call itself instead of the caller guessing.

It is the behaviour that [[reasoning-effort|`effort`]] controls. Per [[claude-effort-and-ultracode-docs-2026|Anthropic's documentation]], for Opus 4.8 / 4.7 / 4.6 and Sonnet 4.6, **`effort` is the recommended control for adaptive-thinking depth**, replacing the older `budget_tokens` knob. The shift is meaningful: `budget_tokens` is an explicit token allowance the caller must size, whereas `effort` expresses *how eagerly the model should spend* and leaves the allocation to the model.

## Why it matters

Fixed thinking budgets are wrong in both directions — wasteful on easy inputs, truncating on hard ones. Adaptive allocation is the inference-time analogue of the [[test-time-compute]] scaling story: given that more thinking buys accuracy, the useful question becomes *where to spend it*, not *whether to*.

There is a measured cost to getting it wrong. [[quantization-hurts-reasoning-liu-2025|Liu et al.]] observed aggressive quantization causing small reasoning models to produce **longer** outputs while scoring worse — thinking more and achieving less, which is the failure adaptive control is meant to avoid.

> [!note] Draft
> Created during a lint pass — `[[adaptive-thinking]]` was a recurring dangling link across [[reasoning-effort]], [[claude-opus-4-8]] and the effort docs. Worth expanding with the reasoning-model literature on self-determined stopping, and with the "knowing when to stop" strand of [[metacognition-in-llms-liu-2026|metacognition]], which frames it as one facet of a monitor-and-control loop.

## Related

- [[reasoning-effort]] — the concrete parameter that exposes it.
- [[chain-of-thought]] · [[test-time-compute]] — what is being budgeted.
- [[metacognition-in-llms-liu-2026]] — "knowing when to stop" as a metacognitive capability.
- [[claude-opus-4-8]] — supports the full `effort` range including `xhigh` and `max`.

## References

- [[claude-effort-and-ultracode-docs-2026]] — the authoritative `effort` documentation.
