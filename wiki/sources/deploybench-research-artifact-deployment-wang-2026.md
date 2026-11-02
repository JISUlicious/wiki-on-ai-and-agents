---
title: "DeployBench: Benchmarking LLM Agents for Research Artifact Deployment"
type: source
created: 2026-11-02
updated: 2026-11-02
sources:
  - deploybench-research-artifact-deployment-wang-2026.pdf
arxiv_id: "2606.05238"
authors:
  - Yuanli Wang
  - Yaoyao Qian
  - Yue Zhang
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# DeployBench: Benchmarking LLM Agents for Research Artifact Deployment

Wang et al. [arXiv:2606.05238](https://arxiv.org/abs/2606.05238), June 2026. The author list above gives the first three of eleven names.

## Summary

The benchmark has 51 research-artifact deployment tasks, each verified by a hidden pipeline that runs the paper's designated experiment. Five LLMs with OpenHands score 7.8–51.0%.

## Key points

- **Completion judgment dominates failure.** 102 of 181 failures are agent-terminated self-stops, where pre-finish checks validate a different or weaker target than the task requires.
- **Why it matters.** The problem is the agent checking the wrong thing, not only overclaiming. A checker sharing that misunderstanding could fail the same way, which is this wiki's inference.

## What it does not show

It doesn't test a goal command or a separate judge, and it uses a single harness.

## Concepts & entities

- [[false-success]] · [[verifier]] · [[agent-benchmark]] · [[harness-extension-and-goal-loop-axes]]

## References

- [arXiv:2606.05238](https://arxiv.org/abs/2606.05238)
