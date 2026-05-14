---
title: "Training language models to follow instructions with human feedback (InstructGPT)"
type: source
created: 2026-05-14
updated: 2026-05-14
sources:
  - ouyang-instructgpt-2022.md
arxiv_id: "2203.02155"
venue: NeurIPS 2022
authors:
  - Long Ouyang
  - Jeff Wu
  - Xu Jiang
  - Diogo Almeida
  - Carroll L. Wainwright
  - Pamela Mishkin
  - Chong Zhang
  - Sandhini Agarwal
  - Katarina Slama
  - Alex Ray
  - John Schulman
  - Jacob Hilton
  - Fraser Kelton
  - Luke Miller
  - Maddie Simens
  - Amanda Askell
  - Peter Welinder
  - "[[paul-christiano]]"
  - "[[jan-leike]]"
  - Ryan Lowe
first_author: Long Ouyang
introduces:
  - "[[instructgpt]]"
year: 2022
tags:
  - 2022
status: complete
importance: high
---

# Training language models to follow instructions with human feedback (InstructGPT)

**Source**: `sources/ouyang-instructgpt-2022.md` (extracted from `ouyang-instructgpt-2022.pdf`)
**arXiv**: [2203.02155](https://arxiv.org/abs/2203.02155)
**Venue**: NeurIPS 2022
**Authors**: Long Ouyang, Jeff Wu, Xu Jiang, ..., John Schulman (RL lead), Amanda Askell, [[paul-christiano]], [[jan-leike]], Ryan Lowe ([[openai|OpenAI]])

## Summary

This paper codifies the **SFT → RM → RLHF** pipeline that turned raw [[gpt-3]] into [[instructgpt|InstructGPT]] — and, with minor variations, into ChatGPT (released ~9 months later, Nov 2022). The recipe has three stages:

1. **Supervised Fine-Tuning (SFT)**: human labelers write demonstrations of desired model behavior on a wide range of prompts. Fine-tune GPT-3 on these `(prompt, response)` pairs via standard supervised learning.
2. **Reward Model (RM)**: human labelers rank multiple SFT outputs for each prompt. Train a reward model (a fine-tuned LM with a scalar head) to predict these rankings via pairwise Bradley-Terry loss.
3. **PPO against the RM**: fine-tune the SFT model via PPO to maximize the RM's score, with a KL penalty against the SFT policy to prevent reward hacking.

This is a direct application of the [[rlhf-christiano-2017|Christiano et al. 2017]] RLHF recipe to language modeling.

The headline result: in human evaluations on the prompt distribution OpenAI's API actually saw, outputs from the **1.3B InstructGPT** model are preferred to outputs from **175B raw GPT-3** — despite the InstructGPT model being **100× smaller**. InstructGPT also improves truthfulness, reduces toxic outputs, and has minimal performance regressions on public NLP benchmarks compared to GPT-3 of the same size.

The paper is the explicit recipe behind ChatGPT and every general-purpose chat LLM since. The "instruction-following assistant" form factor that ChatGPT made famous is precisely what this pipeline produces.

## Key Points

- **Three-stage pipeline**: SFT → RM → PPO. Each stage uses human labels.
- **SFT data**: ~13k demonstrations written by 40 hired labelers covering a wide prompt distribution.
- **RM data**: ~33k prompts × ~4-9 ranked responses each. Bradley-Terry loss.
- **PPO**: KL-regularized policy optimization against the RM. Prevents reward hacking by penalizing divergence from the SFT initialization.
- **Result**: 1.3B InstructGPT preferred to 175B raw GPT-3 by labelers. Smaller alignment beats raw scale on usefulness metrics.
- **Alignment tax**: minimal regression on public NLP benchmarks (vs. the gain on instruction-following).
- **Generalization**: InstructGPT also generalizes to instruction types and languages it wasn't fine-tuned on (limited but real).
- **Direct lineage to ChatGPT**: published March 2022. ChatGPT (a fine-tuned GPT-3.5 with this same pipeline + dialog-specific data) released November 2022.

## Entities Mentioned

- [[paul-christiano]], [[jan-leike]] — senior alignment-team authors (both later left OpenAI for different reasons)
- John Schulman — PPO co-creator, was at OpenAI; later co-founded an Anthropic-adjacent venture
- [[openai]] — origin org
- [[instructgpt]] — the model

## Concepts Discussed

- [[rlhf]] — this is its canonical LLM-era application
- [[preference-modeling]] — RM training via Bradley-Terry loss
- [[reward-modeling]] — the RM stage
- [[fine-tuning]] — SFT and RLHF are both forms of fine-tuning
- [[reward-hacking]] — KL penalty exists specifically to prevent it
- [[instructgpt]] — the model produced

## Notable Quotes

> "Making language models bigger does not inherently make them better at following a user's intent. ... In human evaluations on our prompt distribution, outputs from the 1.3B parameter InstructGPT model are preferred to outputs from the 175B GPT-3, despite having 100x fewer parameters." — Abstract

> "Fine-tuning with human feedback is a promising direction for aligning language models with human intent." — Abstract

## References

_Original source: `sources/ouyang-instructgpt-2022.md`_
