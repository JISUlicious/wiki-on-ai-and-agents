---
title: Few-Shot Learning
type: concept
created: 2026-05-13
updated: 2026-05-13
sources:
  - gpt-3-brown-2020.md
status: complete
importance: medium
---

# Few-Shot Learning

In the LLM literature, **few-shot learning** means specifying a task to a pre-trained language model by providing a natural-language instruction **plus a small number** (typically 1 to ~100) of `(input, output)` example pairs in the prompt, then asking the model to produce the output for a new input. No gradient updates are performed. It is a special case of [[in-context-learning|in-context learning]].

## Distinguishing terms

- **[[zero-shot-learning|Zero-shot]]**: instruction only.
- **One-shot**: instruction + 1 demonstration.
- **Few-shot**: instruction + k demonstrations (k usually small).

These are all distinct from classical "few-shot learning" in ML (where the model is *trained* on a small labeled support set — e.g., MAML-style meta-learning). In the LLM setting there is no training at the few-shot stage; all the "learning" happens in the forward pass of inference.

## Empirical scaling

[[gpt-3-brown-2020|GPT-3]] showed that few-shot performance improves *faster* with model size than zero-shot performance does. This is the central empirical claim of the paper: larger models are not just better LMs, they are better in-context learners. The gap between few-shot and zero-shot widens with scale.

## Sensitivities

- **Example selection**: which examples you pick matters a lot, especially for smaller models. Retrieval-augmented few-shot (kNN over a labeled pool) helps.
- **Ordering**: example order can swing accuracy by several points.
- **Label / format**: format mismatches between examples and the test input can cost accuracy.
- **Calibration**: few-shot models are often poorly calibrated; "contextual calibration" methods (Zhao et al. 2021) can recover several points.

## Modern picture

Few-shot prompting was the dominant adaptation mechanism from GPT-3 (2020) until roughly 2023. As context windows grew (32k → 200k → 1M tokens) and as instruction-tuned models became strong zero-shot followers, **zero-shot with detailed instructions** has become competitive with few-shot for many tasks. Few-shot remains preferred when the task is novel, has unusual output format, or when subtle stylistic priors are important.

## References

- [[gpt-3-brown-2020]]
