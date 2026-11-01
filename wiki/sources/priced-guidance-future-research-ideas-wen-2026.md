---
title: "Priced Guidance: Can Language Models Generate Future Research Ideas?"
type: source
created: 2026-10-29
updated: 2026-10-29
sources:
  - priced-guidance-future-research-ideas-wen-2026.md
arxiv_id: "2610.04976"
authors:
  - Kaiyue Wen
  - Tengyu Ma
  - Percy Liang
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# Priced Guidance: Can Language Models Generate Future Research Ideas?

**Kaiyue Wen, Tengyu Ma, [[percy-liang|Percy Liang]]** ([[stanford-university|Stanford University]]). [arXiv:2610.04976](https://arxiv.org/abs/2610.04976), v1, cs.LG, 4 Oct 2026. Preprint; code at github.com/WhenWen/priced-guidance.

## Summary

Asking whether a model *can* produce a future paper's idea runs into a tiny probability that is too expensive to estimate by sampling. Priced Guidance measures **compression cost** instead, as a game of Twenty Questions:

- The **generator** (the model under test) poses multiple-choice questions and puts a probability on each answer.
- A **guide** (Fable 5) that has read the hidden target paper picks the answers. Picking an answer of probability p costs −log₂ p bits.
- An LLM **judge** checks whether the final idea matches the target's essence: its central research object and defining mechanism.

**Theorem 2.2:** if guided runs cost K bits, the *unguided* generator, sampling its own answers, produces an accepted idea with probability at least E[2^−K]. Lower cost therefore means a stronger idea generator. The guide may be arbitrarily strong, because only the information it sends is counted.

## Key points

- **Test set:** 87 recent Top-1 Hugging Face Daily Papers. Generators: Opus 5, Fable 5.1, GPT-6 Astra, GPT-5.6 Sol, GLM 5.3.
- **Median cost (Essence judge):** Fable 5.1 lowest at **69.9 bits**, then 70.4 and 71.6 for the next two. GLM 5.3 needs 102.4.
- A uniform **ensemble** of Fable 5.1, Opus 5 and Astra reaches **55.8 bits**, which is 14.1 bits or about 18,000× better on the probability bound.
- **For scale:** compressing the same gold summary *verbatim* costs 1,147 bits with GLM 5.3 and 5,712 bits with gzip.
- Generator post-training helps: GLM 5.2 → 5.3 lowers cost substantially. Harder targets separate frontier models from GLM more.
- **No knowledge-cutoff drift:** the ensemble's cost does not correlate with paper date over the four-month window.
- **Technical reports are hardest.** Kimi K3 was the only target no generator recovered under the Directional judge.

## What it does not show

- **The bound is extremely loose in absolute terms.** 2^−56 is about 10^−17, so this is not evidence that a model would produce these ideas unprompted. The authors frame cost as a capability *measure*, not a threshold for autonomy.
- **LLMs at every step.** LLMs summarize the targets and judge the ideas, so judge hacking is possible (they saw vague phrasing, nothing worse). The guide is a frontier model from the same family as one of the generators.
- **Generation, not judgment.** It measures whether a model can produce a target idea at least once. It does not test whether the model knows which ideas work. The scaffold is heuristic, with no tool calls.

## Concepts & entities

- [[ideaanchor-literature-to-research-ideas-chen-2026]]: rubric-based ideation training, a different way to score ideas.
- [[ai-research-preference-models-foster-2026]] · [[scientisttwo-autonomous-ai-human-knowledge-frontier-nam-2026]]: neighbouring work on AI research ideas.
- [[llm-as-a-judge]] · [[benchmark-contamination]] (cutoff analysis) · [[percy-liang]] · [[stanford-university]]

## References

- [arXiv:2610.04976](https://arxiv.org/abs/2610.04976)
