---
title: "On Expressive Power of Looped Transformers"
type: source
created: 2026-05-20
updated: 2026-05-20
sources:
  - expressive-power-looped-transformers-xu-2024.md
arxiv_id: "2410.01405"
authors:
  - Kevin Xu
  - Issei Sato
first_author: Kevin Xu
year: 2024
introduces:
  - "[[looped-transformer]]"
  - "[[timestep-encoding]]"
tags:
  - 2024
status: complete
importance: high
---

# On Expressive Power of Looped Transformers: Theoretical Analysis and Enhancement via Timestep Encoding

[arXiv:2410.01405](https://arxiv.org/abs/2410.01405) — Kevin Xu, Issei Sato (The University of Tokyo). ICML 2025.

## Summary

This paper establishes the first **approximation rate** for [[looped-transformer]]s — Transformers whose fixed-size block is recursively applied `r` times via weight-tying. Building on the modulus-of-continuity framework used for looped ReLU networks (Zhang et al., 2023), the authors prove that a single-block Looped Transformer with two heads, head size 1, and width `O(d)` can approximate any continuous permutation-equivariant sequence-to-sequence function on `[0,1]^{d×N}`, with error that shrinks as the loop count `r` grows (Theorem 3.6). Crucially, the bound involves not one but **three** moduli of continuity: a standard *sequence* modulus `ω_f`, plus *contextual* `ω_f^{cont}` and *token* `ω_f^{tok}` moduli that are specific to the looped architecture. The construction proceeds in three internal stages — token-wise quantization, contextual-mapping via inner-product sequence IDs, and function-value mapping — all packed into a single Transformer block. A universal-approximation corollary follows under infinite-precision weights.

The contextual/token-modulus dependence reveals an **intrinsic structural limitation**: weight-tied feed-forward layers cannot exactly memorize a sequence of target embeddings `{y_k}`, leaving a residual error bounded only by the maximum gap between adjacent targets (Lemma 4.1). The authors fix this by making the FFN scaling depend on the loop index — `FF(X) → η(t) ⊙ FF(X)` — and prove that this **Timestep-Modulated Looped Transformer (TMLT)** memorizes labels *exactly* (Theorem 4.2). To avoid blowing up parameter count, `η(t)` is generated from a shared MLP over a sinusoidal [[timestep-encoding]] (the same form used in Transformer positional embeddings and DiT-style adaptive normalization), and applied to RMSNorm gains and attention/FFN output scales. Experiments on Sudoku, Countdown, edit-distance, longest common subsequence, in-context decision trees, and WikiText-103 confirm: more loops helps, and timestep encoding gives additional consistent gains — empirically validating the theoretical limitation and matching the earlier empirical finding by [[yang-2023]]-style work that conditioning loops on `t` matters.

## Key Points

- **Approximation rate (Theorem 3.6)**: For `f ∈ F_PE([0,1]^{d×N})` and `r > N`, a single-block Looped Transformer with two heads, head size 1, width `q = 49d+25`, embedding dim `17d+9`, achieves
  `‖L₂ ∘ TF^r ∘ L₁ − f‖_{L_p} ≤ (Nd)^{1/p}[ω_f^{tok}(δ√d) + ω_f^{cont}(δ√{Nd}) + ω_f(δ√{Nd})] + O(N^{2/p} δ^{d/p}) + O((Mδ)^{-1}dN)^{1/p}`
  where `δ = ((r−N)/2)^{-1/((N+1)d+1)}`. Parameter count is `O(d)`, independent of accuracy and sequence length; bit complexity is `O(δ^{-(N+1)d})`.
- **Three moduli of continuity**: the standard *sequence* modulus `ω_f(δ)` is insufficient — Transformers compute *contextual* mappings, so the proof requires also bounding (a) `ω_f^{cont}(δ)`: variation in a token's output under context perturbations with the token itself held fixed; and (b) `ω_f^{tok}(δ)`: variation under token perturbations with context held fixed. These two are **unique to Looped Transformers** and absent from prior bounds for either standard Transformers or looped ReLU networks.
- **Proof sketch in three loop-stages**: Step 1 uses `δ^{-1}−1` loops to assign discrete *token IDs*; Step 2 uses `N` loops to compute a *sequence ID* via inner product with `u = (δ^{-d(N-1)}, …, δ^{-d}, 1)`, yielding `|u^⊤ z − u^⊤ z'| > 1` for distinct sorted ID-tuples; Step 3 uses `2δ^{-(N+1)d}−1` loops to map each *contextual token ID* `k ∈ K` to its target embedding `y_k`. All three sub-networks fit inside a single Transformer block — tighter than the Zhang et al. (2023) looped-ReLU construction.
- **Universality (Corollary 3.7)**: The hypothesis class of single-block Looped Transformers with `m, q ≤ Cd`, `h = 2`, `s = 1`, varying `r ∈ ℕ`, is dense in continuous permutation-equivariant maps `[0,1]^{d×N} → [0,1]^{d×N}` in the `L_p` norm. Even weight-tied self-attention with hard-max can compute contextual mappings.
- **Structural limitation (Lemma 4.1)**: A weight-tied `(id+FF)^{K-1}` block cannot exactly fit an arbitrary label sequence `{y_k}`; the error per coordinate is bounded only by `max_k |(y_k − y_{k-1})_i|`. This is the source of the extra `ω_f^{tok}` and `ω_f^{cont}` terms in the rate.
- **Fix via timestep encoding (Theorem 4.2)**: Replacing `FF(X)` with `η(t) ⊙ FF(X)`, where `η: ℕ → ℝ^{4d}` is a per-loop scaling vector, restores **exact** memorization: `|L₂ ∘ (id+η⊙FF)^{K-1} ∘ L₁(k) − y_k|_i = 0`. The needed embedding dim drops from `12d` to `4d` and FFN width from `18d` to `6d`.
- **TMLT architecture**: Sinusoidal `PE(t)` → 2-layer MLP with SiLU → `TE(t) ∈ ℝ^d` → linear `W₅·SiLU(TE(t)) + b₅` produces time-conditional scales `α₁(t), α₂(t), γ₁(t), γ₂(t)`, which gate attention output, FFN output, and the two RMSNorm gains. Design is **inspired by adaptive instance normalization** in DiT (Peebles & Xie, 2023) and analogous to hypernetworks (Ha et al., 2016).
- **Empirical validation**: On Sudoku, Countdown, LCS, edit-distance — increasing `r` substantially closes the gap to (or beats) a 12-layer standard Transformer; adding timestep encoding gives a further bump that is especially large at high `r` (e.g. ED-60 at `r=100`: 47.7 → 88.3; LCS-100 at `r=100`: 88.3-equivalent improvements). Gains also appear on in-context decision-tree learning (MSE 1.4e-2 → 1.7e-3 at `r=12`) and WikiText-103 perplexity (20.6 → 19.6 at `r=24`).
- **Matches Yang et al. (2024)**: The empirical use of timestep conditioning in prior looped-Transformer ICL work is now theoretically justified — without timestep modulation the architecture is provably worse at memorization than its non-tied counterpart.

## Entities

- [[kevin-xu]] (first author, University of Tokyo)
- [[issei-sato]] (advisor, University of Tokyo)
- [[university-of-tokyo]]
- Venue: ICML 2025 (PMLR 267)

## Concepts

- [[looped-transformer]] — main object of study; this paper provides the canonical expressivity-bound result.
- [[timestep-encoding]] — the proposed fix; sinusoidal-PE-derived per-loop scaling that elevates a vanilla looped Transformer to TMLT.
- [[universal-approximation]] — Corollary 3.7 extends universality results (Yun et al., 2020; Kajitsuka & Sato, 2024) to the weight-tied looped setting.
- [[transformer-architecture]] — the underlying block being recursed.
- [[weight-tying]] — the structural constraint that makes Looped Transformers parameter-efficient but introduces the contextual/token modulus dependence.
- [[modulus-of-continuity]] — the quantitative regularity measure on which the rate hinges; three variants are newly defined here.
- [[chain-of-thought]] / [[reasoning]] — Looped Transformers are positioned as an alternative inductive bias for reasoning vs. CoT prompting.
- [[adaptive-instance-normalization]] / DiT — inspiration for the time-conditional RMSNorm gains.

## Notable Quotes

> "Looped Transformers provide advantages in parameter efficiency, computational capabilities, and generalization for reasoning tasks. However, their expressive power regarding function approximation remains underexplored. In this paper, we establish the approximation rate of Looped Transformers by defining the modulus of continuity for sequence-to-sequence functions."

> "We found that this [sequence modulus] alone is insufficient to determine the approximation rate of Looped Transformers, in contrast to the case of ReLU networks. Informally, this issue arises because Transformers compute contextual mappings."

> "The approximation rate depends on three types of continuity, with contextual and token dependencies unique to Looped Transformers; these dependencies are not present in standard Transformers or looped ReLU networks."

> "We employ timestep encodings to condition scaling parameters on the loop index (timestep). This method is inspired by adaptive instance normalization."

> "To the best of our knowledge, this study is the first to investigate the function approximation capabilities of Looped Transformers."

## References (selected)

- Zhang et al. (2023) — approximation rate of looped ReLU networks; main methodological precursor.
- Yun et al. (2020) — universality of standard Transformers for permutation-equivariant seq-to-seq functions.
- Kim et al. (2023) — contextual-mapping construction with `N` parameters.
- Kajitsuka & Sato (2024) — single-layer Transformer universality.
- Dehghani et al. (2019) — Universal Transformer (original recursive/adaptive-depth design).
- Giannou et al. (2023) — Looped Transformers as programmable computers.
- Yang et al. (2024) — Looped Transformers for in-context learning (uses timestep encoding empirically).
- Saunshi et al. (2025) — inductive bias of Looped Transformers for reasoning.
- Peebles & Xie (2023) — DiT, source of the adaLN/adaptive-instance-norm conditioning idea.
- Vaswani et al. (2017) — original Transformer.
