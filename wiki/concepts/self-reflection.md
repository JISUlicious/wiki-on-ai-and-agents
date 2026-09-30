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
- **[[constitutional-ai|Constitutional AI]]** (Bai et al. 2022, Anthropic): the model critiques itself against a written constitution during training, used to generate preference data without human labels.
- **Reasoning models** ([[o1|OpenAI o1]]+, 2024): self-reflection is internalized during post-training, so the model naturally produces hidden chains-of-thought with self-correction baked in.

## Caveat

Self-reflection is bounded by the model's own competence. If the model can't tell that an answer is wrong, asking it to critique itself produces a confidently-wrong critique. Self-reflection helps most when the **evaluation problem is easier than the generation problem** — a common but not universal pattern (e.g., it's easier to check if Python code passes unit tests than to write the function from scratch).

> [!warning] Contradiction — at equal token cost, self-inspection loses to sampling more
> [[sample-more-reflect-less-mirzaei-2026|Mirzaei 2026]] rebuilt the comparison as a designed experiment: 7 methods, Qwen2.5 at 1.5B/3B/7B, GSM8K + MATH-500, **every generated token counted** — critiques, reflections, debate turns, checking — with each method matched against [[self-consistency|repeated sampling]] interpolated to its own *measured* cost. Across 36 paired comparisons with bootstrap intervals and Holm correction: **0 significantly better, 10 significantly worse, 26 indistinguishable**, and **all 18 self-inspection comparisons are negative**.
>
> The cleanest experiment holds the 8 Best-of-N samples fixed and changes only the final step. Counting beats judging by **8.0 / 11.3 points at 1.5B** and 5.3 / 17.3 at 3B — shrinking to 2.0 / 1.3 at 7B, where it is no longer distinguishable from zero. So the caveat above is not merely supported but quantified: the untrained verifier is right in only **3.6–33.3%** of cases where it overrides the majority, at every scale tested.
>
> **This also puts one of the variants above on the wrong side of a line.** Listing self-consistency as "implicit reflection via majority" groups it with the methods it defeats. Counting versus judging is the central split in these results, and counting is the winning side — self-consistency is the *baseline that self-reflection fails to beat*, not a species of it.
>
> Scope: math with exact-match answers, open models ≤7B, one implementation and judging prompt. It does not refute the coding or agentic results in [[reflexion-shinn-2023]]; it establishes that those results do not show the *mechanism* is responsible, because none were budget-matched.

## References

- [[reflexion-shinn-2023]]

## Related 2026 sources

- [[metacognition]] — the monitoring/control distinction that separates what works here from what doesn't
