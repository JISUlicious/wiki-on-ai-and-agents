---
title: "System 2 Attention (is something you might need too)"
type: source
created: 2026-10-10
updated: 2026-10-10
sources:
  - system-2-attention-weston-2023.md
arxiv_id: "2311.11829"
venue: arXiv preprint
authors:
  - Jason Weston
  - Sainbayar Sukhbaatar
first_author: Jason Weston
year: 2023
tags:
  - 2023
  - paper
status: complete
importance: medium
---

# System 2 Attention (S2A)

**Source**: `sources/system-2-attention-weston-2023.md` ([arXiv:2311.11829](https://arxiv.org/abs/2311.11829), 2023-11-20)
**Authors**: Jason Weston and Sainbayar Sukhbaatar ([[meta]] / [[fair]])

## Summary

The paper treats a Transformer's **soft attention** as a System 1 process. It spreads weight over irrelevant context, over-focuses on repeated tokens, and is swayed by opinions in the prompt, which causes [[sycophancy]].

**S2A** adds a deliberate step: the LLM first **regenerates the input context to keep only the relevant parts**, using its ability to reason and follow instructions, and then answers from that rewritten context. On QA with distracting opinions, math word problems with irrelevant sentences, and long-form generation, S2A improves factuality and objectivity and reduces sycophancy.

## Why it matters

- **An early, explicit use of "System 2" for an inference-time LLM technique**: spending extra generated tokens to decide what to attend to. It is a prompt-level ancestor of [[context-engineering]]'s "curate the smallest high-signal context".
- **One of the System 2 methods that [[distilling-system-2-into-system-1-yu-2024]] later distills back into a single System 1 pass.**

## Concepts & entities

[[dual-process-theory]] · [[sycophancy]] · [[context-engineering]] · [[context-rot]] · [[distilling-system-2-into-system-1-yu-2024]]
