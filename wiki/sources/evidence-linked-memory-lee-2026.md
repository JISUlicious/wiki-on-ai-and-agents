---
title: "Don't Just Trust Your Memory: Evidence-Linked Memory Reduces Memory-Induced Degradation in LLM Agents"
type: source
created: 2026-11-17
updated: 2026-11-17
sources:
  - evidence-linked-memory-lee-2026.md
arxiv_id: "2611.14320"
venue: arXiv preprint (cs.AI)
authors:
  - Yukyeong Lee
  - Yumin Choi
  - Jinheon Baek
  - Sung Ju Hwang
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# Don't Just Trust Your Memory: Evidence-Linked Memory Reduces Memory-Induced Degradation in LLM Agents

**Source:** Yukyeong Lee, Yumin Choi, Jinheon Baek, Sung Ju Hwang (KAIST; DeepAuto.ai) — [arXiv:2611.14320](https://arxiv.org/abs/2611.14320), cs.AI, v1 10 Nov 2026. Code: github.com/yukyeongleee/wikitrace.

## Summary

Experience memory distills past trajectories into short lessons, but a lesson can also break tasks the agent would have solved without it. The authors call this **memory-induced degradation**. They measure it by splitting accuracy into **Gain** (memory turns a failure into a success) and **Regression** (memory turns a success into a failure). Average accuracy hides the second.

The cause is that each trajectory carries only a success/failure label, so any lesson drawn from it is a guess about which actions mattered. **WikiTrace** keeps the evidence attached. A compiler turns each past run into a *compilation* (why the run succeeded or failed) and up to three *insights*, and every insight links down to its compilation and to the raw trajectory. At test time the agent sees all insights. It is told to "descend" through the links with terminal tools (read, bash) when an insight lacks needed detail, conflicts with another, or comes from a failed run, and again after any failed action. Memory is built once, offline, and is read-only.

## Key points

- **Diagnosis.** On tasks the agent solves in all three no-memory runs but fails with memory, 52% of failures trace to a memory item the agent followed. The most common cause is a wrong item (20%); the others are out-of-scope use, misuse, or missing a better item.
- **Main results** (deepseek-v4-flash, Pi terminal agent, 8 suites across [[alfworld|ALFWorld]], ScienceWorld, PlanCraft, [[tau-bench|τ³-bench]] Retail and [[appworld|AppWorld]]). Average 90.2 vs 79.5 for no memory (+10.7), and +3.2 over **Insights Only**, which has the same insights without source links. Best on all eight suites, and the only memory method that never falls below no memory.
- Baselines: AWM 81.6, ReasoningBank 81.7, [[expel-zhao-2023|ExpeL]] 80.3.
- **Regression** falls to 4.5%, vs 6.5% for Insights Only and 9.7% for simply rerunning without memory. Gain is 35 points above the rerun baseline, against 27 for Insights Only and 7–14 for prior methods.
- **When it descends.** In 22% of runs on easy tasks and 44% on hard ones. The largest edge (+17.3 points over Insights Only) comes when a blocked attempt sends the agent to look up a specific fact in past runs. The opened run changed the plan in 43% of descents.
- **Failed runs** add useful guidance but raise Regression when given as bare insights (9.3% on PlanCraft); source links lower it to 7.1%.

## Concepts & entities

- [[procedural-memory]] · [[experiential-learning]] · [[reflexion]] · [[react]] — lessons distilled from trajectories.
- [[long-term-memory]] · [[llm-wiki]] — layered, linked memory documents.
- [[alfworld]] · [[appworld]] · [[tau-bench]] · [[agent-evaluation]].
- [[wikiskill-compiling-agent-experience-tang-2026]] · [[from-memory-to-skills-tang-2026]] · [[bad-memory-prompt-injection-agentic-gadgil-2026]] — related 2026 work on compiled experience and memory as a failure source.

## References

- [arXiv:2611.14320](https://arxiv.org/abs/2611.14320)
