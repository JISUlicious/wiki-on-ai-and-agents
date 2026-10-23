---
title: "Mistral 7B"
type: source
created: 2026-10-23
updated: 2026-10-23
sources:
  - mistral-7b-jiang-2023.md
tags:
  - 2023
  - paper
status: complete
importance: medium
authors:
  - Albert Q. Jiang
  - Alexandre Sablayrolles
  - Arthur Mensch
  - Chris Bamford
  - Devendra Singh Chaplot
  - Diego de las Casas
  - Guillaume Lample
  - Lucile Saulnier
  - William El Sayed
introduces:
  - [[mistral-7b]]
  - [[sliding-window-attention]]
year: 2023
venue: arXiv preprint
arxiv_id: "2310.06825"
---

# Mistral 7B

[arXiv:2310.06825](https://arxiv.org/abs/2310.06825), 2023-10-10, by [[mistral-ai]]: Jiang, Sablayrolles, Mensch, Bamford, Chaplot, de las Casas, Bressand, Lengyel, Lample, Saulnier and 8 others. Apache-2.0.

## Summary

[[mistral-7b|Mistral 7B]] is a dense 7B model that beats Llama 2 13B on every reported benchmark. It combines **[[grouped-query-attention|GQA]]** with **[[sliding-window-attention|sliding-window attention (SWA)]]** and a **rolling-buffer [[kv-cache]]**.

**Configuration:**
- dim 4096, 32 layers, head_dim 128, FFN hidden size 14336;
- **32 query heads / 8 KV heads**;
- **window W = 4096**, context length 8192, vocabulary 32K.

That works out to 2·8·128·32 = 65,536 elements, or **128 KiB per token** at fp16 (this page's arithmetic).

## Sliding window and rolling buffer

- **Span through depth.** Each layer attends to the previous W positions, but information propagates up the stack: "theoretical attention span of approximately 131K tokens" (4096 × 32 layers).
- **Rolling buffer.** The cache has fixed size W; position i is stored in slot i mod W. "On a sequence length of 32k tokens, this reduces the cache memory usage by 8x, without impacting the model quality."
- **Chunked prefill.** The prompt is processed in chunks the size of the window.
- **Speed.** Changes to FlashAttention/xFormers give 2× over vanilla attention at 16K.

## Results

| | MMLU | HellaSwag | HumanEval | MBPP | MATH | GSM8K |
|---|---|---|---|---|---|---|
| **Mistral 7B** | **60.1** | **81.3** | **30.5** | **47.5** | **13.1** | **52.2** |
| Llama 2 13B | 55.6 | 80.7 | 18.9 | 35.4 | 6.0 | 34.3 |

- **Size equivalence.** On reasoning, comprehension and STEM it is equivalent to a Llama 2 more than 3× its size; on knowledge benchmarks, only about 1.9×.
- **Instruct model:** MT-Bench 6.84, against 6.65 for Llama 2 13B Chat.

## Caveats

- **The quality claim isn't tested.** No ablation in the paper supports "without impacting the model quality" for SWA or the rolling buffer.
- **The 8× saving is just 32K ÷ 4K.** It applies only beyond the window.
- **The 131K span isn't evaluated.** The declared context is 8192, and the paper has no long-context results for the theoretical span.
- **Different evaluation protocol from Llama 2's.**

## Related

[[sliding-window-attention]] · [[mistral-7b]] · [[mistral-ai]] · [[grouped-query-attention]] · [[kv-cache]] · [[kv-cache-size-vs-active-parameters]]
