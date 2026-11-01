---
title: "Sage-A1.5: From Long-Horizon Execution to Scientific Exploration with Capability Composition"
type: source
created: 2026-10-29
updated: 2026-10-29
sources:
  - sage-a1-5-capability-composition-shanghai-ai-lab-2026.md
arxiv_id: "2610.28578"
authors:
  - Sage-A1.5 Team (Shanghai Artificial Intelligence Laboratory)
year: 2026
status: complete
importance: low
tags:
  - 2026
  - paper
---

# Sage-A1.5: From Long-Horizon Execution to Scientific Exploration with Capability Composition

**Sage-A1.5 Team, Shanghai Artificial Intelligence Laboratory.** [arXiv:2610.28578](https://arxiv.org/abs/2610.28578), v1, cs.AI, 20 Oct 2026. Technical report.

## Summary

Sage-A1.5 is a **35B-total / 3B-active MoE agent model** built on Agents-A1. Agents-A1 was the lab's earlier 35B MoE "scale the horizon, not the parameters" model, trained on trajectories averaging 45K tokens. The report's idea is a shift from **capability coverage** (more domains) to **capability composition** (how capabilities are organized during execution). The model is trained for two such capabilities:
- **Agentic orchestration:** deciding when to delegate sub-tasks to parallel subagents with isolated contexts, and merging what they find.
- **Agentic verification:** recognizing stalls, re-checking evidence and revising strategy. This happens inside the model's own reasoning and tool use, with no external supervisor.

Training runs in three stages. First, domain teachers (science, search, SWE, MLE, long-context, real-world workflows) are built with SFT and RL. They are then merged through "unified outcome–process post-training" into one model, which is also packaged for local quantized deployment.

## Key points

- **Headline claim.** Despite its size, Sage-A1.5 is said to **beat Kimi-K3, GLM-5.3 and GPT-5.6-sol on FrontierScience-Olympiad, CMT, PinchBench and IFBench**. It is called competitive on SWE-Bench Pro and BrowseComp, and it matches or exceeds Agents-A1 and Ornith-1.5 on all 12 benchmarks.
- **Teacher-stage gains over the Qwen3.5-35B-A3B base:**
  - Science teacher: FS-Olympiad 64.5 → 89.0, FS-Research 2.5 → 70.0, CMT 20.0 → 84.0, CritPt 0.9 → 25.7, HLE w/ tools 47.4 → 56.8.
  - Search teacher: GAIA 59.8 → 96.1, BrowseComp 61.0 → 85.5.
  - MLE teacher: MLE-Bench Lite medal rate 72.7% vs 37.9% for its base model (gold 50%).
- **Case studies** cover local deployment, multidisciplinary scientific exploration (chemistry, materials, PXRD) and scientist-aligned problem solving. All run in simulation or on historical data.

## What it does not show

- **Teacher scores are not the final model's scores.** The limitations section says the unified model "does not yet fully retain the peak performance of every domain teacher". The large jumps above belong to the per-domain teachers.
- **Mixed provenance for baselines.** Baseline numbers are taken from each model's own technical report where available, and re-run with Sage's protocol otherwise.
- **The base model is stated inconsistently**: Qwen3.5-35B-A3B in the science and search sections, Qwen3.6-35B-A3B in the MLE section.
- **Not a multi-agent system in the Agensh/GitSwarm sense.** Orchestration is learned *within one model's policy*, and the experiments isolate neither orchestration nor verification.
- Orchestration and verification training centres on search and research synthesis. The scientific showcases involve no physical experiments.

## Concepts & entities

- [[learned-orchestration]] · [[subagents]] · [[mixture-of-experts]]
- [[agentic-rl]] · [[knowledge-distillation]]: teacher merging.
- [[mle-bench]] · [[gaia]] · [[swe-bench]]
- [[kimi-k3-moonshot-2026]] · [[glm-5-2]] · [[qwen]]
- [[ncp-archpreview-next-concept-prediction-shanghai-ai-lab-2026]]: another Shanghai AI Lab report.

## References

- [arXiv:2610.28578](https://arxiv.org/abs/2610.28578)
