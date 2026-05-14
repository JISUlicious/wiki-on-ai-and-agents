---
title: Activation Addition (ActAdd)
type: concept
created: 2026-05-14
updated: 2026-05-14
sources:
  - turner-activation-engineering-2023.md
status: complete
importance: high
---

# Activation Addition (ActAdd)

ActAdd is the canonical [[steering]] algorithm introduced by [[activation-engineering-turner-2023|Turner et al. (2023)]]. Compute a **[[steering-vectors|steering vector]]** from a single contrastive prompt pair and add it to the residual stream during inference.

## Algorithm

1. Pick two contrastive prompts: `p+` (positive example of a concept) and `p−` (negative example). Example: `"Love"` vs `"Hate"`.
2. Run both prompts through the LM up to a chosen layer `L`. Record the residual-stream activation at the **last token position** for each: `h+`, `h-`.
3. Compute the steering vector: `v = h+ − h−`.
4. At inference time, for each forward pass on test inputs, **add** `c · v` to the residual stream at layer `L`, where `c` is a tuned scalar coefficient.

That's it. No training, no optimization, no learning rate. One pair of demonstrations and a vector subtraction.

## Why it's notable

- **Trivial cost**: one vector add per inference. Compares favorably to retraining or RL fine-tuning.
- **Surprising effectiveness**: single contrastive pair often produces SOTA-level behavior shifts on sentiment, toxicity, topic control.
- **Off-target preservation**: unrelated benchmarks (perplexity, ARC) typically unchanged. The steering is localized to the targeted dimension.
- **Sets the template** for the rest of the [[steering]] literature: every later method ([[contrastive-activation-addition|CAA]], [[representation-engineering|RepE]], SAE-based steering) is a refinement of ActAdd.

## Limitations

- **Single pair is noisy**: a single contrastive example can pick up spurious directions. CAA fixes this by averaging over many pairs.
- **Layer choice matters**: typically middle layers work; extremes degrade fluency. Some empirical sweep required.
- **Coefficient tuning**: too small = no effect, too large = output collapse. Sweet spot is narrow.

## References

- [[activation-engineering-turner-2023]]
