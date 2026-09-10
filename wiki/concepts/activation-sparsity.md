---
title: Activation Sparsity
type: concept
created: 2026-09-10
updated: 2026-09-10
sources:
  - llm-in-a-flash-alizadeh-2023.md
  - neuroprefetcher-delta-prefetching-dhar-2026.md
status: draft
importance: medium
---

# Activation Sparsity

The fraction of a feed-forward layer's intermediate neurons that contribute nothing to a given token — and therefore whose weights need not be read. It is the resource that [[weight-streaming]] spends, and its *source* determines whether a speedup is free or bought with accuracy.

## Three kinds, not one

| Source | Exact? | Needs a predictor? | Where measured |
|---|---|---|---|
| **ReLU zeros** — the activation is literally 0 | yes | yes, to know *before* the read (~1.25% of model size) | [[llm-in-a-flash-alizadeh-2023]]: OPT-6.7B is ~97% sparse; 2.4% of FFN neurons loaded per token with windowing |
| **Magnitude thresholding** on SiLU/GELU — keep the top-k by absolute value | **no** — lossy by construction | yes (2.86% of model size) | [[neuroprefetcher-delta-prefetching-dhar-2026]]: 62% sparsity gives 9.5× over demand paging at ~88–90% of dense HellaSwag |
| **MoE routing** — the router selects *k* of *N* experts | yes | **no** — the router *is* the read list | [[mixture-of-experts]]; every Apple Silicon expert streamer on [[weight-streaming]] |
| **Learned block-level top-K at prefill** — top-50% neurons by gated-activation norm over a 128-token block, predicted by a small MLP | no — lossy, with an error-compensation network | yes (~3.6% of model size, plus compensator) | [[fast-forward-predictive-ffn-sparsity-prefill-gautam-2026]]: 1.44× TTFT at 2–4K, <6% LongBench loss. Saves **FLOPs at prefill**, where the rows above save **bytes at decode** |

## The thing that broke between 2023 and 2026

The 2023 technique assumed ReLU. Modern open models use SiLU or GELU, where nothing is exactly zero, and [[llm-in-a-flash-alizadeh-2023|Alizadeh et al.]] found that *relufying* such a model costs **1–3 MMLU points** (Phi-2 57 → 54.3; Llama 2 41.8 → 38.96), and that swapping Swish for ReLU inside a SwiGLU FFN does not by itself yield sparsity — a thresholded variant is required. That is why the field split: dense models went the lossy-thresholding route, and the practical implementations moved to MoE, where sparsity is exact and known in advance.

## Related

- [[weight-streaming]] · [[mixture-of-experts]] · [[token-pruning]] · [[sparse-attention]] — the other sparsities this wiki tracks, which act on tokens and attention rather than neurons.
- [[arithmetic-intensity]] — why fewer bytes per token is the whole point.

## References

- [[llm-in-a-flash-alizadeh-2023]] · [[neuroprefetcher-delta-prefetching-dhar-2026]]
