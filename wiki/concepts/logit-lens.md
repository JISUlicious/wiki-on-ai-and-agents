---
title: Logit Lens
type: concept
created: 2026-07-07
updated: 2026-10-27
sources:
  - global-workspace-in-language-models-anthropic-2026.md
  - neuron-explorer-lyramakesmusic-2026.md
builds-on:
  - [[residual-stream]]
status: draft
importance: medium
---

# Logit Lens

The **logit lens** is the simplest technique for reading a transformer's [[residual-stream|residual stream]] *mid-computation*: take the stream at some intermediate layer ℓ and apply the model's own final normalization and unembedding matrix to it, as if the model were stopping there.

```
logit_lens(h_ℓ) = softmax(W_U · norm(h_ℓ))
```

This yields a ranked list of vocabulary tokens for any intermediate activation — a rough answer to "what would the model say if it had to answer now?" Applied across layers, it shows a prediction sharpening with depth.

## The core assumption (and its failure mode)

The logit lens implicitly assumes the map from layer ℓ to the final layer is the **identity** — i.e. that representations **use the same coordinates in every layer**. That approximation holds reasonably in late layers (the stream is already close to its final form) and **degrades badly in earlier layers**.

Concretely, in [[global-workspace-in-language-models-anthropic-2026]]'s comparison on a multi-hop prompt, the logit lens produces uninterpretable fragments (`vah`, `valea`, `general`) **below layer 58**, while the [[global-workspace-theory|J-lens]] cleanly recovers the intermediate reasoning chain over the same range.

## Successors

| Technique | Map from layer ℓ → final | Derived by |
|---|---|---|
| **Logit lens** | identity (assumed) | nothing — just `W_U` |
| **Tuned lens** | learned affine map | training a probe per layer |
| **[[global-workspace-theory\|Jacobian lens (J-lens)]]** | averaged Jacobian `J_ℓ` | backprop through the frozen model, averaged over a corpus |

The J-lens is described by its authors as "a principled refinement of the logit lens" — it replaces the identity assumption with an empirically measured linear map, correcting for representational drift across layers, and uniquely includes effects on **future** token positions.

## Steer lens (2026)

[[neuron-explorer]] pairs the logit lens of a neuron's output column with a **steer lens**. The steer lens pushes the neuron ±0.8 (in units of the residual norm), measures the change in next-token log-probability over a few contexts, and so includes indirect paths that the logit lens misses.
- **Example:** for Qwen3.5-4B L22 N5344, the logit lens shows only "indicator", while the steer lens surfaces "responsible".
- **Wreckage matches the lens:** the token "wreckage" a neuron produces at very high dose matches its logit-lens readout ([[neuron-explorer-lyramakesmusic-2026]]).

## Related

- [[residual-stream]] — the object all lenses read.
- [[global-workspace-theory]] — the J-lens and the J-space it reveals.
- [[mechanistic-interpretability]] · [[sparse-autoencoder]] — alternative proposals for which directions to read.

## References

- [[global-workspace-in-language-models-anthropic-2026]] — the logit-lens comparison and the refinement argument.
- [[neuron-explorer-lyramakesmusic-2026]] — steer lens vs logit lens on single neurons
