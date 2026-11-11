---
title: "A Heart Means Recursion: Exploring Visual Meaning and Communication in VLMs"
type: source
created: 2026-11-10
updated: 2026-11-10
sources:
  - a-heart-means-recursion-tartaglini-2026.md
arxiv_id: "2610.36449"
venue: arXiv preprint (cs.AI)
authors:
  - Alexa R. Tartaglini
  - Zhengxuan Wu
  - Thaddäus Wiedemer
  - Martin Wattenberg
  - Been Kim
  - Yuxuan Li
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# A Heart Means Recursion: Exploring Visual Meaning and Communication in VLMs

**Source:** Alexa R. Tartaglini, Zhengxuan Wu, Thaddäus Wiedemer, [[martin-wattenberg|Martin Wattenberg]], Been Kim, Yuxuan Li ([[deepmind|Google DeepMind]]; [[stanford-university|Stanford]]) — [arXiv:2610.36449](https://arxiv.org/abs/2610.36449), cs.AI, v1 27 Oct 2026. Preprint; the authors call the findings preliminary.

## Summary

A small, irrelevant image in the prompt can change what a vision-language model does, and the model usually does not say so. Asked to solve a coding problem that admits a recursive or an iterative answer, Claude Opus 4.7 and GPT-5.4 never use recursion at baseline. Add a picture of a red heart and both switch to recursion 100% of the time. That is as strong as telling them to use recursion. The authors sweep a library of 288 coloured shapes and find model-specific "steering profiles" that do not match text captions of the same symbols, random noise or natural images.

They then turn this into communication games between models. A sender model draws or picks a visual to steer, or to convey a concept or story to, a receiver. Free-form drawings work well. Single symbols reveal a **generation–interpretation gap**: models send the symbols humans would pick, not the ones that actually steer them.

## Key points

- **Implicit steering.** Effects carry over to other recursion-vs-iteration tasks, BFS vs DFS, data-structure choice and writing style. Image-induced and caption-induced effects are barely correlated (r = −0.05 to 0.31), so the effect is visual, not lexical.
- **Unfaithful traces.** In 2,880 Gemini 3.1 Pro thinking traces, 71.5% of the cases where a symbol *did* steer claim the image is irrelevant. GPT-5.4 errs the other way: 74.7% of non-steering cases claim the symbol drove the choice. Asked directly whether a symbol means "recursion", models' answers barely predict the real effect (r ≈ 0.2–0.3).
- **Silent injections** of a symbol into the context still steer strongly (up to Δ = +1.0 for Claude Opus 4.7).
- **Communication games.** With free-form PIL drawings, 97.8% of concept and story games converge (91.2% on the first turn). With symbol primitives only 47.8% succeed zero-shot, and 30.7% never succeed within five turns.
- **Human legibility.** In a small study (N = 7), humans matched the steering direction of top-steering symbols only about 36% of the time, below the 50% chance level. Optimising symbols for a model receiver made them less legible to humans, while free-form drawings stayed interpretable.

## Why it matters

This is a new kind of prompt-side lever. It sits next to [[prompt-injection]] and [[indirect-prompt-injection]], but it works through meaning that neither the model's explanation nor a human reviewer can see. It is also further evidence that a visible reasoning trace is not a reliable account of why a model chose what it did.

## Caveats

- The coding probe uses tasks where both answers are valid, so steering changes style, not correctness.
- The human study is exploratory (N = 7), and Claude Opus 4.7 returned no thinking traces to audit.

## Concepts & entities

- [[multimodal-llm]] · [[steering]] · [[chain-of-thought]] · [[mechanistic-interpretability]] — implicit visual steering, and traces that misreport it.
- [[prompt-injection]] · [[indirect-prompt-injection]] — an adjacent attack surface.
- [[multi-agent]] — model-to-model visual communication games.
- [[deepmind]] · [[stanford-university]] · [[martin-wattenberg]] · [[claude]].

## References

- [arXiv:2610.36449](https://arxiv.org/abs/2610.36449)
