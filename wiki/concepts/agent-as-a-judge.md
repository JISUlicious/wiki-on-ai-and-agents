---
title: Agent-as-a-Judge
type: concept
created: 2026-07-07
updated: 2026-07-07
sources:
  - agent-as-a-judge-zhuge-2024.md
builds-on:
  - [[llm-as-a-judge]]
status: complete
importance: medium
---

# Agent-as-a-Judge

**Using agentic systems to evaluate agentic systems** — an extension of [[llm-as-a-judge]] in which the judge is itself an agent that can navigate the produced artifacts and the solver's trajectory, issuing **requirement-level intermediate judgments** rather than one verdict on a final output.

Introduced by [[agent-as-a-judge-zhuge-2024|Zhuge et al. 2024]] (Meta AI + KAUST).

## The argument

Existing evaluation *"either focus[es] exclusively on final outcomes—ignoring the step-by-step nature of agentic systems, or require[s] excessive manual labour."* The paper's framing:

> *"Agentic systems should be evaluated like a human, with rich evaluative feedback which looks at the full thought and action trajectory; evaluating an agentic system in the traditional way is like evaluating a student using multiple-choice testing."*

The empirical case for it is blunt: on the DevAI benchmark, full-task solve rates across three leading agent frameworks were **0.00%, 1.81%, 1.81%**. A binary end-to-end metric cannot distinguish systems that are all failing.

## What makes it different from trajectory matching

This is the distinction that keeps Agent-as-a-Judge compatible with Anthropic's *"grade the outcome, not the path"* rule (see [[agent-evaluation]]):

- **Trajectory matching** asks *did the agent take the expected sequence of tool calls* — which Anthropic and LangChain both find brittle, since agents find valid unanticipated routes.
- **Agent-as-a-Judge** asks *which of the 365 stated requirements are satisfied*, with requirements arranged in a **dependency DAG**. It judges outcomes at fine granularity, not paths.

Enforcing the dependency structure matters: requirement satisfaction drops from 22-45% (flat checklist) to **6.5-29%** (dependencies enforced), showing how much credit a flat checklist over-awards.

## Why an agentic judge beats a plain one

The cleanest result in the paper is an **asymmetry**. Moving from black-box (artifacts only) to gray-box (artifacts + trajectory):

| Judge | MetaGPT |
|---|---|
| LLM-as-a-Judge | 84.15% -> **68.86%** (worse) |
| Agent-as-a-Judge | 88.52% -> **92.07%** (better) |

A single-pass LLM drowns in trajectory context; an agent equipped to navigate it benefits. **More evidence only helps a judge that can search it.**

Against humans, Agent-as-a-Judge (83.88-92.07% alignment) sits **inside the range of individual expert evaluators** (76.23-92.63%) and beats the weakest on every system, without reaching the three-human majority vote (93.98-95.08%).

## As a reward model, not a scoreboard

The stated purpose is process supervision: dense per-requirement signals intended as *"rich and reliable reward signals necessary for dynamic and scalable self-improvement."* This connects it to [[verifier]] — but note the caution there about using verification signal as an RL reward rather than an inference-time gate.

## What didn't work

The ablation is candid. Of eight modules, best performance used only **graph, locate, read, retrieve, ask**: **memory caused error propagation**, **planning was unstable**, and **search was underutilized** on small codebases. The authors disclaim optimality — *"a perfect Agent-as-a-Judge is not the focus of this proof of concept."*

Scope remains narrow: one domain (code generation / AI development), 55 tasks, three solver systems, small codebases, and no venue (preprint).

## Related

- [[llm-as-a-judge]] — the mechanism it extends.
- [[agent-evaluation]] · [[verifier]]

## References

- [[agent-as-a-judge-zhuge-2024]]
