---
title: "LightThinker++: From Reasoning Compression to Memory Management"
type: source
created: 2026-06-03
updated: 2026-06-03
sources:
  - lightthinker-plus-plus-zhu-2026.md
arxiv_id: "2604.03679"
authors:
  - Yuqi Zhu et al.
first_author: Yuqi Zhu
year: 2026
tags:
  - 2026
status: complete
importance: medium
---

# LightThinker++: From Reasoning Compression to Memory Management

Surfaced via the alphaXiv / NLP-newsletter digest.

## Summary

LightThinker++ (Zhejiang University, Ant Group) tackles the surging cognitive overhead of long chain-of-thought traces, where quadratic attention and linear KV-cache growth make extended reasoning expensive. The base method, **LightThinker**, teaches an LLM to dynamically compress intermediate thoughts into compact semantic representations on the fly. But static compression irreversibly discards detail, creating logical bottlenecks on hard problems.

The evolution to **LightThinker++** reframes the problem from reasoning *compression* to **memory management**: it introduces Explicit Adaptive Memory Management, shifting to behavioral-level control via explicit memory primitives, trained through a specialized trajectory-synthesis pipeline that teaches purposeful memory scheduling. Results across three regimes: (1) base LightThinker cuts peak tokens 70% and inference time 26% with minimal accuracy loss; (2) on standard reasoning, LightThinker++ cuts peak tokens 69.9% while *gaining* +2.42% accuracy at fixed budget; (3) on long-horizon agentic tasks it holds a stable footprint past 80 rounds (60-70% reduction) and averages +14.8% performance across complex scenarios. The thesis: managed memory, not just compression, sustains deep reasoning over long horizons.

## Key points

- Frames the next step beyond reasoning compression as explicit **memory management** with memory primitives an LLM learns to schedule.
- LightThinker: dynamic on-the-fly compression of intermediate thoughts into compact semantic tokens.
- Diagnoses static compression's weakness: irreversible loss of intermediate detail causes logical bottlenecks on complex reasoning.
- A trajectory-synthesis pipeline generates training data for purposeful, behavioral-level memory scheduling.
- Efficiency: base method −70% peak tokens / −26% inference time; ++ cuts peak tokens 69.9% while adding +2.42% accuracy.
- Long-horizon agentic: stable footprint beyond 80 rounds (60-70% reduction), +14.8% average performance gain.
- Code: github.com/zjunlp/LightThinker.

## Concepts & entities

[[memory-management]] · [[chain-of-thought]] · [[inference-time-compute]] · [[kv-cache]] · [[latent-reasoning]] · [[long-context-llm]] · [[llm-agent]] · [[zhejiang-university]]

## References

- arXiv: [arXiv:2604.03679](https://arxiv.org/abs/2604.03679)
- Code: https://github.com/zjunlp/LightThinker
