---
title: "From Confident Closing to Silent Failure: Characterizing False Success in LLM Agents"
type: source
created: 2026-11-02
updated: 2026-11-02
sources:
  - false-success-llm-agents-advani-2026.pdf
arxiv_id: "2606.09863"
authors:
  - Laksh Advani
year: 2026
introduces:
  - [[false-success]]
status: complete
importance: high
tags:
  - 2026
  - paper
---

# From Confident Closing to Silent Failure: Characterizing False Success in LLM Agents

Laksh Advani. [arXiv:2606.09863](https://arxiv.org/abs/2606.09863), June 2026.

## Summary

The paper studies [[false-success|false success]]: an agent claims completion while the environment state shows failure. It uses 9,876 tau2-bench trajectories and 1,879 AppWorld trajectories with ground truth independent of the agent's text. It concludes that LLM judges are poor primary monitors for this failure.

## Key points

- **Prevalence.** 45–48% of failures in single-control tau2-bench domains, 3% in dual-control telecom, and 75.8% among AppWorld self-assessing coding-agent trajectories with explicit status claims.
- **LLM judges.** Across 5 judges and 5 prompt strategies, none exceeds AUROC 0.65 on tau2-bench, and they reach only 0.54 on AppWorld API-call traces. They respond to confident closing language and action volume rather than verified state changes.
- **Cheap detectors.** TF-IDF detectors reach task-disjoint AUROC 0.83 and 0.95, recovering 4–8× more false successes than the best judge at the same flag rate.
- **Reasoning doesn't reliably help.** Qwen3-Max-Thinking has the highest tau2-bench rate, 79%.

## What it does not show

- Judges score recorded trajectories after the run; they do not act as in-loop goal gates.
- Both benchmarks are structured API environments, and the paper leaves open-ended code generation as an open question.
- The dual-control effect rests on one domain, with 15 false-success cases.
- Detectors are calibrated to these domains, and the author recommends them for triage.

## Concepts & entities

- [[false-success]] · [[verifier]] · [[agent-evaluation]] · [[appworld]]
- [[harness-extension-and-goal-loop-axes]]

## References

- [arXiv:2606.09863](https://arxiv.org/abs/2606.09863)
