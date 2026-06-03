---
title: "Do Language Models Need Sleep? Offline Recurrence for Improved Online Inference"
type: source
created: 2026-06-03
updated: 2026-06-03
sources:
  - do-language-models-need-sleep-lee-2026.pdf
arxiv_id: "2605.26099"
authors:
  - Sangyun Lee
  - Sean McLeish
  - Tom Goldstein
  - Giulia Fanti
first_author: Sangyun Lee
year: 2026
tags: [2026]
status: complete
importance: medium
---

# Do Language Models Need Sleep? Offline Recurrence for Improved Online Inference

*Surfaced via the alphaXiv / NLP-newsletter digest.*

## Summary

This CMU/University of Maryland paper proposes a sleep-like **memory consolidation** mechanism for hybrid SSM-attention models that attacks the gap between scalable memory and scalable reasoning. Transformer attention scales poorly with context, and while fast-weight memories in efficient sequence models support long-range recall, the authors find vanilla SSM-attention hybrids degrade as required reasoning depth grows. Their fix: periodically the model "sleeps," performing N offline recurrent passes over accumulated context and updating the fast weights in its state-space-model blocks via a learned local rule, then clears the KV cache. This shifts extra computation into the offline sleep phase while preserving wake-time inference latency.

## Key points

- Sleep phase: N offline recurrent passes over accumulated context update SSM-block fast weights through a learned local rule, then the context window is cleared.
- Moves heavy computation off the latency-critical "wake" path; online prediction latency is preserved.
- Distinguishes scalable memory (recall) from scalable reasoning (deep computation over absent tokens).
- Vanilla SSM-attention hybrids and regular transformers fail on the harder reasoning tasks under matched token budgets.
- Tested on synthetic cellular automata, multi-hop graph retrieval, and a realistic math-reasoning task.
- Increasing sleep duration N improves performance, with the largest gains on examples requiring deeper reasoning.

## Concepts & entities

- [[sleep-time-compute]]
- [[memory-management]]
- [[offline-consolidation]]
- [[transformer-architecture]]

## References

- [arXiv:2605.26099](https://arxiv.org/abs/2605.26099)
