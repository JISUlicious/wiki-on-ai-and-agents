---
title: "Agent-as-a-Judge: Evaluate Agents with Agents"
type: source
created: 2026-07-07
updated: 2026-07-07
sources: []
arxiv_id: "2410.10934"
year: 2024
authors:
  - Mingchen Zhuge
  - Changsheng Zhao
  - Yuandong Tian
  - Vikas Chandra
  - Jurgen Schmidhuber
introduces:
  - [[agent-as-a-judge]]
tags:
  - 2024
  - paper
status: complete
importance: high
---

# Agent-as-a-Judge: Evaluate Agents with Agents

**Zhuge et al.** (Meta AI + KAUST) — [arXiv:2410.10934](https://arxiv.org/abs/2410.10934), 2024. **Preprint** — the arXiv comments field lists only project links, no venue.

## The thesis

Existing evaluation *"either focus exclusively on final outcomes—ignoring the step-by-step nature of agentic systems, or require excessive manual labour."* The proposal: use **agentic systems to evaluate agentic systems**, extending LLM-as-a-Judge with the ability to inspect the whole trajectory and issue **intermediate, requirement-level feedback**.

The paper's framing sentence:

> *"Agentic systems should be evaluated like a human, with rich evaluative feedback which looks at the full thought and action trajectory; evaluating an agentic system in the traditional way is like evaluating a student using multiple-choice testing."*

Crucially the judge is positioned as a **reward model, not a scoreboard** — dense per-step signals intended for process supervision and a self-improvement flywheel.

## DevAI benchmark

**55 realistic AI-development tasks**, each with a plain-text query, **365 hierarchical requirements** arranged as a **dependency DAG**, and **125 preferences** (soft criteria judged separately). Manually annotated by experts who drafted queries, established criteria, built the dependency graphs, then cross-analyzed.

The dependency structure earns its keep. Measuring the three solver systems against human consensus:

| | MetaGPT | GPT-Pilot | OpenHands |
|---|---|---|---|
| Requirements met (**independent**) | 22.13% | 44.80% | 42.89% |
| Requirements met (**dependent**) | **6.55%** | **28.96%** | **28.68%** |
| Full task solve rate | 0.00% | 1.81% | 1.81% |

Enforcing dependencies cuts measured performance by a third to two thirds — a direct demonstration of how much credit a **flat requirement checklist over-awards**. And at a ~0-1.8% full-solve rate, binary end-to-end success carries almost no signal.

## The headline comparison

Alignment with human consensus:

| Setting | Judge | MetaGPT | GPT-Pilot | OpenHands |
|---|---|---|---|---|
| **Black-box** (artifacts only) | LLM-as-a-Judge | 84.15% | 65.30% | 60.38% |
| **Black-box** | **Agent-as-a-Judge** | **88.52%** | **83.88%** | **90.44%** |
| **Gray-box** (+ trajectory) | LLM-as-a-Judge | 68.86% | 71.85% | 70.76% |
| **Gray-box** | **Agent-as-a-Judge** | **92.07%** | **86.61%** | **90.16%** |

The sharpest evidence for the thesis is the **asymmetry**: giving the trajectory to a plain LLM judge *hurt* it on MetaGPT (84.15% -> 68.86%) — it drowns in context — while the agentic judge *improved* (88.52% -> 92.07%). More evidence only helps a judge equipped to navigate it.

Against humans: individual expert evaluators align at **76.23-92.63%**; three-human majority vote reaches **93.98-95.08%**. Agent-as-a-Judge (83.88-92.07%) therefore **sits inside the human range and beats the weakest human on every system**, without reaching the majority-vote ceiling.

## Cost claim

*"Agent-as-a-Judge cost only 30.58 USD in API calls and took only 118.43 minutes—2.29% of the cost and 2.36% of the time of Human-as-a-Judge"*, i.e. ~97.7% savings against 115 human-hours costed at $15/hour.

> [!warning] Read the cost claim carefully
> The **$1,297.50 human baseline is imputed labour at minimum wage for PhD-level annotators**, not an actual expenditure — which flatters the ratio considerably. The time percentages also mix denominators: 2.36% uses the 86.5 h initial-evaluation figure while the 97.72% saving appears computed against the 115 h total. The numbers are the paper's; the framing is not conservative.

## Architecture and what didn't work

Eight modules: **graph, locate, read** (33 artifact formats), **search, retrieve** (from trajectory), **ask, memory, planning**. The ablation is candid — best performance used only **graph, locate, read, retrieve, ask**. **Memory caused error propagation**, **planning was unstable**, and **search was underutilized** on DevAI's small codebases.

The paper also calibrates against human unreliability rather than assuming an oracle: inter-evaluator disagreement ran **~10-30%**, with one evaluator at 23.77% error on GPT-Pilot dropping to 6.01% under majority vote. The "ground truth" is a consensus construct.

## Concepts

- [[agent-as-a-judge]] — the concept this introduces.
- [[agent-evaluation]] · [[verifier]] · [[llm-as-a-judge]]

## References

- [arXiv:2410.10934](https://arxiv.org/abs/2410.10934)
