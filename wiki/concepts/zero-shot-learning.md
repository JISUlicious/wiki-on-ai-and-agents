---
title: Zero-Shot Learning
type: concept
created: 2026-05-13
updated: 2026-05-13
sources:
  - gpt-2-radford-2019.md
status: complete
importance: high
---

# Zero-Shot Learning

In the context of language models, **zero-shot learning** means performing a task without any labeled training examples for that task — neither fine-tuning weights nor providing in-context demonstrations. The task is specified entirely by a natural-language prompt (e.g., `"translate to french: <text>"`), and the model produces an answer by continuing the prompt.

## Origins in LLMs

[[gpt-1]] briefly analyzed zero-shot behavior in its final sections. [[gpt-2-radford-2019|GPT-2]] (2019) made zero-shot the central thesis: "Language models are unsupervised multitask learners." The paper argued that sufficiently large language models trained on diverse web text implicitly absorb many tasks because demonstrations of those tasks occur naturally in the training data, and that those tasks can be elicited zero-shot at inference time.

## Zero-shot vs. few-shot vs. fine-tuning

- **Zero-shot**: task in the prompt only, no examples.
- **Few-shot / [[in-context-learning|in-context]]**: task with `k` demonstration (input, output) pairs in the prompt, then a new input. This is what [[gpt-3-brown-2020|GPT-3]] popularized.
- **[[fine-tuning|Fine-tuning]]**: explicit gradient updates on labeled task data.

The progression GPT-1 → GPT-2 → GPT-3 traces a clear arc: fine-tuning required, zero-shot possible (GPT-2), few-shot competitive with fine-tuning at scale (GPT-3).

## Caveat

"Zero-shot" in the LLM literature has a specific operational meaning (no examples in the prompt) that differs subtly from classical zero-shot learning in vision/NLP (where the model has never been trained on the class label at all). Modern LLMs have almost certainly seen *something* like every common task during pre-training — the "zero-shot" claim is about no *additional* examples at inference, not about the model never having seen anything like the task.

## References

- [[gpt-2-radford-2019]]
