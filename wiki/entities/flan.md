---
title: FLAN
type: entity
created: 2026-05-14
updated: 2026-05-14
sources:
  - wei-flan-2021.md
family: flan
predecessor: null
successors: []
parameters: "137B (original); subsequent Flan-T5 / Flan-PaLM derivatives at various sizes"
release_date: 2021-09-03
status: draft
importance: medium
tags:
  - model
---

# FLAN

FLAN (Finetuned Language Net) is the 137B-parameter [[instruction-tuning|instruction-tuned]] model introduced in [[flan-wei-2021|"Finetuned Language Models Are Zero-Shot Learners"]] (Wei et al., 2021) at [[google-research|Google Research]].

The base model is LaMDA-PT (a 137B decoder-only pre-trained LM). FLAN is the result of fine-tuning LaMDA-PT on 60+ NLP datasets verbalized as natural-language instructions. The model name has since been overloaded to describe an entire *family* of follow-up models that apply the same recipe to other base models:

- **Flan-T5** (Chung et al. 2022): apply FLAN's instruction-tuning recipe (with a much larger 1.8k-task dataset) to [[t5|T5]]. Open-weight; became the default strong open-weight instruction-tuned encoder-decoder for several years.
- **Flan-PaLM** (Chung et al. 2022): same recipe applied to [[palm|PaLM]]. Better than FLAN at the 540B-parameter scale.
- **Flan-UL2**, etc.

The original 137B FLAN itself was never publicly released; the lineage's impact came through the Flan-T5 open release and the broader instruction-tuning paradigm it crystallized.

## References

- [[flan-wei-2021]]
