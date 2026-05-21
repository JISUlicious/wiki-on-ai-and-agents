---
title: Length Generalization
type: concept
created: 2026-05-21
updated: 2026-05-21
sources:
  - looped-transformers-length-generalization-fan-2024.md
status: draft
importance: medium
tags:
  - 2024
---

**Length generalization** is the challenge of solving inputs *longer* than anything seen during training — e.g., adding 20-digit numbers after only training on up-to-10-digit examples. Standard transformers degrade sharply past their training length; weight-tied / looped architectures, paired with algorithmically clean tasks (RASP-L-expressible), generalize much further by performing the same step many times.

- Failure mode of vanilla transformers past max-train-length
- [[looped-transformer]] + [[recurrent-depth-transformer]] are a leading mitigation
- Closely tied to algorithmic-expressibility frameworks like [[rasp-l]]
- Tested on tasks like addition, parity, copy, and reverse

Related: [[looped-transformer]], [[recurrent-depth-transformer]], [[rasp-l]]

## References
- [[looped-transformers-length-generalization-fan-2024]]
