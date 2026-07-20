---
title: "LLM.int8(): 8-bit Matrix Multiplication for Transformers at Scale"
type: source
created: 2026-07-07
updated: 2026-07-07
sources: []
arxiv_id: "2208.07339"
venue: NeurIPS 2022
year: 2022
authors:
  - Tim Dettmers
  - Mike Lewis
  - Younes Belkada
  - Luke Zettlemoyer
tags:
  - 2022
  - paper
status: complete
importance: high
---

# LLM.int8(): 8-bit Matrix Multiplication for Transformers at Scale

**Dettmers, Lewis, Belkada, Zettlemoyer** — [arXiv:2208.07339](https://arxiv.org/abs/2208.07339), **NeurIPS 2022**.

## Why this paper matters most

Its lasting contribution is not the method but the **discovery of emergent outlier features** — the empirical foundation for essentially every later [[quantization]] technique.

A **phase shift occurs at ~6.7B parameters**, where extreme-magnitude features appear in *all* transformer layers and 75% of sequence dimensions. At 6.7B there are **~150,000 outliers per sequence concentrated in only 6 feature dimensions**.

These outliers are functionally essential:

| Intervention | Effect |
|---|---|
| Zero the 6 outlier dimensions (0.1% of features) | top-1 attention softmax mass drops **>20%**; perplexity degrades **600-1000%** |
| Zero the same number of *random* features | <=0.3% attention change; ~0.1% perplexity |

That asymmetry is why naive activation quantization fails at scale, and why the field split into keep/move/rotate/prevent strategies.

## Method

1. **Vector-wise quantization** — a separate normalization constant per row of X and per column of W, so each inner product gets its own scale.
2. **Mixed-precision decomposition** — any feature dimension containing a value above threshold **alpha = 6.0** is split off and computed in FP16; the remaining **>99.9%** runs in INT8.

**Calibration**: none — a 16/32-bit checkpoint is loaded, converted, and used immediately.

## Concepts

- [[quantization]] — the outlier problem and the "keep in high precision" family.
- [[massive-activations-attention-sinks-sun-2026]] — later characterization of the same phenomenon.
- [[smoothquant-xiao-2022]] · [[quarot-ashkboos-2024]] — the responses.

## References

- [arXiv:2208.07339](https://arxiv.org/abs/2208.07339)
