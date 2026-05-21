---
title: RASP-L
type: concept
created: 2026-05-21
updated: 2026-05-21
sources:
  - looped-transformers-length-generalization-fan-2024.md
status: draft
importance: medium
tags: []
---

# RASP-L

RASP-L is the subset of operations expressible in the RASP programming language under **causal masking** and **length-restricted attention**. It functions as the canonical complexity class for characterizing "what transformers can learn algorithmically" — programs in RASP-L correspond, by construction, to functions a causal transformer can represent and (empirically) tends to learn from data.

- RASP itself models a transformer as a small set of primitives (select, aggregate, elementwise); RASP-L is the causal, finite-attention restriction.
- Heavily used as an analysis tool for **length generalization**: tasks expressible in RASP-L tend to length-generalize; tasks outside it tend not to.
- Central to the argument in [[looped-transformers-length-generalization-fan-2024]] that looping a transformer extends the expressible class.
- Provides a formal handle for designing tasks that probe what depth / recurrence buys over a fixed-depth feed-forward transformer.

## Related

- [[looped-transformers-length-generalization-fan-2024]] — primary source citing RASP-L
- [[looped-transformer]]
- [[length-generalization]]

## References

- [[looped-transformers-length-generalization-fan-2024]]
- [[looped-transformer]]
