---
title: Token Pruning
type: concept
created: 2026-09-03
updated: 2026-09-03
sources:
  - lazyllm-dynamic-token-pruning-fu-2024.md
  - discovering-the-gems-in-early-layers-shi-2024.md
  - sliminfer-dynamic-token-pruning-long-2025.md
  - probe-and-skip-self-predictive-token-skipping-wu-2026.md
  - claa-cross-layer-attention-aggregation-mcdanel-2026.md
builds-on:
  - [[sparse-prefill]]
status: complete
importance: high
---

# Token Pruning

Drop prompt tokens from the sequence *during* prefill using the **target model's own early-layer attention** as the selector — no draft model. One family of [[sparse-prefill]]; the sibling of [[speculative-prefill]], which externalizes the same decision to a small model.

The shared insight is that a partial forward pass already knows what matters. By layer ~13 of 32, the last token's attention over the prompt is a reliable importance signal ([[discovering-the-gems-in-early-layers-shi-2024|GemFilter]] finds it flat from layer 13 to 25, poor at layer 1 and at layer 31). So you can score at that layer and run the remaining layers on a fraction of the tokens.

## The methods differ on three axes

| Method | Static or dynamic? | Dropped tokens' KV | Selector |
|---|---|---|---|
| [[discovering-the-gems-in-early-layers-shi-2024\|GemFilter]] (2024) | **static**, once | **discarded** — pass 1 thrown away, pass 2 never sees them | one fixed layer (13/32 LLaMA-3.1); last-token pre-softmax Q·Kᵀ |
| [[lazyllm-dynamic-token-pruning-fu-2024\|LazyLLM]] (Apple, 2024) | **dynamic per decode step, with revival** | never computed at the pruning layer; hidden states parked in an Aux Cache so revival resumes there | every pruning layer, using the previous layer's last-token attention |
| [[sliminfer-dynamic-token-pruning-long-2025\|SlimInfer]] (2025) | **dynamic, with revival** | **kept**, offloaded to CPU and prefetched back | Slim layers 10/20/30; 64-token blocks; swaps suppressed unless overlap < 0.9 |
| [[probe-and-skip-self-predictive-token-skipping-wu-2026\|SPTS]] (2026) | **static w.r.t. decode**; layer-wise skipping *within* prefill | never computed at skipped layers; cache is layer-ragged | each skip layer probes *itself* plus a low-rank proxy FFN |
| [[claa-cross-layer-attention-aggregation-mcdanel-2026\|CLAA]] (2026) | static, once at layer 15 | discarded | 4-layer rolling max of FastKV's recent-token attention |

All five genuinely reduce **prefill FLOPs** — pruned tokens skip attention *and* FFN in every later layer — which distinguishes this family from KV-cache *eviction* (SnapKV, H2O), which prefills everything and then forgets.

**Static vs. dynamic is the important axis.** A static method decides once and can never recover a token a later decode step needs; that is why GemFilter's NarrativeQA falls 32.0 → 20.7 and summarization is where every static method loses most. Dynamic methods pay for revival with bookkeeping: LazyLLM's schedule is never disclosed, and SlimInfer never explains how a block pruned *during* prefill gets KV for the layers it skipped.

**Skipping vs. pruning** is SPTS's contribution: an unselected token passes through the residual shortcut `Y = X + F(X)` and *stays in the sequence*, so it can be re-eligible at the next layer without any cache. Selection then costs only a K projection over all candidates.

## Measured

LongBench averages, LLaMA-3.1-8B, all at comparable TTFT gain:

| | Score (dense 47.98) | TTFT |
|---|---|---|
| SPTS | **47.80** | 1.68× |
| SlimInfer | 47.56 | 1.62× |
| LazyLLM | 47.13 | 1.63× |
| GemFilter | 47.09 | 1.31× |
| MInference (sparse attention, for contrast) | 47.55 | **0.47×** — slower than dense |

At 32K: SlimInfer 2.53× TTFT on an RTX 4090; SPTS 2.46× on an A800. At **8K or below, end-to-end gains vanish** (SlimInfer 1.00× at 8K, 0.93× at 4K on Jetson Orin) — the selection overhead needs enough prompt to amortize.

The MInference row is the [[sparse-prefill|attention-share]] rule in one number: at LongBench prompt lengths, sparsifying attention alone costs more than it saves, while dropping tokens shrinks the FFN that actually dominates.

> [!warning] Reporting hygiene in this literature
> GemFilter's "1000×" title is a single Figure-1 demo (108,172 → 100 tokens); every benchmark keeps ≥ 1,024. SlimInfer's 2.53× and SPTS's 2.29× E2E exist only as figures. SPTS's Qwen baseline rows are digit-for-digit copies of SlimInfer's table, and its "training-free" claim rests on an offline SVD calibrated on the evaluation benchmark's own Qasper split. Read the tables, not the abstracts.

## Related

- [[sparse-prefill]] · [[speculative-prefill]] · [[sparse-attention]]
- [[kv-cache]] · [[prefix-caching]] · [[prompt-compression]] · [[attention-sink]] · [[residual-stream]]
- [[methods-to-speed-up-prompt-processing]] · [[llm-inference-roofline-yuan-2024]]

## References

- [[lazyllm-dynamic-token-pruning-fu-2024]] · [[discovering-the-gems-in-early-layers-shi-2024]] · [[sliminfer-dynamic-token-pruning-long-2025]] · [[probe-and-skip-self-predictive-token-skipping-wu-2026]] · [[claa-cross-layer-attention-aggregation-mcdanel-2026]]
