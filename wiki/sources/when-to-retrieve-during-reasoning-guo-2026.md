---
title: "When to Retrieve During Reasoning: Adaptive Retrieval for Large Reasoning Models"
type: source
created: 2026-06-03
updated: 2026-06-03
sources:
  - when-to-retrieve-during-reasoning-guo-2026.pdf
arxiv_id: "2604.26649"
authors:
  - Dongxin Guo
  - Jikun Wu
  - Siu Ming Yiu
first_author: Dongxin Guo
year: 2026
tags: [2026]
status: complete
importance: medium
---

# When to Retrieve During Reasoning: Adaptive Retrieval for Large Reasoning Models

Surfaced via the alphaXiv / NLP-newsletter digest.

## Summary

Large reasoning models (DeepSeek-R1, OpenAI o1) generate extended chains of thought spanning thousands of tokens, but their integration with retrieval-augmented generation is misaligned: standard RAG optimizes for providing context *before* reasoning begins, while reasoning models need evidence injected *during* multi-step inference. **ReaLM-Retrieve** is a reasoning-aware retrieval framework that injects evidence at the right moment within the reasoning chain rather than only at the start.

## Key points

- **Step-level uncertainty detector** identifies knowledge gaps at reasoning-step granularity (not token or sentence level).
- **Retrieval intervention policy** learns *when* external evidence maximally benefits ongoing reasoning.
- **Efficiency-optimized integration** cuts per-retrieval overhead by 3.2× vs. naive integration.
- On MuSiQue, HotpotQA, and 2WikiMultiHopQA: average +10.1% absolute answer-F1 over standard RAG (range 9.0–11.8%).
- Reduces retrieval calls by 47% vs. fixed-interval retrieval.
- Core thesis: *when* to retrieve during reasoning matters as much as *what* to retrieve.

## Concepts & entities

[[retrieval-augmented-generation]] · [[chain-of-thought]] · [[latent-reasoning]] · [[llm-agent]]

## References

- [arXiv:2604.26649](https://arxiv.org/abs/2604.26649)
