---
title: "Reasoning with Latent Thoughts: On the Power of Looped Transformers"
type: source
created: 2026-05-20
updated: 2026-05-20
sources:
  - latent-thoughts-looped-transformers-saunshi-2025.md
arxiv_id: "2502.17416"
venue: ICLR 2025
authors:
  - "[[nikunj-saunshi]]"
  - Nishanth Dikkala
  - Zhiyuan Li
  - Sanjiv Kumar
  - Sashank Reddi
first_author: Nikunj Saunshi
year: 2025
introduces:
  - "[[looped-transformer]]"
  - "[[latent-reasoning]]"
tags:
  - 2025
status: complete
importance: high
---

# Reasoning with Latent Thoughts: On the Power of Looped Transformers

**Source file**: [[latent-thoughts-looped-transformers-saunshi-2025]]
**arXiv**: [arXiv:2502.17416](https://arxiv.org/abs/2502.17416)
**Venue**: ICLR 2025
**Authors**: Nikunj Saunshi, Nishanth Dikkala, Zhiyuan Li, Sanjiv Kumar, Sashank J. Reddi (Google Research; Zhiyuan Li also at TTIC)
**Year**: 2025 (Feb)

## Summary

Saunshi et al. argue that **many reasoning problems require depth but not necessarily parameters**, and that weight-shared **looped transformers** are an unusually good fit for this regime. Their core empirical finding, denoted `(k ⊗ L)` for "a `k`-layer block looped `L` times", is that a `k`-layer transformer looped `L` times nearly matches a non-looped `kL`-layer model (the **iso-FLOP** baseline) and substantially beats the same-parameter `k`-layer model (the **iso-param** baseline). This holds first on synthetic reasoning suites — n-ary addition, p-hop induction, and synthetic grade-school math (i-GSM) — where even a 1-layer model looped 12 times solves problems that crush a 1-layer baseline. It then carries over to **causal language modeling at 1B-parameter scale on the Pile**, where the looped model has worse perplexity (capacity-limited) but matches or exceeds the 24-layer baseline on reasoning-heavy downstream tasks (open-book QA, math word problems, reasoning primitives) despite having `24/k` times fewer parameters.

The paper's flagship conceptual contribution is the **"latent thoughts"** interpretation: each loop iteration is reframed as the model emitting *multiple* unobserved thought vectors, in contrast to [[chain-of-thought-prompting]] which emits exactly one discrete thought token per autoregressive step. The authors prove that a looped transformer can in fact **simulate `T` steps of CoT with `T` loops** (with a scratchpad), giving a theoretical bridge between looping and CoT-style inference-time compute. Empirically they also observe a clean **`Acc = α log(D) + β` scaling law** in effective depth `D = kL`, with `α_loop / α_base > 1` on reasoning primitives — i.e., depth-via-looping is *more* effective per unit depth than depth-via-parameters for reasoning. The paper positions looping as a parameter-efficient analogue of o1 / [[deepseek-r1]]-style [[inference-time-compute]], and closes with a cosine-similarity regularizer that pushes a fully parameterized model toward a looped solution and inherits the reasoning inductive bias *without* the perplexity cost.

This is one of the cleanest 2025 papers arguing that **iterative latent computation is a first-class alternative to emitted-token reasoning** — a conceptual sibling to the broader [[latent-reasoning]] trend and a key reference for "thinking in continuous space".

## Key Points

- **Notation**: `(k ⊗ L)` denotes a `k`-layer transformer block looped `L` times. Compared against the **iso-param** `(k ⊗ 1)` baseline (same parameters, `L` times less depth) and the **iso-FLOP** `(kL ⊗ 1)` baseline (same depth, `L` times more parameters).
- **Claim 1 — synthetic reasoning needs depth, not parameters**: On n-ary addition (`n ∈ {2..32}`, 3-digit operands), p-hop induction (alphabet size 4, length 256, `p ∈ {16, 32}`), and i-GSM (mod-7 DAG math problems of depth 4), `(k ⊗ L)` looped models nearly match the `(kL ⊗ 1)` baseline and crush `(k ⊗ 1)`. Even `(1 ⊗ 12)` solves addition that `(1 ⊗ 1)` cannot.
- **Claim 2 — language modeling inductive bias**: At 1B params, 24 layers, 250B tokens of the Pile, looped models cover only ~34–50% of the perplexity gap between iso-param and iso-flop baselines but cover 60–130%+ of the gap on **open-book QA, math word problems, and reasoning primitives** (sometimes *exceeding* the 24-layer baseline with `24/k` times fewer parameters).
- **Reasoning vs. memorization dichotomy**: Closed-book QA (memorization) tracks perplexity tightly, so looped models do not help there. Open-book QA, math, and reasoning primitives improve disproportionately. Isoplots (perplexity vs. downstream as training proceeds) confirm a separation that is not explained by capacity.
- **Log-depth scaling law**: `Acc ≈ α log(D) + β` in effective depth `D = kL` for both looped and non-looped models, with `α_loop / α_base ≈ 1.19` on reasoning primitives — i.e., loops add reasoning capability faster than adding fresh layers does.
- **Latent thoughts**: Each loop iteration is interpreted as emitting multiple unobserved "thought vectors". CoT is reframed as a degenerate looped model that emits exactly one discrete thought token per step; looped transformers generalize this by allowing multiple high-dimensional latent thoughts per iteration.
- **CoT simulation theorem**: A looped transformer with a scratchpad can simulate `T` steps of chain-of-thought with `T` loops (theoretical, Section 5.4). Provides a depth-equivalence between latent looping and explicit token-level CoT.
- **Middle looping variant**: Keep the first and last layers independent and loop only the middle block (inspired by gradual stacking / MidAS). Better perplexity than naive looping with similar reasoning gains; connection to [[saunshi-2024-stacking]]-style gradual stacking is explicit.
- **Looping-inspired regularizer**: Add a cosine-similarity penalty pushing successive blocks of `k` layers toward each other (Eq. 3–4). At `λ_reg = 10` this drives cross-block cosine to ≥0.98 and inherits the reasoning inductive bias **without sacrificing perplexity** — closing the loop on the reasoning/memorization dichotomy.
- **Theoretical results (Section 5)**: Looped transformers can solve group composition (generalizes addition) and the p-hop problem with near-optimal depth and far fewer parameters than non-looped constructions. General simulation result: a non-looped model with few distinct layers can be emulated by a looped model with small size blowup.
- **Positioning vs. o1 / R1**: Authors explicitly frame looped models as a parameter-efficient analogue of the test-time-thinking paradigm popularized by o1 and [[deepseek-r1]] — generate more loops at inference to get more reasoning, much like generating more CoT tokens.

## Entities Mentioned

- [[google-research]] (org) — host affiliation for four of the five authors.
- [[nikunj-saunshi]] (person) — first author; this paper sits in a sequence of his work on stacking, looping, and the perplexity-vs-reasoning inductive bias.
- [[deepseek-r1]] (model) — cited as a marquee example of inference-time-scaling "thinking" models that looping aims to compete with parametrically.

## Concepts Discussed

- [[looped-transformer]] — the central architectural object; weight-shared multi-layer transformer with explicit loop count.
- [[latent-reasoning]] — the paper's "latent thoughts" framing is one of the foundational 2025 references for this concept.
- [[chain-of-thought-prompting]] — recast as a one-token-per-iteration special case of looped computation; looped models simulate `T` CoT steps with `T` loops.
- [[inference-time-compute]] — looping treated as a parameter-side analogue of test-time CoT scaling.
- [[transformer-architecture]] — the underlying backbone being looped.
- Iso-param / iso-FLOP analysis methodology — a clean compute-vs-capacity decomposition that is reusable in any architecture study.
- Perplexity-vs-downstream isoplots — diagnostic from Saunshi et al. (2022) used to argue that the reasoning gains are an inductive bias, not a capacity artifact.
- Cosine-similarity weight regularization — Eq. (3)–(4); a soft path from a fully parameterized model toward a looped one.

## Notable Quotes

> "Many reasoning problems require depth but not necessarily parameters. That is, they can be solved via looped models."

> "Looped models generate latent thoughts and can, in theory, simulate CoT reasoning."

> "CoT reasoning ... is essentially a looped model that generates a single thought token in each iteration. However, looped models can be more powerful since they can generate multiple 'latent thoughts' in each iteration."

> "For language modeling, looped models have an inductive bias towards good reasoning despite having worse perplexity and memorization to an iso-flop non-looped model."

> "More loops continue to help, and the relative benefit of loops is higher for reasoning tasks ... the impact of loops is even higher (1.19x) than impact of depth for reasoning primitives."

> "We believe that leveraging looping explicitly for inference-time scaling is a very promising future direction."

## References (selected)

- Lan et al. (2020) — ALBERT; parameter-sharing precedent for looped transformers.
- Dehghani et al. (2018) — Universal Transformer; adaptive-compute looping precedent.
- Bai et al. (2019) — Deep Equilibrium Models.
- Saunshi et al. (2024) — gradual stacking / MidAS; the immediate precursor for the middle-looping variant and the perplexity-vs-reasoning isoplot methodology.
- Saunshi et al. (2022); Liu et al. (2023) — perplexity-vs-downstream inductive bias studies.
- Ye et al. (2024) — argument that depth matters more than width for reasoning, and the i-GSM design that this paper builds on.
- Sanford et al. (2024b) — the p-hop benchmark used in Section 2.
- Wei et al. (2022b) — chain-of-thought prompting.
- Guo et al. (2025) — [[deepseek-r1]]; cited as the o1-class thinking model that motivates the inference-time-scaling framing.
