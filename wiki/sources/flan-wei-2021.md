---
title: "Finetuned Language Models Are Zero-Shot Learners (FLAN)"
type: source
created: 2026-05-14
updated: 2026-05-14
sources:
  - wei-flan-2021.md
arxiv_id: "2109.01652"
venue: ICLR 2022
authors:
  - "[[jason-wei]]"
  - Maarten Bosma
  - Vincent Y. Zhao
  - Kelvin Guu
  - Adams Wei Yu
  - Brian Lester
  - Nan Du
  - Andrew M. Dai
  - "[[quoc-le]]"
first_author: "[[jason-wei]]"
introduces:
  - "[[instruction-tuning]]"
  - "[[flan]]"
year: 2021
tags:
  - 2021
status: complete
importance: high
---

# Finetuned Language Models Are Zero-Shot Learners (FLAN)

**Source**: `sources/wei-flan-2021.md` (extracted from `wei-flan-2021.pdf`)
**arXiv**: [2109.01652](https://arxiv.org/abs/2109.01652)
**Venue**: ICLR 2022
**Authors**: [[jason-wei]], Maarten Bosma, Vincent Y. Zhao, Kelvin Guu, Adams Wei Yu, Brian Lester, Nan Du, Andrew M. Dai, [[quoc-le]] ([[google-research|Google Research]])

## Summary

This paper introduces **[[instruction-tuning|instruction tuning]]** — fine-tuning a pre-trained language model on a collection of NLP tasks **expressed as natural-language instructions**. The trained model, **FLAN** (Finetuned Language Net), is then evaluated zero-shot on **held-out task types** to test generalization.

Setup: take a 137B-parameter pre-trained LM (LaMDA-PT). Fine-tune on 60+ NLP datasets, each verbalized via multiple natural-language instruction templates (e.g., "Translate this sentence to Spanish: ... → ..." or "Premise: ... Hypothesis: ... Does the premise entail the hypothesis?"). Evaluate on held-out **task types** (e.g., train on translation, sentiment, etc.; test on natural language inference). Compare zero-shot to base 137B LM and to 175B [[gpt-3]].

The result: FLAN substantially outperforms its base model and surpasses zero-shot 175B GPT-3 on **20 of 25 datasets** evaluated. On some (ANLI, RTE, BoolQ, AI2-ARC, OpenbookQA, StoryCloze) FLAN even beats *few-shot* GPT-3. Ablations show that **(1) number of fine-tuning tasks**, **(2) model scale**, and **(3) using natural language instructions specifically** (vs. dataset names) are all critical.

The paper's lasting contribution is the recipe: collect many tasks, verbalize them as instructions, fine-tune. The result is a model that follows new instructions zero-shot. This is the **non-RLHF cousin** of [[instructgpt-ouyang-2022|InstructGPT]] (which appears 5 months later and adds RLHF on top of SFT-on-instructions). Both papers point to the same insight: pre-trained LMs need a fine-tuning step that exposes them to **diverse natural-language task descriptions** to become useful assistants.

The FLAN recipe was extended in subsequent papers (FLAN-T5, Chung et al. 2022; FLAN-PaLM; T0; Super-NaturalInstructions) into the family of "instruction-tuned" open-weight models that dominated the 2022–2023 era of fine-tuned LLMs.

## Key Points

- **Instruction tuning recipe**: fine-tune the pre-trained LM on `(instruction, input, output)` triples drawn from many NLP datasets verbalized as natural-language tasks.
- **Held-out task generalization**: at evaluation time, the model sees task *types* it never saw during fine-tuning. Performance on these unseen types is the measure of instruction-following generalization.
- **Ablation findings**:
  - **Scale matters**: at 8B, instruction tuning hurts; at 68B+, it helps; gains grow with scale. Emergent behavior similar to [[chain-of-thought-prompting|CoT]].
  - **Number of tasks matters**: more tasks → better generalization.
  - **Natural-language instructions matter**: dataset-name templates fail; only NL instructions generalize.
- **Result**: zero-shot FLAN beats zero-shot GPT-3 175B on 20/25 datasets; few-shot FLAN often beats few-shot GPT-3.
- **Non-RLHF**: pure supervised fine-tuning. No human preference data, no reward model. The complementary RLHF path is in [[instructgpt-ouyang-2022|InstructGPT]] (Ouyang 2022).

## Entities Mentioned

- [[jason-wei]] — first author (also [[chain-of-thought-wei-2022|CoT prompting]])
- [[quoc-le]] — senior author
- [[google-research]] — author affiliation
- [[flan]] — the model

## Concepts Discussed

- [[instruction-tuning]] — the recipe introduced
- [[flan]] — the model name
- [[zero-shot-learning]] — the evaluation regime
- [[emergent-abilities]] — instruction tuning helps only at sufficient scale

## Notable Quotes

> "We show that instruction tuning — finetuning language models on a collection of tasks described via instructions — substantially improves zero-shot performance on unseen tasks." — Abstract

> "FLAN substantially improves the performance of its unmodified counterpart and surpasses zero-shot 175B GPT-3 on 20 of 25 datasets that we evaluate." — Abstract

## References

_Original source: `sources/wei-flan-2021.md`_
