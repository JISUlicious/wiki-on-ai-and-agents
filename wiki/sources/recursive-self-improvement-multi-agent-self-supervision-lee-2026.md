---
title: "Recursive Self-Improvement through Multi-Agent Self-Supervision (MASS)"
type: source
created: 2026-10-29
updated: 2026-10-29
sources:
  - recursive-self-improvement-multi-agent-self-supervision-lee-2026.md
arxiv_id: "2610.12176"
authors:
  - Hyunin Lee
  - Jinglue Xu
  - Somayeh Sojoudi
  - Jeffrey Seely
  - Donghyun Lee
  - Matei Zaharia
  - Yujin Tang
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# Recursive Self-Improvement through Multi-Agent Self-Supervision (MASS)

**Hyunin Lee, Jinglue Xu, Somayeh Sojoudi, Jeffrey Seely, Donghyun Lee, Matei Zaharia, Yujin Tang** ([[uc-berkeley|UC Berkeley]] · [[sakana-ai|Sakana AI]]; first author interned at Sakana) — [arXiv:2610.12176](https://arxiv.org/abs/2610.12176), v1, cs.AI, 8 Oct 2026. Preprint.

## Summary

MASS targets recursive self-improvement on **non-verifiable** research tasks, where the model being improved is also its own best available supervisor. The authors argue that one model critiquing itself shares the same prior and reinforces its own errors. MASS breaks this "homogeneous loop" by having copies of the model act as a team. **Inner loop:** the model proposes a multi-agent workflow (roles, instructions, *contracts* for what each subagent returns, *hops* for call order), runs it, self-evaluates pairwise, and the same model revises the workflow through evolutionary search with structural guardrails. **Outer loop:** trajectories from the optimized workflows (orchestrator and subagent windows together) are used for LoRA SFT of the shared model. The updated model then takes on all three roles (agent, optimizer, evaluator) again. Two cycles were run: L(0) → L(1) → L(2).

## Key points

- **Setup.** Base model Qwen3.6-27B in the qwen-code scaffold. Training uses 8 synthetic research tasks (finance, robotics, pharmacy), and testing uses 3 synthetic tasks plus 6 public benchmarks. Cycle-1 SFT used 119 teacher trajectories, giving 2,435 windows and 17.2M supervised tokens (LoRA rank 64). Final scores come from **external judges (GPT-5.5 and Claude Opus 4.8, 3 seeds each)** that never enter the loop.
- **Synthetic test tasks:** win rate vs base is 53.9% for L(1) and **69.9% for L(2)**. L(2) beats L(1) at 60.8%.
- **Public research benchmarks (L0 → L2):** ScienceAgentBench 28.8 → 31.7, MLR-Bench 1.58 → 2.40, AstaBench 0.053 → 0.067, DSBench 0.473 → 0.482 (inside the SD). On these four, **score per output token rises 1.2–1.6×**.
- **Self-evaluator agreement** with the external judges rises 0.73 → 0.91 → 0.93, although training used only task-solving traces. Workflows written by L(1) beat L(0)'s workflows 64% of the time even when L(0) executes them.
- **Information flow.** Over optimization, roles and instructions become *less* task-specific, while contracts and hops become *more* task-specific (mutual information with task embeddings). Redundancy between components falls.
- **Training efficiency.** A multi-agent student wins 68.3% against the base model. A single-agent student trained on ~1.4× more supervised tokens wins 64.0%.
- **Bottleneck.** Ablations point to the **workflow optimizer**, not the evaluator, as the binding constraint in the homogeneous loop.

> [!warning] Newsletter vs paper
> The blurb's "1.2–1.6× performance per output token" is correct, but it covers only the four research benchmarks. On the two coding benchmarks the paper reports in the same figure, MASS goes **backwards**: Terminal-Bench 2.0 37.5 → 35.2 and SWE-bench Verified 62.6 → 61.1, with per-token ratios of 0.94× and 0.96×. "Sakana" is half the affiliation; the senior authors include UC Berkeley's Zaharia and Sojoudi.

## What it does not show

- **No non-recursive baseline at matched compute.** For example, SFT on workflows found by a stronger fixed optimizer, or simply more cycles of single-agent self-distillation.
- **Absolute gains on public benchmarks are small.** Most of the per-token gain comes from shorter outputs, not higher scores (DSBench and AstaBench sit near the SD).
- **Checkpoints for the SFT-data ablations were selected using the external judges.** That is the same signal that is later reported.
- **Only two cycles, one 27B model.** "Continuous recursive bootstrapping" is extrapolated, not shown.

## Concepts & entities

- [[self-improving-agent]] · [[multi-agent]] — RSI where the improvement signal comes from orchestrating copies of the model.
- [[llm-as-a-judge]] — the in-loop self-evaluator and the external GPT-5.5/[[claude-opus-4-8|Opus 4.8]] judges.
- [[supervised-fine-tuning]] · [[lora]] · [[knowledge-distillation]] — distilling multi-agent traces back into shared weights.
- [[gepa]] · [[adas]] · [[prompt-optimization]] — the workflow-search lineage (ADAS, AFlow and GPTSwarm are cited).
- [[terminal-bench]] · [[swe-bench-verified]] — the two benchmarks where it regresses.
- Siblings: [[terminal-rsi-recursive-self-improving-terminal-agents-yang-2026]] (verifiable-reward RSI), [[recursive-multi-agent-systems-yang-2026]], [[meta-n-recursive-self-improvement-emergent-depth-kim-2026]], [[self-improvement-via-fast-tree-search-fu-2026]] (also Sakana).
- [[recursive-self-improvement]]: concept page for RSI definitions, autonomy levels and recurring caveats across papers.

## References

- [arXiv:2610.12176](https://arxiv.org/abs/2610.12176)
