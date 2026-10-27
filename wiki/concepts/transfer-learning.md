---
title: Transfer Learning
type: concept
created: 2026-10-27
updated: 2026-10-27
sources:
  - transferable-features-deep-nets-yosinski-2014.md
  - visualizing-understanding-convnets-zeiler-2013.md
status: draft
importance: medium
---

# Transfer Learning

Transfer learning trains a network on a large *base* task and reuses its learned features for a *target* task. The reused layers are either kept **frozen**, with only new top layers trained, or **[[fine-tuning|fine-tuned]]** along with them.

In the ImageNet era this was the main justification for large-scale pretraining. It is the direct ancestor of the pretrain-then-fine-tune recipe for [[bert|BERT]] and LLMs.

## Key results

- **Frozen ImageNet features beat hand-crafted ones** ([[visualizing-understanding-convnets-zeiler-2013]]). A softmax trained on them reaches 86.5% on Caltech-101 and 74.2% on Caltech-256, against 55.2% previously.
- **Transferability falls with depth** ([[transferable-features-deep-nets-yosinski-2014]]):
  - layers 1–2 transfer almost perfectly;
  - middle layers lose accuracy because co-adapted neighbouring layers are split;
  - top layers lose it because their features are specific to the task.
- **Distance between tasks matters.** For dissimilar tasks the gap at the final layer is 25%, against 8% for similar ones. Even distant transfer beats random weights.
- **Pretraining leaves a lasting benefit.** Transferring and then fine-tuning generalises better than training on the target from scratch (+1.6% to +2.1%).

## Related

- [[feature-hierarchy]]: why early layers are the reusable ones
- [[fine-tuning]] · [[lora]] · [[convolutional-neural-network]]

## References

- [[transferable-features-deep-nets-yosinski-2014]] · [[visualizing-understanding-convnets-zeiler-2013]]
