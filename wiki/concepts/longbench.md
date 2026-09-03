---
title: LongBench
type: concept
created: 2026-09-03
updated: 2026-09-03
sources:
  - speculative-prefill-liu-2025.md
  - lazyllm-dynamic-token-pruning-fu-2024.md
  - sliminfer-dynamic-token-pruning-long-2025.md
status: draft
importance: medium
tags:
  - benchmark
---

# LongBench

The default benchmark for long-context *understanding* — as distinct from RULER-style synthetic *retrieval*. Sixteen to seventeen datasets across six task categories (single- and multi-document QA, summarization, few-shot learning, synthetic, and code), with average prompts of a few thousand tokens; **LongBench v2** extends to ~250K-token prompts.

It appears in nearly every [[sparse-prefill]] paper because its task mix exposes the family's central weakness: **retrieval-style tasks survive aggressive token dropping and coverage-style tasks do not.** The same 10% keep that leaves multi-doc QA untouched costs ~16% on summarization and 11 points on NarrativeQA.

Two cautions when reading LongBench numbers across papers. The dataset count varies (14 English-only vs 16 vs 17-with-code), so averages are not directly comparable across papers. And LongBench prompts are short enough (~3–8K) that **attention is not the dominant prefill cost** — which is why [[sparse-attention]] methods measure *slower than dense* on it while [[token-pruning]] methods show gains. LongBench speedups say nothing about the 128K–1M regime.

## Related

- [[long-context-llm]] · [[sparse-prefill]] · [[token-pruning]] · [[speculative-prefill]] · [[sparse-attention]]
- [[mmlu]] · [[humaneval]] · [[swe-bench]] — other benchmark pages.

## References

- [[speculative-prefill-liu-2025]] · [[lazyllm-dynamic-token-pruning-fu-2024]] · [[sliminfer-dynamic-token-pruning-long-2025]] · [[discovering-the-gems-in-early-layers-shi-2024]]
