---
title: "Distilling System 2 into System 1"
type: source
created: 2026-10-10
updated: 2026-10-10
sources:
  - distilling-system-2-into-system-1-yu-2024.md
arxiv_id: "2407.06023"
venue: arXiv preprint
authors:
  - Ping Yu
  - Jing Xu
  - Jason Weston
  - Ilia Kulikov
first_author: Ping Yu
year: 2024
tags:
  - 2024
  - paper
status: complete
importance: high
---

# Distilling System 2 into System 1

**Source**: `sources/distilling-system-2-into-system-1-yu-2024.md` ([arXiv:2407.06023](https://arxiv.org/abs/2407.06023), 2024-07-08)
**Authors**: Ping Yu, Jing Xu, Jason Weston, Ilia Kulikov ([[meta]] FAIR)

## Summary

**The framing:**
- **System 1** = an LLM answering directly.
- **System 2** = any method that spends extra inference compute generating intermediate thoughts, such as [[chain-of-thought-prompting|CoT]], Rephrase-and-Respond (RaR), [[system-2-attention-weston-2023|System 2 Attention]] (S2A) or Branch-Solve-Merge (BSM).

The paper **"compiles" System 2 outputs back into System 1** with self-supervised distillation:
1. run the System 2 method on unlabelled inputs;
2. filter its outputs by **self-consistency**: majority vote, or universal self-consistency for open-ended outputs;
3. fine-tune the model to produce the final answer **directly, with no intermediate tokens**.

**The human analogy, explicit:** in psychology, moving a skill from deliberate (System 2) to automatic (System 1) is **automaticity**, which relies on procedural memory (citing Kahneman 2011).

## Results

- **RaR (Llama-2-70B-chat, last-letter concatenation):** System 1 scored 30.0%. 1-step RaR scored 39.5% and 2-step RaR 44.5%. **Distilled 2-step RaR scored 98.0%**, at System 1 cost.
- **S2A and BSM distill well too.** Distilled S2A reduces bias on SycophancyEval, and distilled BSM matches or beats BSM as an LLM judge, at a fraction of the tokens.
- **The exception:** "In this case System 2 Distillation of **CoT does not work well**" on [[gsm8k|GSM8K]]. Some tasks still need the deliberate trace, as with humans who cannot do some tasks without reasoning step by step.

## Why it matters

- **The bridge from "System 2 methods" to "System 1 models".** A fast, single-pass model can absorb what a slow method produced, for the tasks where the slow method's output is learnable as a direct mapping.
- **That describes the System One product class.** [[s1-decision-model-raghavan-2026|s1]] uses its base model as a teacher for soft targets. More broadly, typed decision models compile judgment into one forward pass ([[system-one-model]]).
- **The limit applies to them too.** Multi-step arithmetic and long reasoning resist this kind of compilation. That is where System One models are explicitly *not* meant to compete.
- **The authors' outlook:** "System 2 distillation will be an important feature of future continually learning AI systems, enabling them to focus System 2 capabilities on the reasoning tasks that they cannot yet do well."

## Concepts & entities

[[dual-process-theory]] · [[knowledge-distillation]] · [[self-consistency]] · [[chain-of-thought-prompting]] · [[system-one-model]] · [[system-one-models-and-dual-process-thinking]] · [[test-time-compute]]
