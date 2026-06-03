---
title: "Attention Residuals (Technical Report)"
type: source
created: 2026-06-03
updated: 2026-06-03
sources:
  - attention-residuals-kimi-team-2026.pdf
arxiv_id: "2603.15031"
authors:
  - Kimi Team (Moonshot AI)
first_author: Kimi Team
year: 2026
tags: [2026]
status: complete
importance: medium
---

# Attention Residuals (Technical Report)

Surfaced via the alphaXiv / NLP-newsletter digest.

## Summary

Residual connections with PreNorm are standard in modern LLMs but accumulate every layer's output with fixed unit weights. This uniform aggregation causes uncontrolled hidden-state growth with depth and progressively dilutes each layer's contribution. The Kimi Team (Moonshot AI) proposes Attention Residuals (AttnRes), which replaces fixed accumulation with softmax attention over preceding layer outputs, letting each layer selectively aggregate earlier representations with learned, input-dependent weights.

To control the memory and communication cost of attending over all prior layers in large-scale training, they introduce Block AttnRes, which partitions layers into blocks and attends over block-level representations, preserving most of the gains. Combined with cache-based pipeline communication and a two-phase computation strategy, Block AttnRes becomes a near-drop-in replacement for standard residuals with minimal overhead. Scaling-law experiments show consistent improvements across model sizes; integrated into the Kimi Linear architecture (48B total / 3B activated) and pre-trained on 1.4T tokens, AttnRes mitigates PreNorm dilution, yields more uniform output magnitudes and gradients across depth, and improves downstream performance on all evaluated tasks.

## Key points

- Diagnoses PreNorm residual dilution: fixed unit-weight accumulation grows the hidden state and dilutes per-layer contributions with depth.
- AttnRes replaces fixed accumulation with softmax attention over preceding layer outputs (learned, input-dependent depth-wise weights).
- Block AttnRes attends over block-level (not per-layer) representations to cut memory/communication overhead while keeping most gains.
- Cache-based pipeline communication + two-phase computation make it a practical drop-in for standard residuals.
- Scaling-law experiments confirm consistent gains across sizes; ablations validate content-dependent depth selection.
- Integrated into Kimi Linear (48B/3B activated, 1.4T tokens); produces more uniform magnitudes/gradients and improves downstream tasks.

## Concepts & entities

- [[residual-connection]]
- [[attention-mechanism]]
- [[layer-normalization]]
- [[transformer-architecture]]
- [[scaling-laws]]
- [[pre-training]]
- [[mixture-of-experts]]

## References

- [arXiv:2603.15031](https://arxiv.org/abs/2603.15031)
- Code: https://github.com/MoonshotAI/Attention-Residuals
