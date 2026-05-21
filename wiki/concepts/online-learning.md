---
title: Online Learning
type: concept
created: 2026-05-21
updated: 2026-05-21
sources: []
status: draft
importance: medium
tags: []
---

# Online Learning

Online learning is a classical ML setting where the model's parameters (or policy) are updated incrementally as new data arrives, rather than trained offline on a fixed dataset. Each example is typically observed once, used to update the model, and then either discarded or stored in a buffer. The framing contrasts with batch / offline training and is foundational for streaming data, non-stationary environments, and continually-learning agents.

- Updates happen per-example or per-small-batch, often under regret-minimization guarantees.
- Closely related to [[reinforcement-learning]] when feedback is reward rather than label.
- A core ingredient for any [[self-improving-agent]] that wants to incorporate experience without full retraining.
- Distinct from in-context learning: online learning changes weights, in-context learning does not.

## Related

- [[reinforcement-learning]]
- [[self-improving-agent]]
- [[memory-evolution]]

## References

- [[reinforcement-learning]]
- [[self-improving-agent]]
