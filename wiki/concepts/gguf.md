---
title: GGUF
type: concept
created: 2026-07-07
updated: 2026-07-07
sources:
  - which-quantization-kurt-2026.md
builds-on:
  - [[quantization]]
status: draft
importance: medium
---

# GGUF

**GGUF** (GGML Universal File) is the model container format used by [[llama-cpp]] and downstream tools (ollama, LM Studio). It is the dominant format for **locally-run community models**, and its quantization schemes are `ggml`'s own rather than any of the academic methods.

## k-quants

Introduced by Iwan Kawrakow in llama.cpp PR #1684 (June 2023). No paper exists; the PR is the primary source.

**Mechanism**: block-wise scaling with **super-blocks**. Weights are grouped into small blocks, each with its own scale (and min, for asymmetric types); those per-block scales are **themselves quantized** within a 256-weight super-block carrying an FP16 scale. This two-level design is directly analogous to [[qlora-dettmers-2023|QLoRA's Double Quantization]].

| Type | Super-block | Weight bits | Scale bits | bpw |
|---|---|---|---|---|
| Q2_K | 16 x 16 | 2 | 4 | 2.5625 |
| Q3_K | 16 x 16 | 3 | 6 | 3.4375 |
| Q4_K | 8 x 32 | 4 | 6 | 4.5 |
| Q5_K | 8 x 32 | 5 | 6 | 5.5 |
| Q6_K | 16 x 16 | 6 | 8 | 6.5625 |

**The `_S` / `_M` / `_L` suffixes are tensor-assignment policies, not block formats.** Sensitive tensors (`attention.wv`, `attention.wo`, `feed_forward.w2`) are kept at higher bit width. So **Q4_K_M** = mostly Q4_K with Q6_K on sensitive tensors, and is llama.cpp's recommended sweet spot.

This is why *effective* bits-per-weight diverges from nominal: Q3_K_S/M/L share a nominal 3.4375 bpw but measure 3.64 / 4.00 / 4.30 effective. See [[quantization-performance]].

k-quants are **calibration-free** by default (round-to-nearest per block).

> [!warning] The "K" does not mean k-means
> A widely repeated claim, including in several blog posts. The originating PR describes super-block scaling with no k-means at all; the actual etymology is undocumented.

## i-quants

`IQ2_XXS`, `IQ3_S`, `IQ4_XS` and friends use **E8/D4 lattice codebooks**, explicitly crediting QuIP# ("the 'codebook' idea comes originally from QuIP# and is also being used in, e.g., AQLM").

Unlike k-quants they effectively **require an importance-matrix (imatrix) calibration pass** at low bit widths. They win clearly at <=3 bits but generally decode slower, especially on CPU and Apple Silicon.

## Related

- [[llama-cpp]] · [[quantization]] · [[quantization-performance]] · [[quantization-conditioned-backdoor]] (GGUF is an attacked format)

## References

- [[which-quantization-kurt-2026]] — a unified evaluation across 13 GGUF formats.
- llama.cpp PR #1684 (k-quants), PRs #4773 / #5196 (i-quants).
