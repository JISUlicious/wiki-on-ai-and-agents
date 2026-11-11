---
title: "Work Agents: Systems, Capabilities, and Optimization"
type: source
created: 2026-11-10
updated: 2026-11-10
sources:
  - work-agents-systems-capabilities-and-optimization-xu-2026.md
arxiv_id: "2610.35845"
venue: arXiv preprint (cs.LG); survey
authors:
  - Zishan Xu
  - Enjun Du
  - Mingqi Gao
  - Cheng Yang
  - Junhao Shen
  - Zirong Chen
  - Zhiyuan Yao
  - Yifu Guo
  - Yuquan Lu
  - Jinyang Huang
  - Zhengxi Lu
  - Siyi Liu
  - Ziyu Zheng
  - Yongqi Zhang
  - Difan Zou
  - Weiwen Liu
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - survey
---

# Work Agents: Systems, Capabilities, and Optimization

**Source:** Zishan Xu, Enjun Du, Mingqi Gao et al. ([[shanghai-jiao-tong-university|SJTU]], [[hku|HKU]], HKUST-GZ, [[tsinghua-university|Tsinghua]], [[chinese-university-of-hong-kong|CUHK]], [[zhejiang-university|Zhejiang]], and others) — [arXiv:2610.35845](https://arxiv.org/abs/2610.35845), cs.LG, v1 26 Oct 2026.

## Summary

A large survey that takes a *work-centric* view of agents. A **work agent** is a foundation model (the decision core) plus a runtime harness (context, tools, memory, skills, execution control, recovery), carrying out delegated tasks in a *stateful digital workspace* and producing outcomes that can be *verified*. The unit of analysis moves from single outputs or tool calls to the whole path from request to verified outcome. [[claude-code|Claude Code]] and [[openai-codex|Codex]] are given as examples of multi-day, cross-session work.

## Key points

- **Six elements:** task, model, harness, workspace, work outcome, verification. Outcomes include answers, artifacts, state changes and, for ongoing duties, maintained conditions. Verification can come from programs, humans or model judges.
- **Two working-condition axes.** *Work persistence*: does responsibility close on delivery, or stay active across stages and events? *Work openness*: are objects, interfaces and rules specified up front, or discovered during execution?
- **Five work forms** by scope: conversation, atomic action, multi-step, workflow and workspace level.
- **Five capability families:** perception & understanding; reasoning & decision making; planning & coordination; execution & state management; verification, recovery & replanning.
- **Optimization in three parts.** *Model* (pre-, mid- and post-training on work environments, curated trajectories and reward design). *Harness* (observation and action interfaces, workflows and execution control, long context, memory and [[agent-skills|skills]]). *Joint* (harness-aware policy training, co-adaptation, [[recursive-self-improvement]]).
- **Evaluation should be outcome-based**: artifact quality, environment-state change, workflow completion, constraint satisfaction. Economics is a separate axis: the performance–cost Pareto frontier, a "harness tax" (same model, similar success, very different API cost across harnesses on SWE-bench Lite and Terminal-Bench 2.0) and **agentic ROI** (quality vs a human baseline, time saved, monetary cost, including human specification and checking effort).
- **Frontiers.** Personal work agents, human digital twins for professional work, and organizational work systems. Two coupled loops: a *work execution loop* that carries current responsibilities, and a *capability improvement loop* that learns from them. The open question is how to learn without disturbing commitments already in flight.

## Concepts & entities

- [[llm-agent]] · [[agentic-harness-engineering]] · [[computer-use-agent]] · [[gui-agent]] · [[long-term-memory]] · [[agent-skills]] · [[planning]] · [[verifier]]
- Optimization: [[agentic-rl]] · [[supervised-fine-tuning]] · [[rl-environment-synthesis]] · [[self-improving-agent]] · [[recursive-self-improvement]]
- Evaluation: [[agent-evaluation]] · [[agent-benchmark]] · [[swe-bench]] · [[terminal-bench]] · [[osworld]] · [[llm-as-a-judge]]
- Products: [[claude-code]] · [[openai-codex]]
- Related: [[harness-vs-model-attribution]] (the harness-tax point)

## References

- [arXiv:2610.35845](https://arxiv.org/abs/2610.35845)
