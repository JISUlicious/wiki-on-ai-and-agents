---
title: "To CoT or To Loop? A Formal Comparison Between Chain-of-Thought and Looped Transformers"
type: source
created: 2026-05-20
updated: 2026-05-20
sources:
  - cot-or-loop-xu-2025.md
arxiv_id: "2505.19245"
authors:
  - Kevin Xu
  - Issei Sato
first_author: Kevin Xu
year: 2025
introduces:
  - "[[looped-transformer]]"
tags:
  - 2025
status: complete
importance: high
---

# To CoT or To Loop? A Formal Comparison Between Chain-of-Thought and Looped Transformers

[arXiv:2505.19245](https://arxiv.org/abs/2505.19245) — Kevin Xu, Issei Sato (University of Tokyo), 2025.

## Summary

This paper provides a formal complexity-theoretic comparison between two ways of scaling inference-time compute in transformers: **token-recursion** via Chain-of-Thought (where the model extends the input by emitting intermediate reasoning tokens) and **depth-recursion** via the [[looped-transformer]] (where a fixed-size transformer block is applied repeatedly in the embedding space without producing tokens). Both mechanisms increase the effective number of computational steps, but they do so along orthogonal axes; the paper asks where each one dominates.

The central result is a two-sided separation that resolves what looks at first like a clean ordering. For **deterministic** tasks, the authors formalize computation as evaluation over directed acyclic graphs (DAGs) and show that CoT requires steps proportional to the **size** of the DAG (sequential, topological traversal), while a looped transformer requires loops proportional only to the **depth** of the DAG (layer-wise parallel evaluation). This yields a strict complexity-class separation in the polylogarithmic regime: under standard circuit assumptions, `CoT[log^k n, poly(n), log n] ⊊ LOOP[log^k n, poly(n), log n]`. But for **probabilistic / approximate-inference** tasks — formalized via self-reducibility (e.g., approximately uniform sampling from SAT solutions, FPAUS) — the direction flips: probabilistic CoT with stochastic decoding can act as an FPRAS, whereas deterministic looped transformers cannot, assuming FPRAS ≠ FPTAS. The paper also shows that even a "weak" CoT (only token-wise weak learning) becomes an approximate sampler when paired with [[self-consistency]] via coordinate-wise median aggregation. The takeaway: depth-recursion wins on structurally parallel deterministic computation; token-recursion with stochastic decoding wins on compositional probabilistic inference.

## Key Points

- **Task taxonomy.** Two regimes are formalized: (i) the *deterministic problem* — compute a fixed function `f: X → Y` exactly; (ii) the *probabilistic problem* — sample approximately from a target distribution over a multi-valued solution set, with an error parameter `ε`. Self-reducible relations (Schnorr 1976) tie the two together via the equivalence of approximate counting and approximate sampling.

- **Computation graphs as the deterministic substrate.** A DAG of local functions abstracts straight-line programs, Boolean circuits, dynamic-programming computations, and arithmetic. The paper's key contribution here is recasting CoT vs Looped TF as two different *simulation strategies* over the same DAG.

- **CoT simulates DAGs in size; Looped TF simulates them in depth.** Theorem 4.4: a log-precision CoT computes a DAG in steps proportional to `|V_n|` (topological-order decoding, one node per step). Theorem 4.7: a log-precision Looped TF computes the same DAG in loops proportional to its depth (each loop simulates a circuit layer in parallel), under additional assumptions of bounded fan-in, linear parallel space `O(n)`, and ≤`n` output nodes.

- **Polylog-regime separation.** Looped TF corresponds to threshold circuits whose depth scales with the number of loops (Theorem 4.11); CoT corresponds to circuits of `TC^0` gates whose size scales with the number of steps (Li et al. 2024). Therefore, assuming `TC^{k-1} ⊊ TC^k`, `CoT[log^k n, ...] ⊊ LOOP[log^k n, ...]`. Concrete instances in `LOOP[log^2 n, ...] \ CoT[log^2 n, ...]`: linear-equation solving (LIN), bipartite perfect matching (MATCH), fixed context-free language membership (FCFL).

- **Reverse separation for approximate inference.** Theorem 5.6: assuming FPRAS ≠ FPTAS, there exists a self-reducible relation for which a probabilistic CoT can approximate `|R(x)|` within any `ε` in polynomial time, but no deterministic looped transformer can match this within polynomial loop iterations. CoT-with-sampling can simulate polynomial-time probabilistic Turing machines (Nowak et al. 2024); looped TF cannot, by construction.

- **Weak CoT + self-consistency suffices.** Even a CoT that only satisfies `(α, γ)`-weak token-wise learning (low cross-entropy with probability > 1/2 + γ per token, given a sampled reasoning trajectory) can be promoted to an approximate sampler. Lemma 5.11: coordinate-wise median over `T = O(γ^{-2} log(|V|/δ))` independent reasoning paths drives the prediction within `O(√α)` of the true distribution. Theorem 5.13: combining this with self-reducibility yields an FPAUS for `p(· | x)` in time poly(`|x|`, log(1/ε)). This connects the theoretical PTM result of Nowak et al. to what practical LLMs actually do — train to bounded error, then exploit at inference via repeated sampling.

- **Depth-recursion vs token-recursion as complementary mechanisms.** The unifying frame: CoT externalizes intermediate state as tokens (scratchpad memory, supports arbitrary serial computation but no parallelism); Looped TF keeps state in the embedding (exploits structural parallelism but is constrained by fixed space and determinism). They are not substitutes — they are different inference-compute axes, and the right one depends on whether the task is parallel-deterministic or compositional-stochastic.

- **Practical guidance.** "Which to scale" depends on task structure: for parallelizable deterministic computation (graph problems, linear algebra, parsing), prefer looped depth-recursion; for tasks where the answer space requires exploring stochastic branches (planning, SAT-like search, theorem proving), prefer CoT with sampling and self-consistency.

## Related Concepts

- [[looped-transformer]] — depth-recursion architecture; this paper is a primary reference for its theoretical comparison with CoT.
- [[chain-of-thought-prompting]] — token-recursion baseline; analyzed in both deterministic and probabilistic regimes.
- [[inference-time-compute]] — both CoT and looped TF are inference-compute scaling axes; the paper is essentially about *which axis to scale*.
- [[latent-reasoning]] — looped transformers reason in embedding space rather than emitting tokens; this paper formalizes when that mode wins.
- [[expressive-power-looped-transformers]] — complexity-class characterization (`LOOP[T, d, p]` ↔ circuits of depth `T`) extends prior expressivity results.
- [[self-consistency]] — leveraged to amplify weak token-wise learners into approximate samplers.

## Notable Quotes

> "Looped Transformers can efficiently simulate parallel computations for deterministic tasks, which we formalize as evaluation over directed acyclic graphs. In contrast, CoT with stochastic decoding excels at approximate inference for compositional structures, namely self-reducible problems." — Abstract

> "CoT employs intermediate steps as scratchpad memory, supporting general computation without structural constraints. In contrast, Looped TF exploits structural parallelism to achieve greater efficiency, but requires additional assumptions." — §4.2

> "Under the same polylogarithmic number of steps or loops, Looped TF can simulate parallel computations more efficiently than CoT. This highlights the inherent parallelism of looped models, in contrast to the sequential nature of CoT." — §4.3

> "Assuming that FPTAS ≠ FPRAS, there exists a relation R such that a probabilistic CoT can approximate the count |R(x)| within any desired relative error ε > 0 with high probability in polynomial time, whereas no (deterministic) Looped TF can achieve the same approximation within a polynomial number of loop iterations." — Theorem 5.6

> "These findings offer guidance on whether to CoT or to Loop when scaling inference compute." — Conclusion

## References

- Vaswani et al. 2017 — Transformer ([[attention-is-all-you-need-vaswani-2017]]).
- Wei et al. 2022 — Chain-of-Thought prompting.
- Dehghani et al. 2019 — Universal Transformers (original looped formulation).
- Giannou et al. 2023 — Looped Transformers as Programmable Computers (Turing completeness).
- Saunshi et al. 2025 — Reasoning with Latent Thoughts: the power of looped transformers.
- Merrill & Sabharwal 2024, 2025 — Expressive power of transformers / log-depth transformers.
- Feng et al. 2023 — CoT theoretical perspective; DAG/DP simulations.
- Li et al. 2024 — CoT empowers transformers to solve inherently serial problems (`SIZE^{TC^0}` characterization).
- Nowak et al. 2024 — CoT as probabilistic Turing machine.
- Prystawski et al. 2023 — Why think step by step? (locality of experience).
- Jerrum, Valiant & Vazirani 1986 — Random generation from a uniform distribution; FPAUS / FPRAS framework.
- Schnorr 1976 — Self-reducibility.
- Wang et al. 2023 — Self-consistency improves CoT reasoning.
- Xu & Sato 2025 — Expressive power of looped transformers with timestep encoding.
- Welleck et al. 2024 — From decoding to meta-generation (inference-time algorithms survey).
