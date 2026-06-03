---
title: "HyperAgents"
type: source
created: 2026-06-03
updated: 2026-06-03
sources:
  - hyperagents-zhang-2026.md
arxiv_id: "2603.19461"
authors:
  - Jenny Zhang
  - Bingchen Zhao
  - Wannan Yang
  - Jakob Foerster
  - Jeff Clune
  - Minqi Jiang
  - Sam Devlin
  - Tatiana Shavrina
first_author: Jenny Zhang
year: 2026
tags: [2026]
status: complete
importance: medium
---

# HyperAgents

Surfaced via the alphaXiv / NLP-newsletter digest.

## Summary

HyperAgents introduces *self-referential* agents for open-ended recursive self-improvement. Prior self-improving systems rely on a fixed, handcrafted meta-level mechanism that bounds how the base system can be improved; adding a meta-meta layer just shifts the problem upward into infinite regress. A hyperagent instead integrates a **task agent** (solves the target task) and a **meta agent** (modifies itself and the task agent) into a single editable program — and crucially the meta-level modification procedure is itself editable, enabling *metacognitive self-modification* that improves not only task behavior but the very mechanism generating future improvements.

The framework is instantiated by extending the Darwin Gödel Machine (DGM) into **DGM-Hyperagents (DGM-H)**. The original DGM achieved open-ended self-improvement in coding by repeatedly generating and evaluating self-modified variants, but it depended on coding-domain alignment (coding ability ≈ self-modification ability). By letting the improvement procedure evolve, DGM-H removes that domain-specific assumption and can in principle self-accelerate on any computable task. Across coding, paper review, robotics reward design, and Olympiad-level math-solution grading, DGM-H improves over time and beats both non-self-improving baselines and prior systems like DGM; meta-level improvements (e.g., persistent memory, performance tracking) transfer across domains and accumulate across runs. Experiments used sandboxing and human oversight; the paper discusses safety implications.

## Key points

- **Self-referential** design: a single editable program containing both task agent and meta agent, where even the meta-modification procedure is editable (metacognitive self-modification).
- Targets the **infinite-regress** limitation of fixed meta-levels in recursive self-improvement.
- **DGM-H** extends the Darwin Gödel Machine, dropping its assumption that task performance and self-modification skill are aligned (true only in coding).
- Evaluated across four diverse domains: coding, paper review, robotics reward design, math-grading.
- Outperforms no-self-improvement baselines and prior self-improving systems (DGM); meta-level gains (persistent memory, performance tracking) transfer across domains and accumulate across runs.
- Explicit safety framing: sandboxing, human oversight, discussion of what safety entails for self-accelerating systems. Code released by FAIR/Meta.

## Concepts & entities

[[self-improving-agent]] · [[recursive-self-improvement]] · [[darwin-godel-machine]] · [[meta-learning]] · [[llm-agent]] · [[multi-agent]] · [[open-endedness]] · [[code-as-harness]] · [[ai-safety]] · [[fair]]

## References

- arXiv: [arXiv:2603.19461](https://arxiv.org/abs/2603.19461)
- Code: https://github.com/facebookresearch/Hyperagents
- Local source: `sources/hyperagents-zhang-2026.md`
