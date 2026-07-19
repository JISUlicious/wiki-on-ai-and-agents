---
title: Direct Feedback Alignment (DFA)
type: concept
created: 2026-07-07
updated: 2026-07-07
sources:
  - diffusing-blame-yamada-2026.md
builds-on:
  - [[backprop-free-learning]]
status: draft
importance: low
---

# Direct Feedback Alignment (DFA)

**Direct Feedback Alignment** trains a network without [[backpropagation]] by broadcasting the **output error directly to every hidden layer through fixed random matrices**, rather than propagating it layer-by-layer through transposed forward weights. It sidesteps the weight-transport problem: the feedback pathway is random and never updated, and the forward weights learn to *align* with it.

Because the error reaches all layers simultaneously, DFA also removes backward locking — layers can update in parallel.

## As a baseline

DFA is the standard strong baseline for [[backprop-free-learning]]. From [[diffusing-blame-yamada-2026]]:

| | MNIST | CIFAR-10 | Craftax (open-ended RL) |
|---|---|---|---|
| **DFA** | **97.6%** | **69.1%** | **weakest** of the compared methods |
| Error Diffusion (Dale-compliant) | 96.7% | 61.7% | beats DFA-PPO |

DFA outperforms Dale-compliant Error Diffusion on supervised classification — but it **violates [[dales-principle|Dale's principle]]**, requiring millions of negative weights, and it is the *worst* method on Craftax. The authors read this as evidence that **random feedback pathways that suffice for supervised learning can fail on complex, open-ended RL**.

## Related

- [[backprop-free-learning]] — the family, including feedback alignment and Error Diffusion.
- [[backpropagation]] — the exact-gradient baseline.
- [[dales-principle]] — the constraint DFA does *not* satisfy.

## References

- [[diffusing-blame-yamada-2026]] — cross-domain DFA comparison.
