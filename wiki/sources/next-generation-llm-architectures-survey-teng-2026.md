---
title: "Towards Next-Generation Architectures of Large Language Models: A Survey"
type: source
created: 2026-11-19
updated: 2026-11-19
sources:
  - next-generation-llm-architectures-survey-teng-2026.md
authors:
  - Yao Teng
  - Guoxuan Chen
  - Yimeng Chen
  - Han Shi
  - Weiyang Liu
  - Jürgen Schmidhuber
  - et al.
year: 2026
venue: "alphaXiv (2611.next-generation-llm-architectures-survey)"
status: complete
importance: medium
tags:
  - 2026
  - survey
---

# Towards Next-Generation Architectures of Large Language Models: A Survey

**Yao Teng, Guoxuan Chen** (equal contribution), **Yimeng Chen, Han Shi** (corresponding) and 12 others, from [[hku]], KAUST, Huawei, [[bytedance-seed]], [[chinese-university-of-hong-kong]], [[peking-university]], MPI-IS and IDSIA ([[jurgen-schmidhuber]]). The project page is `surveyllmarch.github.io`. The main text runs to about 83 pages before the references.

> [!note] No arXiv ID
> This survey is **alphaXiv-only and has no arXiv ID**. It was read from the full PDF at https://www.alphaxiv.org/abs/2611.next-generation-llm-architectures-survey.pdf, which downloaded normally.

## Summary

The survey covers what has changed in LLM architecture beyond the canonical decoder-only [[transformer-architecture|Transformer]]. It goes component by component, with one section each on efficient attention and KV-cache management, linear and recurrent sequence models, [[mixture-of-experts]], activations, normalization and residuals, [[positional-encoding]], tokenization, multi-token prediction, and adaptive or recurrent depth.

Section 14 then traces open-weight model families. It finds a shift "from establishing a common dense backbone to configuring capacity, state, and prediction jointly". Section 15 reads the whole field as a relaxation of four implicit assumptions of the canonical decoder.

## Taxonomy: four assumptions of the canonical decoder (Table 15)

| Direction | Canonical assumption | Alternatives surveyed |
|---|---|---|
| Flexible and conditional processing | Fixed tokens and positions; one operator repeated; every token uses the same dense blocks and the same depth | Adaptive units, hybrid operators, MoE routing, early exit, conditional or recurrent depth |
| Refined transformation dynamics | Fixed FFN nonlinearity (e.g. SwiGLU); fixed residual and norm paths | Squared ReLU, clamped SwiGLU, new norm placement, residual scaling or gating |
| Selective and flexible state management | History kept as token-level states; append-only cache | KV compression, eviction, merging; recurrent or hybrid state; branching and rollback |
| Lookahead and delayed commitment | Each state predicts only the next token; one token committed per pass | Multi-token or blockwise prediction; drafting, verification, delayed commitment |

## Key points

- **Recurring patterns in open-weight models (Sec. 14.2).**
  - Canonical dense decoders: the [[llama]] line, with Pre-RMSNorm, SwiGLU and RoPE.
  - Sparse capacity with compressed attention state: [[deepseek-v2]], [[deepseek-v3]], Kimi K2, [[glm-5]], built on MoE plus [[multi-head-latent-attention|MLA]].
  - Hybrid sequence and state processing: [[griffin]], [[jamba]], Hymba, Nemotron-H.
  - Prediction-augmented backbones, for example the auxiliary MTP module in DeepSeek-V3.
- **Two separate budgets.** MoE separates total parameters from active ones. KV compression changes the stored history. Neither one fixes the other, so they often appear together. The authors note that this "does not by itself establish" that either mechanism improves the other.
- **When selection happens.** Sparse reads select at retrieval time. Eviction selects at storage time. Merging builds aggregate entries. Recurrent models compress as they go.
- **Speculative decoding** changes the execution schedule, "not the dependency structure of the model", as long as verification is exact.
- **Coupling.** Recurrent depth needs stable repeated transforms, so depth propagation (norms and residuals) constrains computation allocation. Long drafts need temporary states that may branch or roll back, and aggressive KV compression can make rollback harder.
- **Infrastructure matters.** [[minimax]] moved from a Lightning-Attention hybrid (MiniMax-01) to full attention (M2), and later to block-sparse attention (M3). The stated reasons were evaluation uncertainty and immature kernels.

## What it does not show

- It reports no new experiments. The patterns come from model releases, and the authors say releases do not isolate the causal effect of any one component.
- The four-way framing is analytic. The authors note that the dimensions overlap and that one mechanism can change several of them.

## Concepts & entities

- [[transformer-architecture]] · [[decoder-only-transformer]] · [[multi-head-attention]] · [[grouped-query-attention]] · [[multi-head-latent-attention]] · [[sliding-window-attention]] · [[sparse-attention]]
- [[kv-cache]] · [[kv-cache-eviction]] · [[kv-cache-quantization]] — Sec. 5.
- [[linear-attention]] · [[state-space-model]] · [[gated-deltanet]] · [[mamba]] · [[rwkv]] · [[xlstm]] — Sec. 6.
- [[mixture-of-experts]] — Sec. 7. [[layer-normalization]] · [[residual-connection]] · [[hyper-connections]] — Sec. 9.
- [[positional-encoding]] · [[rope-scaling]] · [[byte-pair-encoding]] — Secs. 10–11.
- [[multi-token-prediction]] · [[speculative-decoding]] — Sec. 12.
- [[layer-skipping]] · [[adaptive-computation-time]] · [[looped-transformer]] · [[recurrent-depth-transformer]] — Sec. 13.

## References

- alphaXiv: https://www.alphaxiv.org/abs/2611.next-generation-llm-architectures-survey.pdf (no arXiv ID)
- Project page: https://surveyllmarch.github.io
- Cited anchors: DeepSeek-V2/V3/V3.2 (DeepSeek-AI), Jamba (Lieber et al.), Griffin (De et al.), Hymba, MambaFormer, MiniMax-01/M2/M3.
