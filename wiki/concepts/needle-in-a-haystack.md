---
title: Needle in a Haystack
type: concept
created: 2026-09-29
updated: 2026-10-06
sources:
  - sale-low-bit-estimation-sparse-attention-prefill-ji-2025.md
  - sts-speculative-token-sparsity-yu-2026.md
  - discovering-the-gems-in-early-layers-shi-2024.md
  - sliminfer-dynamic-token-pruning-long-2025.md
  - misa-mixture-of-indexer-sparse-attention-zhou-2026.md
  - native-sparse-attention-yuan-2025.md
  - can-lms-actually-retrieve-in-context-gollapudi-2026.md
status: draft
importance: medium
---

# Needle in a Haystack

**Needle in a Haystack (NIAH)** is the simplest long-context probe. A single fact, the "needle", is planted at some depth inside a long distractor text, the "haystack", and the model is asked to retrieve it. Results are usually shown as a grid of context length × needle depth. The *passkey* test is a variant with a random key. The original public harness is Greg Kamradt's `needle-in-a-haystack` repository. [[ruler]] extends it with multi-needle, multi-key and multi-value variants, and adds tasks that go beyond retrieval.

## In this wiki

NIAH is mostly used as a **sanity check** for methods that drop or approximate attention. The question is whether sparse selection still finds the one token that matters.

- **Sparse prefill:** [[sale-low-bit-estimation-sparse-attention-prefill-ji-2025|SALE]] keeps NIAH at 96.0% vs 96.1% dense and reports its 3.81× end-to-end speedup on a 64K NIAH input. [[discovering-the-gems-in-early-layers-shi-2024|GemFilter]] *beats* full attention on NIAH (0.887 vs 0.841 on LLaMA 3.1). This is the setting where a single query token defines the whole task. [[sliminfer-dynamic-token-pruning-long-2025|SlimInfer]] is essentially unchanged (0.986 vs 0.984).
- **Sparse decode:** [[sts-speculative-token-sparsity-yu-2026|STS]] uses passkey top-10 recall at 10K context as its probe.
- **Trained or retrofitted sparse attention:** [[misa-mixture-of-indexer-sparse-attention-zhou-2026|MiSA]] runs its ablations on NIAH at 128K. The longest context [[native-sparse-attention-yuan-2025|NSA]] evaluates is 64K NIAH.

> [!warning] Passing NIAH is weak evidence
> Single-needle retrieval is the case that token pruning handles best. [[can-lms-actually-retrieve-in-context-gollapudi-2026|Gollapudi et al.]] argue that NIAH-style synthetic evals overstate real in-context retrieval. Read NIAH results alongside [[ruler]] aggregation tasks or [[longbench]].

## Related

- [[ruler]] · [[longbench]] · [[long-context-llm]] · [[sparse-prefill]] · [[token-pruning]] · [[sparse-attention]]

## References

- Kamradt, needle-in-a-haystack test harness: [github.com/gkamradt/needle-in-a-haystack](https://github.com/gkamradt/needle-in-a-haystack)
- Hsieh et al., RULER (2024), [arXiv:2404.06654](https://arxiv.org/abs/2404.06654). Its abstract describes NIAH as widely adopted but only a superficial test of long-context understanding.

## Lineage sources (added 2026-10-06)

- [[gemini-1-5-google-2024]] — 99% NIAH recall at 1M tokens (vendor claim, Feb 2024)
- [[context-rot]] — why single-needle recall overstates effective context
