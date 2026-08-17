---
title: Attention Sink
type: concept
created: 2026-08-17
updated: 2026-08-17
sources:
  - cracks-in-the-foundation-long-context-extension-bertsch-2026.md
status: draft
importance: medium
---

# Attention Sink

The phenomenon where a transformer devotes a large, near-constant share of [[attention-mechanism|attention]] mass to a few early tokens — typically the very first — regardless of their semantic content. The leading explanation is that [[scaled-dot-product-attention|softmax]] forces the attention weights to sum to one, so a head with nothing to attend to must dump its mass somewhere; the first token becomes the default drain because every position can see it.

Sinks are why naively evicting the earliest tokens from a [[kv-cache|KV cache]] degrades a model far more than their information content suggests, and why streaming-attention schemes pin the first few tokens permanently.

## A sign inversion worth noting

Prior work generally treats sinks as a pathology to be suppressed. [[cracks-in-the-foundation-long-context-extension-bertsch-2026|Bertsch et al.]] report the opposite correlation on their 26-model 7B sweep: **attention sinks correlate *positively* with long-context performance (R² = 0.38)**, and QK-norm — one of the four architectural choices they find damaging for context extension — suppresses both attention entropy and sinks together.

That does not settle the mechanism; it is a correlation across 26 models sharing a training recipe, and sink formation and context extensibility may both descend from a third property. But it is direct evidence against reading sink suppression as automatically good, and it means an architectural change that removes sinks should be evaluated on post-extension long-context tasks, not on loss.

## Related

- [[attention-mechanism]] · [[scaled-dot-product-attention]] · [[multi-head-attention]]
- [[layer-normalization]] — QK-norm is the variant implicated above.
- [[kv-cache]] · [[long-context-llm]] · [[length-generalization]]
- [[superposition]] · [[mechanistic-interpretability]] — the interpretability reading of "spare" attention mass.

## References

- [[cracks-in-the-foundation-long-context-extension-bertsch-2026]]
