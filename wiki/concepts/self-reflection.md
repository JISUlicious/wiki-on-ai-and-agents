---
title: Self-Reflection (in LLMs)
type: concept
created: 2026-05-13
updated: 2026-05-13
sources:
  - reflexion-shinn-2023.md
status: draft
importance: medium
---

# Self-Reflection (in LLMs)

Self-reflection in the LLM context means **prompting a model to critique, evaluate, or improve its own output**. The model is repurposed as its own judge: it produces a response, then is asked something like "What's wrong with that answer?" or "How could this be improved?", and the critique is used to drive a revision.

## Variants

- **Self-Refine** (Madaan et al. 2023): single-task iterative refinement. Generate → critique → revise → repeat.
- **[[reflexion|Reflexion]]** (Shinn et al. 2023): episodic memory of reflections across attempts.
- **Self-consistency** (Wang et al. 2022): sample many reasoning paths, vote on the answer — implicit reflection via majority.
- **Constitutional AI** (Bai et al. 2022, Anthropic): the model critiques itself against a written constitution during training, used to generate preference data without human labels.
- **Reasoning models** (OpenAI o1+, 2024): self-reflection is internalized during post-training, so the model naturally produces hidden chains-of-thought with self-correction baked in.

## Caveat

Self-reflection is bounded by the model's own competence. If the model can't tell that an answer is wrong, asking it to critique itself produces a confidently-wrong critique. Self-reflection helps most when the **evaluation problem is easier than the generation problem** — a common but not universal pattern (e.g., it's easier to check if Python code passes unit tests than to write the function from scratch).

## References

- [[reflexion-shinn-2023]]
