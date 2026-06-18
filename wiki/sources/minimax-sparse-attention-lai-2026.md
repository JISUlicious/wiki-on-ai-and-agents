---
title: "MiniMax Sparse Attention"
type: source
created: 2026-06-18
updated: 2026-06-18
sources:
  - minimax-sparse-attention-lai-2026.md
arxiv_id: "2606.13392"
authors:
  - Xunhao Lai
  - et al.
first_author: Xunhao Lai
year: 2026
venue: arXiv preprint (MiniMax)
tags:
  - 2026
status: complete
importance: medium
---

# MiniMax Sparse Attention

Surfaced via the alphaXiv weekly digest (2026-06-17).

## Summary

MiniMax Sparse Attention (MSA) is a blockwise sparse attention mechanism built on Grouped Query Attention (GQA), aimed at making ultra-long-context LLMs (hundreds of thousands to millions of tokens) tractable at deployment scale, where quadratic softmax attention becomes the dominant cost. MSA splits attention into a lightweight **Index Branch** that scores key–value blocks and independently selects a Top-k subset for each GQA group, and a **Main Branch** that performs exact block-sparse attention over only the selected blocks (the local block is always included). During training a KL loss aligns the Index Branch distribution with the group-averaged Main Branch distribution, with the index gradient detached from the main branch.

The design follows a simplicity/scalability principle (Occam's razor over ablated components) and is co-designed with a GPU execution path using exp-free Top-k selection and KV-outer sparse attention to keep tensor cores well utilized under block-granular access. On a 109B-parameter natively-multimodal model, MSA matches dense GQA quality while cutting per-token attention compute by 28.4x at 1M context, and the co-designed kernel delivers 14.2x prefill and 7.6x decoding wall-clock speedups on H800. The inference kernel is open-sourced, and a production model (MiniMax-M3) powered by MSA was released.

## Key points

- Blockwise sparse attention over GQA: per-group Top-k block selection enables group-specific sparse retrieval with block-level execution.
- Two-branch design: lightweight Index Branch scores/selects KV blocks; Main Branch does exact attention over selected blocks only (local block always kept).
- Training uses a KL loss aligning index and main-branch distributions; index gradient is detached.
- 28.4x reduction in per-token attention compute at 1M context, on par with dense GQA in quality (109B native-multimodal model).
- Co-designed GPU kernel (exp-free Top-k, KV-outer sparse attention) gives 14.2x prefill / 7.6x decode speedups on H800.
- Open-source inference kernel; powers the publicly released MiniMax-M3 multimodal model.

## Concepts & entities

- [[long-context-llm]] — MSA targets ultra-long (up to 1M-token) context efficiency.
- [[self-attention]] — MSA is a sparse approximation of softmax self-attention.
- [[kv-cache]] — block selection and KV-outer attention reduce KV access cost at decode time.
- [[minimax-m2-series-minimax-2026]] — same lab (MiniMax); MSA powers the MiniMax-M3 model.
- GQA, sparse attention, blockwise Top-k selection — plain text (no dedicated pages).

## References

- [arXiv:2606.13392](https://arxiv.org/abs/2606.13392) — "MiniMax Sparse Attention," Lai et al., 2026 (MiniMax).
- Inference kernel: github.com/MiniMax-AI/MSA. Model: huggingface.co/MiniMaxAI/MiniMax-M3.
- Local source text: `minimax-sparse-attention-lai-2026.md`.
