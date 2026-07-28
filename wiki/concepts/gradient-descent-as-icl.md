---
title: Gradient Descent as In-Context Learning
type: concept
created: 2026-07-28
updated: 2026-07-28
sources:
  - looped-transformers-learning-algorithms-yang-2023.md
  - looped-transformers-multi-step-gd-gatmiry-2024.md
  - looped-transformers-programmable-computers-giannou-2023.md
status: draft
importance: medium
---

# Gradient Descent as In-Context Learning

The hypothesis that **transformers perform [[in-context-learning]] by implicitly running an optimization algorithm in their forward pass** — that a forward pass over a prompt of examples approximates gradient descent on those examples, with each layer performing roughly one step.

Advanced independently around 2022 by **Akyürek et al.** and **von Oswald et al.**, it is the mechanistic story that reframes in-context learning from "pattern matching" into "the model is *learning*, in the ordinary optimization sense, at inference time."

## Why it matters here

This hypothesis is the **direct motivation for [[looped-transformer|looped transformers]]**. If one block ≈ one gradient step, then iterating a single block is not a parameter-saving hack — it is *running more optimization steps*, and depth-recursion becomes principled rather than merely economical. [[looped-transformers-learning-algorithms-yang-2023|Yang et al.]] make exactly this argument: each loop step corresponds to about one gradient step.

The theoretical thread then hardens it: [[looped-transformers-multi-step-gd-gatmiry-2024|Gatmiry et al.]] **turn the hypothesis into a theorem** for the looped, linear-attention case — establishing multi-step gradient descent rather than assuming it — while [[looped-transformers-programmable-computers-giannou-2023|Giannou et al.]] generalize the SGD-by-construction thread into transformers as programmable computers.

## Caveats

The strongest results are proved under **restricted settings** — typically linear attention and linear-regression tasks — so "transformers do gradient descent" is a claim about a tractable regime, not a demonstrated account of what frontier LLMs do on natural language. Treat it as a well-supported mechanistic *hypothesis* with theorems in special cases.

> [!note] Draft
> Created during a lint pass — referenced from three looped-transformer sources with no page behind it. The primary sources (Akyürek et al. 2022; von Oswald et al. 2022) are **not yet ingested**; this page currently rests on the looped-transformer papers that cite them. Ingesting the originals would put it on a proper footing.

## Related

- [[looped-transformer]] — the architecture this hypothesis motivates.
- [[in-context-learning]] — the phenomenon being explained.
- [[attention-mechanism]] · [[transformer-architecture]] · [[linear-attention]] (the regime most proofs assume).

## References

- [[looped-transformers-learning-algorithms-yang-2023]] · [[looped-transformers-multi-step-gd-gatmiry-2024]] · [[looped-transformers-programmable-computers-giannou-2023]]
