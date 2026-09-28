---
title: LiveCodeBench
type: concept
created: 2026-09-29
updated: 2026-09-29
sources:
  - deepseek-r1-2025.md
  - swe-bench-live-zhang-2025.md
  - embarrassingly-simple-self-distillation-zhang-2026.md
  - vibethinker-3b-verifiable-reasoning-xu-2026.md
  - llms-know-when-they-know-cao-2026.md
  - quantization-hurts-reasoning-liu-2025.md
  - how-transparent-is-diffusiongemma-engels-2026.md
status: draft
importance: medium
---

# LiveCodeBench

**LiveCodeBench** (Jain et al., 2024) is a code-generation benchmark that keeps adding new problems from LeetCode, AtCoder and CodeForces contests. Because each problem carries a release date, a model can be scored only on problems published after its training cutoff. That is the benchmark's answer to [[benchmark-contamination]]. It also covers self-repair, code execution and test-output prediction. Results are usually reported per release, e.g. "v6".

## In this wiki

- **Reasoning-model headline metric.** [[deepseek-r1-2025|DeepSeek-R1]] reports 65.9%. Later model and distillation papers use **LiveCodeBench v6** as their standard coding number. Examples: [[embarrassingly-simple-self-distillation-zhang-2026|Simple Self-Distillation]] (Qwen3-30B-Instruct 42.4 → 55.3 pass@1), [[vibethinker-3b-verifiable-reasoning-xu-2026|VibeThinker-3B]] (80.2), [[llms-know-when-they-know-cao-2026|Cao et al.]] (388 problems), [[kimi-linear-kimi-team-2025|Kimi Linear]] and [[qwen3.8-flash-next-model-card-2026|Qwen3.8-Flash-Next]] (91.9).
- **Robustness probes.** [[quantization-hurts-reasoning-liu-2025|Liu et al.]] measure the drop in LiveCodeBench after quantization. [[how-transparent-is-diffusiongemma-engels-2026|Engels et al.]] use it in their transparency tests on DiffusionGemma.
- **The template for live benchmarks.** [[swe-bench-live]] is described as being to long-horizon coding agents what LiveCodeBench is to competitive programming. It streams whole-repository issues where LiveCodeBench streams algorithmic problems.

## Related

- [[benchmark-contamination]] · [[code-generation]] · [[humaneval]] · [[swe-bench-live]] · [[swe-bench]]

## References

- Jain et al., "LiveCodeBench: Holistic and Contamination Free Evaluation of Large Language Models for Code" (2024), [arXiv:2403.07974](https://arxiv.org/abs/2403.07974)
- The wiki sources listed in frontmatter.
