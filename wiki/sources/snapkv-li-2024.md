---
title: "SnapKV: LLM Knows What You are Looking for Before Generation"
type: source
created: 2026-10-23
updated: 2026-10-23
sources:
  - snapkv-li-2024.md
tags:
  - 2024
  - paper
status: complete
importance: medium
authors:
  - Yuhong Li
  - Yingbing Huang
  - Bowen Yang
  - Bharat Venkitesh
  - Acyr Locatelli
  - Hanchen Ye
  - Tianle Cai
  - Patrick Lewis
  - Deming Chen
introduces:
  - [[kv-cache-eviction]]
year: 2024
venue: arXiv preprint (under review)
arxiv_id: "2404.14469"
---

# SnapKV

[arXiv:2404.14469](https://arxiv.org/abs/2404.14469).

| | |
|---|---|
| **Authors** | Li, Huang (equal contribution), Yang, Venkitesh, Locatelli, Ye, Cai, Lewis, Chen |
| **Affiliations** | UIUC, Cohere, Princeton |

## Idea

Each attention head consistently attends to particular prompt positions during generation, and that pattern **is already visible from an "observation window" at the end of the prompt**. SnapKV uses it to compress the **prompt** KV **once, at prefill**, with no fine-tuning ([[kv-cache-eviction]]).

**How positions are chosen:**
1. **Vote.** Sum the observation window's attention onto earlier positions, then keep the top-k per head.
2. **Pool.** 1D pooling (kernel 5–13) before top-k keeps neighbouring tokens. Otherwise the model might "retrieve only the country code of a phone number and hallucinate the rest".
3. **Assemble.** The cache is the selected prefix KV plus the whole observation window. The window is 16–64 tokens and the budget 1,024–4,096.

**Generated tokens are not compressed.** SnapKV targets the prompt KV that H2O and similar methods overlook.

## Results

- **Speed and memory at 16K input:** "3.6x increase in generation speed and an 8.2x enhancement in memory efficiency".
- **Context limit:** "up to 380K context tokens on a single A100-80GB" with a "negligible accuracy drop" on needle-in-a-haystack. The unmodified model runs out of memory at 33K. Retrieval dips slightly after about 140K.
- **LongBench** (inputs of about 13K tokens):
  - A 1K budget (≈ 92% compression) and a 4K budget (68%) both show "negligible drops".
  - SnapKV at 1K beats H2O at 4K on 11 of 16 tasks.
- **Command-R (35B):**
  - Needle at 128K: 9.866 → 9.819 at 32× compression.
  - RAG citation F1 −1.2%.
- **With Medusa:** 2.2× faster than native decoding at 10K.

## Limitations (stated)

- **No help for weak long-context models:** it "cannot extend a model's long context capability".
- **Prefill unchanged:** it "does not cover the processing of the prompt inference".
- **One compression, many questions.** The paper's own §4.2.1 shows that **different instructions on the same document attend to different positions**. That undercuts static, one-shot compression for multi-turn use. The paper doesn't test multi-turn.

## Related

[[kv-cache-eviction]] · [[h2o-heavy-hitter-oracle-zhang-2023]] · [[kv-cache]] · [[sparse-prefill]] · [[token-pruning]] · [[prefix-caching]]
