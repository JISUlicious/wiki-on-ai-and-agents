---
title: "Who Verifies the Agent's Success Claim? Self-Report Leakage in GUI-Agent Completion Judging"
type: source
created: 2026-11-02
updated: 2026-11-02
sources:
  - self-report-leakage-gui-agent-judging-zhu-2026.pdf
arxiv_id: "2610.29609"
authors:
  - Yunfeng Zhu
  - Shengcheng Yu
  - Zihan Li
  - Zhenyu Chen
year: 2026
status: complete
importance: high
tags:
  - 2026
  - paper
---

# Who Verifies the Agent's Success Claim? Self-Report Leakage in GUI-Agent Completion Judging

Zhu et al. [arXiv:2610.29609](https://arxiv.org/abs/2610.29609), October 2026.

## Summary

A controlled aware/blind test gives the same judge the same task and trace, and only includes or withholds the agent's self-report. Seeing the report changes the judgment of the trace, which the authors call self-report leakage.

## Key points

- **Effect size.** Withholding the report raises recall of dangerous misreports (unhedged success claims the trace doesn't support) by 43.4 points with Qwen3.5-35B-A3B and 13.3 with DeepSeek-284B-A13B. Held-out Batch 2 shows the same direction, though the smaller DeepSeek gain there has p = 0.065.
- **Fix.** A structured auditor separates claim interpretation from claim-blind trace analysis, then applies deterministic rules.
- **Result.** F1 of 0.75–0.78 on 465 records and 0.77–0.81 on Batch 2. It blocks 74–90% of dangerous misreports at 45–56% inspection load.

## What it does not show

It covers GUI agents and passive, post-hoc traces. The auditor doesn't re-execute tasks or query the environment, and the study doesn't test coding-agent goal commands.

## Concepts & entities

- [[false-success]] · [[verifier]] · [[gui-agent]] · [[harness-extension-and-goal-loop-axes]]

## References

- [arXiv:2610.29609](https://arxiv.org/abs/2610.29609)
