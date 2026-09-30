---
title: "Recurrent Looped Transformer"
type: source
created: 2026-09-30
updated: 2026-09-30
sources:
  - recurrent-looped-transformer-zhang-2026.md
authors:
  - Yifan Zhang
year: 2026
venue: "alphaXiv (2609.recurrent-looped-transformer); design report dated 12 Sep 2026"
status: complete
importance: low
tags:
  - 2026
  - paper
---

# Recurrent Looped Transformer

**Yifan Zhang** (single author; no affiliation given), dated 12 September 2026. The project page is `github.com/yifanzhang-pro/recurrent-looped-tranformer`.

> [!note] No arXiv ID
> This paper is **alphaXiv-only and has no arXiv ID**. It was read from the full PDF at https://www.alphaxiv.org/abs/2609.recurrent-looped-transformer.pdf, which downloaded normally. Unlike [[dspark-confidence-scheduled-speculative-decoding-deepseek-2026|DSpark]], this page is therefore based on the paper itself, not on coverage of it.

## Summary

This is a **design specification with no experiments**. The report says it "does not report measured efficiency or scaling results".

RLT pairs a causal **encoder** with a **recurrent decoder**. The encoder builds global KV memory that the decoder cross-attends to, similar to YOCO. The decoder carries its **final hidden state s_t and its layer-wise SWA cache** from each token to the next, across the prompt and the response alike, without a reset at the serving boundary.

Each token runs a fixed L_E + L_D blocks: 96 for a 48 + 48 model, or two logical passes through one backbone when weights are tied. The state path to token t, however, passes through t·L_D decoder blocks. This is the paper's claim of **"infinite temporal depth"** at constant work per token.

The rest of the report works through what this recurrence means for hardware and for RL.

## Key points

- **Merge.** The previous decoder output s_{t−1} is merged with the current encoder representation e_t through a gate (W_g ∈ R^{d×2d}). Each decoder block runs causal SWA, then cross-attention to encoder memory, then an FFN.
- **Tied configuration.** The encoder self-attention and FFN weights are reused by the decoder's SWA and FFN. Only cross-attention has its own Q and O projections. The report describes this as "parameter reuse with different attention wiring, not activation copying".
- **Invariance to the serving split (Prop. 3.1).** Batched encoder prefill followed by recurrent decoder updates gives exactly the same states as incremental processing. Where the prompt ends and the response begins does not change the conditional distribution.
- **Prefill cost.** The decoder is inherently sequential over all T prompt tokens, and no parallel scan is assumed. The report states explicitly that "no reduced-prefill speedup is claimed". This is the **opposite trade-off** to [[deepseek-v4.1-flash-kv-cache-compression-deepseek-2026|DeepSeek-V4.1-Flash's CED]] and [[hysparse2-hybrid-sparse-attention-two-level-kv-sharing-wei-2026|HySparse2]], both of which skip the decoder during prefill.
- **Hardware co-design.** The report lists these, but only as targets:
  - Encoder work parallel over tokens.
  - Decoder transitions batched *across* sequences.
  - Reuse of resident weights and KV.
  - Activation checkpointing with full BPTT.
- **RL co-design.** Pretraining, SFT, sampling and RL replay all use the same transition. After each parameter update the trainer **rebuilds every recurrent state and SWA cache from the full history under the current parameters**, rather than reusing stale rollout states. It keeps the sampler's behaviour log-probabilities, including temperature and truncation. The report notes that top-k/top-p sampling breaks the support condition needed for exact importance sampling.

## What it does not show

- There are no trained models, benchmarks, throughput or latency numbers. Every efficiency statement is labelled an "implementation target".
- The report states that structural depth "is not a reasoning guarantee". Whether useful latent reasoning emerges along the t·L_D path is not tested.
- It does not quantify the cost of sequential prefill over long prompts.

## Concepts & entities

- [[looped-transformer]] · [[recurrence]] · [[recurrent-depth-transformer]] · [[universal-transformer]] — RLT advances its state across tokens, whereas these loop over depth.
- [[latent-reasoning]] — the stated motivation.
- [[kv-cache]] — two stores: immutable encoder-derived global memory, and a bounded decoder SWA cache per layer.
- [[reinforcement-learning]] · [[grpo]] — the analysis of replay fidelity and importance ratios.
- [[thinking-with-looped-flows-suleymanzade-2026]] — attacks the same problem of training recurrence from the other direction: local losses without BPTT, where RLT uses full BPTT with checkpointing.

## References

- alphaXiv: https://www.alphaxiv.org/abs/2609.recurrent-looped-transformer.pdf (no arXiv ID)
- YOCO: Sun et al. 2024. DeepSeek-V4.1-Flash: DeepSeek-AI 2026. Feedback Transformer: Fan et al. 2020. Recurrent Transformer: Oncescu et al. 2026.
- Universal Transformers: Dehghani et al. 2018. Recurrent-depth latent reasoning: Geiping et al. 2025. Execution-level policy definition: Zhang et al. 2026.
