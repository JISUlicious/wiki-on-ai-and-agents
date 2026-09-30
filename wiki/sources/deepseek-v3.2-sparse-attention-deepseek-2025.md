---
title: "DeepSeek-V3.2: DeepSeek Sparse Attention and the Lightning Indexer"
type: source
created: 2026-09-10
updated: 2026-09-30
sources: []
arxiv_id: "2512.02556"
authors:
  - DeepSeek-AI
year: 2025
status: complete
importance: high
tags:
  - 2025
  - paper
---

# DeepSeek-V3.2: DeepSeek Sparse Attention and the Lightning Indexer

[arXiv:2512.02556](https://arxiv.org/abs/2512.02556) — *"DeepSeek-V3.2: Pushing the Frontier of Open Large Language Models,"* [[deepseek|DeepSeek-AI]], 2 December 2025. This page covers **DeepSeek Sparse Attention (DSA)** only; the RL recipe and benchmark suite get one paragraph at the end. DSA is the third member of the [[trained-sparse-attention]] lineage after [[native-sparse-attention-yuan-2025|NSA]] and [[moba-mixture-of-block-attention-lu-2025|MoBA]], and the first shipped in a frontier open model.

## The problem DSA solves differently

NSA trains sparsity in from scratch; MoBA switches a dense model to block attention with fine-tuning. DSA is a **retrofit onto an already-trained dense model** (V3.1-Terminus) that keeps per-token, not per-block, selection — and pays for the selector with a separate, cheap network rather than with the main attention's own scores.

## Mechanism

**Lightning indexer.** A small side network with a handful of heads computes, for each query token *t* and every preceding token *s*:

`I(t,s) = Σ_j w^I(t,j) · ReLU(q^I(t,j) · k^I(s))`

The ReLU and the small head count make it cheap enough to run in **FP8**. Its complexity is still O(L²) — the paper is explicit that it "requires much less computation compared with MLA," not that it is sub-quadratic. Every efficiency claim in this paper rests on that constant factor.

**Fine-grained selection.** For each query, the **top-2,048** entries by index score are selected, per token rather than per block, and main attention runs only over those. Complexity of the main attention drops from O(L²) to O(L·k). DSA is instantiated under [[multi-head-latent-attention|MLA]]'s MQA mode, so each selected latent vector is shared across all query heads — which is what makes gathering 2,048 entries per query affordable.

## How a dense model was adapted, in two stages

| Stage | Steps | Seqs/step | Length | Tokens | LR | What trains |
|---|---|---|---|---|---|---|
| **Dense warm-up** | 1,000 | 16 | 128K | **2.1B** | 1e-3 | indexer only; main model frozen; loss = KL between indexer scores and the aggregated main-attention distribution |
| **Sparse training** | 15,000 | 480 | 128K | **943.7B** | 7.3e-6 | whole model + indexer, indexer loss detached |

The warm-up teaches the indexer to imitate the dense model's own attention. The sparse stage then lets the model adapt to seeing only what the indexer picks. Roughly a trillion tokens of continued pre-training is the price of retrofitting sparsity onto a frontier dense model — far more than MoBA's fine-tune, far less than NSA's from-scratch run.

## Cost and quality

- **Inference cost** (Figure 3, H800 at $2/GPU-hour): prefill and decode cost per token are plotted against position for V3.2 vs V3.1-Terminus. The paper describes the DSA curves as growing far more slowly with context; the exact per-token dollar values are in the figure, not the text, and are not transcribed here. Vendor commentary describing the prefill curve as "almost flat to 128K" is a reading of that figure, not a number from the paper.
- **Short context**: "for short-sequence prefilling, we specially implement a masked MHA mode to simulate DSA." That is, below some length the indexer-plus-gather path is not worth it and dense attention with a mask is used instead — the same crossover every method on [[sparse-prefill]] has.
- **Quality**: "we do not observe substantial performance degradation compared with DeepSeek-V3.1-Terminus, on both short- and long-context tasks." Chatbot Arena Elo "closely matched"; on AA-LCR long-context reasoning V3.2-Exp scores four points *higher* in reasoning mode.

> [!warning] What the paper does not give
> No per-token cost numbers in text, no end-to-end TTFT, no ablation of *k*, and no task-level breakdown of where sparse selection hurts. The indexer's own O(L²) cost is acknowledged but never separated out in a table. Treat "DSA makes long context cheap" as supported by the architecture and the parity results, not by a published latency table.

The rest of the paper — the RL training pipeline, reasoning-mode results, the V3.2 vs V3.2-Speciale variants — is outside this page's scope.

## Related

- [[trained-sparse-attention]] — the concept page; NSA, MoBA and DSA compared.
- [[native-sparse-attention-yuan-2025]] · [[moba-mixture-of-block-attention-lu-2025]] — the two predecessors.
- [[flashmemory-lookahead-sparse-attention-wang-2026]] — DeepSeek-V4's successor to the indexer: lookahead selection for ultra-long context.
- [[sparse-attention]] · [[sparse-prefill]] · [[minference-dynamic-sparse-attention-jiang-2024]] — the training-free retrofits DSA competes with.
- [[deepseek]] · [[deepseek-r1]] · [[long-context-llm]] · [[kv-cache]] · [[arithmetic-intensity]]

## References

- [arXiv:2512.02556](https://arxiv.org/abs/2512.02556)

## Related 2026 sources

- [[misa-mixture-of-indexer-sparse-attention-zhou-2026]] — skips 56 of the indexer's 64 heads per query, training-free, at LongBench 50.85 vs 51.05
- [[gated-sparse-attention-shen-2026]] — assumes the indexer has 4 heads; the released config says 64 — see the contradiction on [[trained-sparse-attention]]
- [[deepseek-v4.1-flash-kv-cache-compression-deepseek-2026]] — DeepSeek-V4.1-Flash: encoder–decoder prefill split, cross-layer sparse KV reuse, FP4 KV; indexer cut to 32 heads, top-k 512
