---
title: Backpropagation
type: concept
created: 2026-07-07
updated: 2026-07-07
sources:
  - diffusing-blame-yamada-2026.md
status: draft
importance: high
---

# Backpropagation

**Backpropagation** is the algorithm that computes gradients of a loss with respect to every parameter in a neural network, by applying the chain rule backward through the computation graph. It is the engine behind essentially all modern deep learning, including every model in this wiki — [[transformer-architecture|transformers]] are trained with it, and so is every fine-tuning and [[rlhf]] stage layered on top.

## Mechanism

1. **Forward pass** — inputs flow through the network; each layer's activations are cached.
2. **Loss** — compare output to target.
3. **Backward pass** — propagate the error backward. Each layer's error is obtained by multiplying the downstream error by the **transpose of that layer's forward weights** (`Wᵀ`), then gating by the local activation derivative.
4. **Update** — each weight's gradient is (local error × presynaptic activation); an optimizer applies it.

The key property is **exact credit assignment**: every parameter receives a gradient reflecting its true contribution to the loss.

## Known difficulties

- **Vanishing / exploding gradients** — repeated multiplication through depth shrinks or blows up the signal. Mitigated by [[residual-stream|residual connections]], [[layer-normalization|normalization]], and better activations.
- **Sequential backward pass** — layers must wait for downstream errors, limiting parallelism (*backward locking*).
- **Memory** — activations must be cached for the backward pass.

## The biological-plausibility objection

Backprop requires each synapse to know the weight of a *different, forward* synapse in order to use `Wᵀ` — the **weight transport problem**. Biological neurons have no such mechanism, and backprop additionally requires a precisely timed global backward phase. This objection motivates the entire [[backprop-free-learning]] literature (feedback alignment, [[direct-feedback-alignment|DFA]], Error Diffusion), which trades exact gradients for locally available learning signals. Those methods work but currently pay an accuracy tax that **grows with task difficulty** — see [[diffusing-blame-yamada-2026]].

> [!note] Draft
> Created as a gap-fill during the [[diffusing-blame-yamada-2026]] ingest, which is centrally about learning *without* backpropagation. Deserves expansion with its history (Rumelhart, Hinton & Williams 1986), autodiff, and the optimizer layer.

## Related

- [[backprop-free-learning]] · [[direct-feedback-alignment]] · [[dales-principle]] — the biologically motivated alternatives.
- [[residual-stream]] · [[layer-normalization]] — architectural features that make deep backprop trainable.

## References

- [[diffusing-blame-yamada-2026]] — the weight-transport framing used here.
