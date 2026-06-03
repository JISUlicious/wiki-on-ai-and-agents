---
title: "OCR-Memory: Optical Context Retrieval for Long-Horizon Agent Memory"
type: source
created: 2026-06-03
updated: 2026-06-03
sources:
  - ocr-memory-optical-context-retrieval-li-2026.pdf
arxiv_id: "2604.26622"
authors:
  - Jinze Li
  - Yang Zhang
  - Xin Yang
  - Jiayi Qu
  - Jinfeng Xu
  - Shuo Yang
  - Junhua Ding
  - Edith Cheuk-Han Ngai
first_author: Jinze Li
year: 2026
tags: [2026]
status: complete
importance: medium
---

# OCR-Memory: Optical Context Retrieval for Long-Horizon Agent Memory

Surfaced via the alphaXiv / NLP-newsletter digest.

## Summary

Autonomous LLM agents in long-horizon, interactive settings succeed by reusing experience accumulated over extended histories, but existing agent memory systems are bottlenecked by text-context budgets: storing or revisiting raw trajectories is token-expensive, while summarization and text-only retrieval trade tokens for information loss. **OCR-Memory (Optical Context Retrieval Memory)** leverages the *visual modality* as a high-density representation of agent experience, retaining arbitrarily long histories with minimal prompt overhead at retrieval time. It renders historical trajectories into images annotated with unique visual identifiers and retrieves via a **locate-and-transcribe** paradigm.

## Key points

- Reframes the agent-memory token-budget problem as a modality problem: images pack far more context per token than text.
- Historical trajectories are rendered to images annotated with unique visual anchors / identifiers.
- Retrieval = locate relevant regions via visual anchors, then transcribe the verbatim text — avoiding free-form generation and reducing hallucination.
- Preserves faithful evidence recovery rather than lossy summaries.
- Consistent gains on long-horizon agent benchmarks under strict context limits.
- Demonstrates optical encoding increases *effective* memory capacity beyond text-context constraints.

## Concepts & entities

[[memory-management]] · [[retrieval-augmented-generation]] · [[llm-agent]] · [[multi-agent-memory]]

## References

- [arXiv:2604.26622](https://arxiv.org/abs/2604.26622)
