---
title: Speculative Prefill
type: concept
created: 2026-09-03
updated: 2026-09-10
sources:
  - speculative-prefill-liu-2025.md
  - cross-family-speculative-prefill-upasani-2026.md
  - draft-based-approximate-inference-galim-2025.md
  - omlx-specprefill-github-2026.md
builds-on:
  - [[sparse-prefill]]
status: complete
importance: high
---

# Speculative Prefill

Use a small **draft model** to estimate which prompt tokens matter, then have the large **target** prefill only those — at their original positions — so time-to-first-token falls by roughly the fraction dropped. Introduced as SpecPrefill by [[speculative-prefill-liu-2025|Liu, Chen & Zhang (ICML 2025)]]; training-free; one family of [[sparse-prefill]].

> [!warning] This is not speculative decoding
> [[speculative-decoding]] uses a draft to propose *output* tokens the target verifies — it accelerates decode and does nothing for prefill. Speculative prefill uses a draft to score *input* tokens so the target prefills a subset — it accelerates prefill and does nothing for decode. Same word, opposite phase. They compose (a draft with an MTP head can serve both), but community advice that mentions "spec prefill with MTP" is describing two levers, not one.

## The estimator

The draft prefills the whole prompt, then decodes **N = 8 lookahead tokens**, capturing its attention. Importance per prompt token is:

1. `softmax(Q_lookahead · K_promptᵀ / √d)` per head, per layer, per lookahead token
2. 1-D **average-pool** (kernel 13) to smooth
3. **max** across layers × heads
4. **mean** across lookahead tokens

Then group into **32-token chunks**, keep the top-K% by mean score, and always keep a **trailing window**. The draft is answering *"if I started generating, what would I look back at?"* — which is why lookahead matters and why the last-token-only variants ([[draft-based-approximate-inference-galim-2025|SpecPC]] uses n=1 with a 64-query window) trade a little quality for speed.

## The measured case

Draft Llama-3.1-8B, targets 70B and 405B-FP8, 8×H200:

| | Result |
|---|---|
| Draft cost | **2.96%** of 405B FLOPs; 14.24% of 70B |
| 405B, 10% keep, 32K prompt | **7.66× TTFT** (theoretical 7.72×); up to 7× QPS at >95% accuracy |
| 70B, 10% keep, LongBench | 52.74 vs 53.55 dense — **98.5%** |
| 70B, 30% keep | 53.92 — *above* dense |
| Code completion, 10% keep | 52.33 → **63.33** |
| Summarization, 10% keep | ~**−16%** relative |
| RULER aggregation (CWE/FWE), 10% keep | 99.7 → **77.9** at 4K — collapses |

The pattern: retrieval and QA are nearly free at 10%; anything requiring broad coverage or counting is not. Against [[minference-dynamic-sparse-attention-jiang-2024|MInference]], SpecPrefill is **2.54–6.54× faster at 99.5% quality** for large-batch prompts under 128K — the regime where attention is not yet dominant and shrinking the FFN wins.

## Positions: the design choice that splits the literature

Liu et al. keep every surviving token at its **original position ID**, so the target's RoPE sees `[0, 1, 3, 6, 7, 10, ...]` and decode starts at the full length M. They call this "crucially essential." [[cross-family-speculative-prefill-upasani-2026|Upasani et al.]] and SpecPC instead **re-index contiguously** (SpecPC explicitly so it can exceed the target's context limit) and claim negligible impact — **neither reports an ablation.** The [[omlx-specprefill-github-2026|oMLX implementation]] follows Liu, with manual RoPE at original positions and an offset-adjusted RoPE for decode.

## Can the draft come from another family?

[[cross-family-speculative-prefill-upasani-2026|Yes, mostly]]: Qwen3 ↔ Llama-3 drafts land within 0–1 points of in-family at 8B-scale targets, and what matters more is whether the draft's *native context length* covers the input (a 32K-context draft on 250K prompts retains only 79–82%). But SpecPC in Galim et al. loses **6.4 points** with a Qwen2.5-0.5B draft for a Llama-70B target — filed as a Contradiction on that page. Production implementations sidestep the question: oMLX requires draft and target to share a tokenizer.

## The standing limitation

**Dropped tokens never get KV.** They are gone for the generation and for any later turn. A follow-up question about dropped content cannot be answered without re-prefilling. The practical mitigations are structural: always prefill the system prompt in full, always keep a trailing window (oMLX added a mandatory 512-token tail after users reported chat-structure tokens being dropped — issues #419, #548, #1401), and cache stable prefixes on the target so they never enter the selection at all.

> [!warning] Field measurement: the draft is a full prefill, and against a sparse target it does not pay (2026-09-10)
> The draft does a complete prefill of the prompt plus its lookahead steps, so its cost is a full forward pass, not a cheap scorer: `speedup = 1 / (c_draft/c_target + keep)`, a win only if `c_draft/c_target < 1 − keep`. Measured on an M4 Air with both sides cache-defeated, dense Qwen3.5-4B draft (2.3 GB, vocab 248,320) against Qwen3.6-35B-A3B:
>
> | Prompt tokens | Draft tok/s | Target tok/s | Cost ratio | Speedup at keep 0.20 |
> |---|---|---|---|---|
> | 1,562 | 236.0 | 381.1 | 1.61 | **0.75×** |
> | 5,981 | 355.0 | 403.7 | 1.14 | **0.79×** |
> | 17,357 | 313.7 | 233.1 | 0.74 | 1.06× |
> | 35,449 | 164.3 | 125.3 | 0.76 | 1.04× |
>
> The draft degrades with length too, so the ratio plateaus near 0.75 rather than opening up. A dense ~3.1B-active draft is simply not cheap relative to a 3B-active target: **prefill cost tracks active parameters on both sides**, and "a 4B draft for a 35B model" is a category error when the 35B is sparse. Against a dense 27B the same draft projects to a 0.16–0.26 ratio and 2.2–2.8× — projected, not measured. Contributed by a peer session; the full pipeline was not run, only the cost ratio that bounds it.

## Related

- [[sparse-prefill]] — the umbrella and the attention-share decision rule.
- [[token-pruning]] — the same idea with the target's own layers as selector, no draft.
- [[sparse-attention]] — keep tokens, skip attention entries; composes with this.
- [[speculative-decoding]] — the decode-phase technique this is routinely confused with.
- [[prompt-compression]] · [[prefix-caching]] · [[kv-cache]] · [[rope-scaling]] · [[methods-to-speed-up-prompt-processing]]

## References

- [[speculative-prefill-liu-2025]] · [[cross-family-speculative-prefill-upasani-2026]] · [[draft-based-approximate-inference-galim-2025]] · [[omlx-specprefill-github-2026]]
