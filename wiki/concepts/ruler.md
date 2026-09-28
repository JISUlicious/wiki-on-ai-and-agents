---
title: RULER
type: concept
created: 2026-09-29
updated: 2026-09-29
sources:
  - uniprefill-block-wise-dynamic-sparsification-fan-2026.md
  - gated-sparse-attention-shen-2026.md
  - trianglemix-decoding-time-contribution-sparsity-he-2025.md
  - speculative-prefill-liu-2025.md
  - cross-family-speculative-prefill-upasani-2026.md
  - draft-based-approximate-inference-galim-2025.md
  - kimi-linear-kimi-team-2025.md
  - nemotron-3-puzzle-75b-a9b-nvidia-2026.md
  - long-context-quantization-mekala-2025.md
  - can-lms-actually-retrieve-in-context-gollapudi-2026.md
status: draft
importance: medium
---

# RULER

**RULER** (Hsieh et al., NVIDIA, 2024) is a synthetic long-context benchmark with configurable sequence length and task complexity. It builds on the [[needle-in-a-haystack]] test by adding needles of several types and in larger numbers, plus two new categories: **multi-hop tracing** (variable tracking) and **aggregation** (common- and frequent-word extraction, CWE/FWE). The aim is to test more than retrieval from context. The paper's headline question is a model's "real" context size.

## In this wiki

RULER is the most common long-context evaluation in the wiki's sparse-attention and prefill-compression sources. The [[uniprefill-block-wise-dynamic-sparsification-fan-2026|UniPrefill]] page calls it "now the standard long-context suite".

- **Prefill compression and sparsification:** [[speculative-prefill-liu-2025|SpecPrefill]], [[draft-based-approximate-inference-galim-2025|SpecKV/SpecPC]], [[cross-family-speculative-prefill-upasani-2026|cross-family SpecPrefill]], [[uniprefill-block-wise-dynamic-sparsification-fan-2026|UniPrefill]] and [[trianglemix-decoding-time-contribution-sparsity-he-2025|TriangleMix]]. A pattern repeats: **retrieval and QA hold up, and aggregation collapses**, because every token matters for CWE/FWE. The UniPrefill page notes that reporting RULER alone flatters token pruning.
- **Architectures:** [[gated-sparse-attention-shen-2026|GSA]] reports RULER at 128K only as YaRN extrapolation. [[kimi-linear-kimi-team-2025|Kimi Linear]] scores 84.3 against 81.3 for its [[multi-head-latent-attention|MLA]] baseline.
- **Compression that preserves the model:** [[nemotron-3-puzzle-75b-a9b-nvidia-2026|Nemotron 3 Puzzle]] stays within 1–2 points at 256K–1M. [[long-context-quantization-mekala-2025|Mekala et al.]] measure how much RULER accuracy quantization costs.

> [!warning] Synthetic scores overstate real retrieval
> [[can-lms-actually-retrieve-in-context-gollapudi-2026|Gollapudi et al.]] note that a model reporting near-perfect RULER NIAH degraded sharply on real in-context retrieval. They argue that NIAH/RULER-style synthetic evals overstate real retrieval capability.

## Related

- [[needle-in-a-haystack]] · [[long-context-llm]] · [[longbench]] · [[sparse-prefill]] · [[token-pruning]] · [[rope-scaling]]

## References

- Hsieh et al., "RULER: What's the Real Context Size of Your Long-Context Language Models?" (2024), [arXiv:2404.06654](https://arxiv.org/abs/2404.06654)
- The wiki sources listed in frontmatter.
