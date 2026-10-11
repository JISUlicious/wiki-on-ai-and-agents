---
title: Test-Time Compute
type: concept
created: 2026-06-19
updated: 2026-10-10
sources:
  - sleep-time-compute-lin-2025.md
status: draft
importance: high
tags:
  - 2026
---

# Test-Time Compute

Test-time compute is the strategy of spending more inference-time computation — through longer reasoning, search over candidates, or drawing multiple samples — to raise accuracy on hard problems. Rather than relying solely on a larger model, the system trades extra compute at answer time for better results. It builds on [[chain-of-thought]] and motivates ideas like [[sleep-time-compute]].

## References

- [[sleep-time-compute-lin-2025]]

## Related 2026 sources

- [[ttpo-test-time-policy-optimization-wang-2026]] — transductive test-time *training*: LoRA adapters fitted over the unlabeled test set with **consensus as the reward** — no verifier, no execution. The paper never compares against self-consistency at matched compute, and its own Figure 6 shows the trained model rising to meet what untrained 12-sample voting already achieved
- [[thinking-with-looped-flows-suleymanzade-2026]] — Looped flows: train looped reasoners with flow-matching losses; ARC-AGI-1 58.8 pass@2 at 5–7M parameters
- [[using-claude-code-spending-your-effort-shihipar-2026]] — Shihipar: on Terminal-Bench 3.0, more effort buys verification and edge-case testing, not better approaches

## Lineage sources

- [[dual-process-theory]] — System 2 as extra inference compute
- [[dualformer-su-2024]] — one model with fast, slow and auto modes
- [[system-1x-saha-2024]] — a hybridization dial between fast and slow planning
