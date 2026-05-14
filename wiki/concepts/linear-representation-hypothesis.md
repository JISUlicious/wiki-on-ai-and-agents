---
title: Linear Representation Hypothesis
type: concept
created: 2026-05-14
updated: 2026-05-14
sources:
  - turner-activation-engineering-2023.md
  - representation-engineering-zou-2023.md
  - emotion-concepts-anthropic-2026.md
status: complete
importance: high
---

# Linear Representation Hypothesis (LRH)

The hypothesis that **concepts in trained neural networks (especially Transformers) are encoded as approximately linear directions in activation space**, such that vector arithmetic over those directions produces predictable changes in the network's behavior.

The LRH is the theoretical bedrock of the modern [[steering]] toolkit. It's implicit in [[word-embeddings|word2vec]] (`king − man + woman ≈ queen`), explicit in [[activation-engineering-turner-2023|ActAdd]] (steering vectors as concept directions), foundational to [[representation-engineering|RepE]] ("linear artificial tomography"), justifies [[sparse-autoencoder|SAEs]] (decompose into a linear feature basis), and is the operating assumption of [[contrastive-activation-addition|CAA]], [[inference-time-intervention|ITI]], and the [[emotion-concepts-anthropic-2026|emotion-vectors paper]].

## What the LRH claims

For a concept `c` (truthfulness, an emotion, a topic, a behavioral disposition):

- There exists a direction `v_c` in the model's activation space such that the projection `h · v_c` correlates with the degree to which the input/output "is about" `c`.
- Adding `v_c` to `h` (suitably scaled) increases `c`-ness in the model's outputs.
- Subtracting it decreases `c`-ness.
- Roughly, these operations compose: stacking `v_c1 + v_c2` produces the conjunction.

## Why it's *approximately* linear

Strictly linear representations would be a miracle. The empirical claim is that **at the granularity of high-level concepts** (semantic, behavioral), the dominant variation is well-approximated by a linear subspace, with non-linear interactions being secondary. This works because:

- Transformers use a lot of linear projection (Q/K/V matrices, MLP up/down).
- Residual streams have an additive structure.
- Training pressure pushes toward sparse, near-orthogonal feature encoding ([[superposition]]).

The hypothesis breaks down at fine granularity (some concepts are non-linearly encoded, especially compositional ones) and at extreme activations.

## Evidence

| Source | Evidence |
|---|---|
| [[efficient-estimation-of-word-representations-mikolov-2013\|word2vec]] (2013) | Static word embeddings exhibit linear analogies. The original demonstration. |
| Probing literature (2018–2022) | Linear classifiers recover syntactic / semantic features from intermediate Transformer activations. |
| [[activation-engineering-turner-2023\|ActAdd]] (2023) | Linear vector additions to residual stream produce predictable behavior changes. |
| [[representation-engineering-zou-2023\|RepE]] (2023) | Wide range of safety-relevant concepts admit linear-readout + linear-write probes. |
| [[contrastive-activation-addition-rimsky-2023\|CAA]] (2023) | Behavioral dispositions (corrigibility, sycophancy) are linearly steerable. |
| [[sparse-autoencoders-cunningham-2023\|SAE]] features (2023+) | Unsupervised linear decomposition recovers thousands of interpretable features. |
| [[emotion-concepts-anthropic-2026\|Emotion concepts]] (2026) | Emotion directions in Claude Sonnet 4.5 are linear, geometric, and causal. |

## Limitations

- **Not perfectly linear**: composed concepts and edge cases break linearity.
- **Direction non-uniqueness**: multiple directions may encode related variants of "the same" concept. SAE features split into surprising sub-concepts.
- **Off-target correlations**: a "truthfulness" direction may correlate with surface features (formal language, hedging). Pure linear interventions may not surgically isolate the target concept.

## Why it matters

If concepts weren't linearly encoded, **the entire [[steering]] paradigm wouldn't work**. The fact that simple vector arithmetic in activation space produces clean behavioral changes is the empirical miracle the field is built on.

## References

- [[activation-engineering-turner-2023]]
- [[representation-engineering-zou-2023]]
- [[sparse-autoencoders-cunningham-2023]]
- [[contrastive-activation-addition-rimsky-2023]]
- [[inference-time-intervention-li-2023]]
- [[emotion-concepts-anthropic-2026]]
