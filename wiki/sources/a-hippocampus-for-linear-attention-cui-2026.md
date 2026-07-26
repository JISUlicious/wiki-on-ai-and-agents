---
title: "A Hippocampus for Linear Attention: An Exact Memory for What the Recurrent State Forgets"
type: source
created: 2026-07-22
updated: 2026-07-22
sources:
  - a-hippocampus-for-linear-attention-cui-2026.md
arxiv_id: "2607.02303"
year: 2026
authors:
  - Wanyun Cui
tags:
  - 2026
  - paper
status: complete
importance: medium
---

# A Hippocampus for Linear Attention: An Exact Memory for What the Recurrent State Forgets

**Wanyun Cui** (Shanghai University of Finance and Economics — sole author) — [arXiv:2607.02303](https://arxiv.org/abs/2607.02303), cs.AI, 2 Jul 2026.

## Summary

Linear-attention and state-space models compress the whole prefix into a fixed-size recurrent state, buying O(1) memory at the cost of a *lossy* exact memory: a bounded state holds only so many distinct key→value associations before new writes overwrite old ones along shared key directions. The paper frames this through **Complementary Learning Systems (CLS)** theory — linear attention is a "neocortex" (slow, compressive, generalizing) that lacks a hippocampus (fast, exact, one-shot). **HOLA (Hippocampal Linear Attention)** keeps the standard delta-rule state as compressive memory and bolts on a small, bounded, *exact* KV cache, giving a **semiparametric test-time memory**: pure GDN is the parametric estimator `q^T S_t`, full softmax attention is an unbounded non-parametric kernel estimator, and HOLA is the bounded semiparametric case in between.

Two design choices follow. **What to store:** not the most recent tokens (the standard sliding-window hybrid), but the most *surprising* ones — and the delta rule already computes the surprise signal for free. HOLA evicts by `β·||e||`, the magnitude of the prediction residual actually committed to the state, so it keeps tokens that changed the state most rather than tokens that are merely recent. No learned eviction module is needed. **How to read:** naively reusing DeltaNet/GDN's L2-normalized queries and keys makes cache logits too small, so the cache softmax goes nearly uniform and the exact memory degenerates into another lossy average (with w=64 entries a perfectly matching key gets only ~3.5% of the mass). A Qwen3-style **decoupled RMSNorm-γ on the cache path only** restores sharp, near-argmax retrieval while preserving the unit-norm keys the delta rule needs for a stable state update.

At 340M parameters trained on 15B SlimPajama tokens (2k context), HOLA cuts Wikitext perplexity from the same-backbone GDN anchor's 27.32 to **22.92 (−16.1%)** — below a full-attention Transformer++ at 26.88 — and holds up on RULER needle recall out to 32k, i.e. **16× its 2k training length**, where GDN collapses. The cache costs almost nothing: 12,480 trainable scalars (<0.004% of the model) and ~5% peak memory over GDN (0.75 GB vs. 0.72 GB, flat at both 32k and 128k).

## Key points

- **Setup:** GDN architecture at 340M (d_model=1024, 24 layers, H=4, d=256), Preconditioned-DeltaNet recipe, SlimPajama 15.0B tokens, Mistral tokenizer, 2048 training context. Cache: window **w=64**, chunk C=256, eviction score `β·||e||`.
- **Main results (340M, ctx-2048)** — sub-quadratic models, Wikitext / LAMBADA perplexity, and in-context retrieval (FDA / SWDE):

| Model | Wiki ppl ↓ | LMB ppl ↓ | FDA ↑ | SWDE ↑ |
|---|---|---|---|---|
| Transformer++ (full attention, quadratic) | 26.88 | 42.15 | 46.1 | 25.9 |
| DeltaNet | 29.04 | 45.76 | 8.5 | 27.1 |
| GLA | 28.78 | 39.00 | 11.3 | 16.8 |
| GSA | 28.17 | 42.57 | 6.4 | 16.9 |
| KDA (Kimi Delta Attention) | 26.18 | 31.37 | 13.9 | 34.1 |
| GDN (paper's own anchor) | 27.32 | 30.95 | 11.7 | 29.0 |
| HOLA+recency (matched control) | 25.04 | 32.33 | 16.9 | 29.9 |
| **HOLA** | **22.92** | **30.26** | **20.1** | **35.9** |

- **Perplexity gain is consistent across scale** — HOLA lowers Wikitext ppl by 15–16% vs. a matched-backbone GDN at every size: 46M 71.0 → 59.5; 170M 35.98 → 30.51; 340M 27.32 → 22.92.
- **Length generalization (RULER S-NIAH-1, 340M, evaluated 2k→32k):** at **32k**, HOLA 0.58 vs. HOLA+recency 0.24 vs. GDN 0.14 — a +0.44 margin over the bare state, and **+0.64 at 16k**. Also S-NIAH-2 at 8k: 0.35 vs. 0.09; multi-value at 2k: 0.28 vs. 0.17.
- **Recency is the wrong eviction signal.** The matched control (identical architecture, only position-vs-surprise eviction changed) barely improves on having no cache at all on the far needle: 0.24 at 32k vs. GDN's 0.14, far below importance eviction's 0.58. Once a distant-but-needed token slides out of a window "it is as lost as if the state had overwritten it."
- **Full attention is not a length baseline.** The Transformer++ ceiling is the strongest model within its 2k training length (exact softmax retrieval) but this RoPE checkpoint **drops to 0 on every shown RULER task at 4k and beyond** — precisely the regime where HOLA still works.
- **Commonsense is a wash.** Differences among GDN / HOLA+recency / HOLA on the six-task commonsense average are within single-seed noise (<0.7 points); HOLA's six-task average is 42.85 vs. GDN's own anchor. The cache's gains are concentrated in perplexity and retrieval, as designed.
- **Cost:** the only cache-specific learned parameters are cache-path Q/K RMSNorm scales plus a per-head sink and cache gate — L(2d+2H) = 24(512+8) = 12,480 trainable scalars. The cache is inference state, ~31 MB in bf16 at (w+C) KV pairs per layer.
- **Closest prior work** is LTE (He & Garner, 2025), which also augments GDN with a bounded evictable KV cache but *learns* the eviction rule; HOLA's contribution is that the delta rule's committed residual is already a usable surprise signal, so no eviction module is learned.

## Concepts & entities

- [[state-space-model]] — the "neocortex" side: DeltaNet / Gated DeltaNet / Mamba-family fixed-size recurrent states are exactly what HOLA is complementing. Linear attention itself is treated here as part of this family.
- [[kv-cache]] — HOLA's hippocampal component is a bounded, selectively evicted KV cache (w=64 per layer), read with a decoupled RMSNorm-γ path so it retrieves rather than averages.
- [[attention-mechanism]] — full softmax attention is the O(T) memory / O(T²) compute gold standard for exact recall that HOLA tries to partially recover at linear cost.
- [[long-context-llm]] — evaluated on RULER out to 32k, 16× the 2k training length; the headline claim is length-robustness, not just in-distribution quality.
- [[episodic-memory-llm]] — CLS framing: the cache is an explicitly episodic, one-shot, surprise-gated store, contrasted with the state's slow semantic compression.
- [[transformer-architecture]] — Transformer++ serves as the full-attention comparison point (26.88 Wikitext ppl at 340M).

## References

- [arXiv:2607.02303](https://arxiv.org/abs/2607.02303)
