---
title: Latent Reasoning
type: concept
created: 2026-05-20
updated: 2026-05-20
sources:
  - latent-thoughts-looped-transformers-saunshi-2025.md
  - cot-or-loop-xu-2025.md
  - loop-think-generalize-2026.md
status: complete
importance: high
tags:
  - 2025
---

**Latent reasoning** is reasoning via internal iteration of a [[looped-transformer|looped transformer's]] hidden states, without emitting explicit [[chain-of-thought-prompting|Chain-of-Thought]] tokens. The reasoning happens inside the model — each loop iteration is a "thought" carried in the embedding space rather than serialized to the token stream. Saunshi et al. (2025) coined the **"latent thoughts"** framing: where CoT emits one discrete thought token per autoregressive step, a looped transformer can emit multiple high-dimensional latent thoughts per iteration, and `T` loops can in theory simulate `T` steps of CoT. It is the depth-recursion counterpart to token-recursion (CoT), and the two win on different task classes (Xu & Sato 2025).

## Core mechanism

- Depth-recursion as silent CoT: extra reasoning happens inside the model across loop iterations; no extra tokens are generated and no scratchpad is exposed.
- A `k`-layer transformer looped `L` times approaches a `kL`-layer non-looped model on reasoning tasks (n-ary addition, p-hop induction, i-GSM math) while emitting nothing extra (Saunshi 2025; iso-FLOP / iso-param decomposition).
- Provably can simulate explicit CoT for some task classes: a looped transformer with a scratchpad simulates `T` steps of CoT with `T` loops (Saunshi 2025, §5.4), and Xu & Sato 2025 give a tight formal complexity-class characterization `LOOP[T, d, p]` ↔ threshold circuits of depth `T`.
- [[loop-think-generalize-2026|Loop, Think, Generalize]] (2026) shows tail-only supervision elicits implicit multi-hop reasoning + classic [[grokking]] dynamics — the model transitions from memorization to in-distribution generalization to systematic generalization across ~10⁴ training epochs, with logit-lens analysis confirming bridge entities become readable at intermediate depths only after the grokking transition.
- The depth-recursion counterpart to token-recursion (CoT): per Xu & Sato 2025, depth-recursion wins on parallel-deterministic tasks (linear-equation solving, bipartite matching, CFL membership) while CoT-with-sampling wins on compositional-stochastic tasks (approximate counting, SAT-like search) — different inference-compute axes with different sweet spots.

## What it advances

Latent reasoning legitimizes reasoning in continuous space as a first-class alternative to emitted-token reasoning. It decouples reasoning compute from generation length: a single forward pass at high `R` can do multi-hop traversal that would otherwise require dozens of generated CoT tokens. This matters operationally for inference-time-compute scaling (no autoregressive bottleneck, no scratchpad memory cost, no risk of token-level leakage of internal reasoning) and architecturally as the motivation for the entire [[recurrent-depth-transformer]] pretraining program. Saunshi 2025's empirical isoplots show the reasoning gains persist even when looped models have worse perplexity than parameter-matched baselines — i.e., latent reasoning is a distinct inductive bias, not a capacity artifact.

## Relation to other concepts

[[chain-of-thought-prompting]] is the token-recursion sibling; latent reasoning is the depth-recursion alternative. [[looped-transformer]] is the architectural family that makes latent reasoning operational; [[recurrent-depth-transformer]] is its pretraining-scale incarnation. [[inference-time-compute]] is the broader axis both CoT and latent reasoning sit on; the two are complementary, not substitutes (Xu & Sato 2025). [[deepseek-r1]] and the o1 lineage represent the RL-reasoning thread that latent reasoning competes with on the parameter side. Connections also run to [[agentic-rl]] (where chains of latent reasoning steps might internalize what is currently externalized as tool-call traces) and to [[grokking]] as the training-dynamics phenomenon underlying implicit-reasoning emergence.

## References

- [[latent-thoughts-looped-transformers-saunshi-2025]] — coins "latent thoughts"; 1B-scale evidence.
- [[cot-or-loop-xu-2025]] — formal CoT-vs-loop separation.
- [[loop-think-generalize-2026]] — implicit-reasoning + grokking case study.
- [[looped-transformer]] — the architectural family.
- [[chain-of-thought-prompting]] — the externalized-token alternative.
