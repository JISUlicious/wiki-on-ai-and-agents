---
title: Looped Transformer
type: concept
created: 2026-05-20
updated: 2026-05-20
sources:
  - universal-transformer-dehghani-2018.md
  - looped-transformers-programmable-computers-giannou-2023.md
  - looped-transformers-learning-algorithms-yang-2023.md
  - looped-transformers-multi-step-gd-gatmiry-2024.md
  - looped-transformers-length-generalization-fan-2024.md
  - expressive-power-looped-transformers-xu-2024.md
  - latent-thoughts-looped-transformers-saunshi-2025.md
  - cot-or-loop-xu-2025.md
  - loop-think-generalize-2026.md
  - parcae-scaling-laws-looped-2026.md
status: complete
importance: high
tags:
  - 2023
---

A **looped transformer** is a [[transformer-architecture|transformer]] whose forward pass is re-applied (looped) over the same set of layers — depth-recursion as a substitute for, or complement to, depth-via-stacking. Instead of scaling capability by adding more distinct layers, a `k`-layer block is unrolled `L` times with tied weights, producing an effective depth of `kL` at roughly `1/L` the parameter count. The construction trades parameters for a recurrent inductive bias that is unusually well-matched to iterative algorithms (gradient descent, multi-hop graph traversal, dynamic programming) and to [[latent-reasoning|silent reasoning]] in embedding space. The line runs from the 2018 [[universal-transformer]] through the constructive 2023 work of Giannou et al. to 2026's [[recurrent-depth-transformer|pretraining-scale RDTs]] and [[parcae|Parcae]] scaling laws.

## Core mechanism

- Loop = repeated application of a fixed transformer block to its own output, with weights tied across iterations (`h_{t+1} = f_θ(h_t)` or, with input injection, `h_{t+1} = f_θ(h_t + e)`).
- A `k`-layer block looped `L` times approximates a `kL`-layer non-looped network on reasoning tasks at ~`1/L` the parameter count — Yang 2023 shows a 1-layer × 20-loop model matches a 12-layer baseline on in-context linear regression with <10% of the parameters.
- Hand-constructed weights can make a constant-depth looped transformer simulate a programmable computer (Giannou 2023): 13 layers + a loop suffice to emulate SUBLEQ and run matrix algebra, power iteration, and SGD over an implicit network.
- Learned looping captures iterative in-context-learning algorithms — Gatmiry 2024 proves looped transformers implement multi-step preconditioned gradient descent, and probing in Yang 2023 shows monotonic refinement of `w_OLS` across loop iterations.
- Adding a per-iteration **timestep encoding** closes the expressivity gap with non-looped models (Xu & Sato 2024), so looped transformers are not constrained to be position-permutation-invariant across depth.

## What it advances

Looped transformers are the architectural answer to a recurring question: must depth-of-computation always be paid for in distinct parameters? They demonstrate that for problems with iterative structure — algorithms, multi-hop reasoning, optimization — depth and parameters are partially separable axes, and that weight-shared depth is often the more parameter-efficient one. They also reposition [[inference-time-compute]] from a tokens-only knob (CoT) to a depth-knob: more loops at inference yields more reasoning compute without any extra tokens emitted (Saunshi 2025). The 2026 wave (Parcae, RDTs, [[openmythos]]) shows the recipe is stable and predictable at pretraining scale.

## Relation to other concepts

The 2018 [[universal-transformer]] is the direct ancestor — same idea, [[adaptive-computation-time|ACT-based halting]], framed as "transformer-as-RNN-over-depth". [[recurrent-depth-transformer]] is the modern (2025–2026) rebranding for pretraining-scale LMs. [[latent-reasoning]] is the cognitive framing — each loop iteration is a "latent thought". [[chain-of-thought-prompting]] is the orthogonal token-recursion alternative; Xu & Sato 2025 give the formal comparison (depth-recursion wins on parallel-deterministic tasks, token-recursion wins on stochastic-compositional ones). [[parameter-sharing]] is the underlying mechanism. Connections also run to [[in-context-learning]] (looped models learn iterative ICL algorithms), to [[deepseek-r1]] as the RL-reasoning sibling thread, and to [[gradient-descent-as-icl]] as the mechanistic hypothesis the architecture validates.

## References

- [[universal-transformer-dehghani-2018]] — the ancestor.
- [[looped-transformers-programmable-computers-giannou-2023]] — the anchor; modern object named.
- [[looped-transformers-learning-algorithms-yang-2023]] — first strong empirical evidence loops are learnable.
- [[looped-transformers-multi-step-gd-gatmiry-2024]] — proof that loops implement multi-step preconditioned GD.
- [[expressive-power-looped-transformers-xu-2024]] — timestep-encoding expressivity result.
- [[latent-thoughts-looped-transformers-saunshi-2025]] — latent-thoughts framing + 1B-scale evidence.
- [[cot-or-loop-xu-2025]] — formal CoT-vs-loop separation.
- [[loop-think-generalize-2026]] — implicit-reasoning RDT case study.
- [[parcae-scaling-laws-looped-2026]] — the stable pretraining recipe + scaling laws.
- [[looped-transformers-length-generalization-fan-2024]] — looping for length generalization.
