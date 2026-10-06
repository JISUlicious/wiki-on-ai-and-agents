---
title: "Pre-train, Prompt, and Predict: A Systematic Survey of Prompting Methods in Natural Language Processing"
type: source
created: 2026-10-06
updated: 2026-10-06
sources:
  - pre-train-prompt-predict-liu-2021.md
arxiv_id: "2107.13586"
venue: "arXiv preprint (later ACM Computing Surveys)"
authors:
  - Pengfei Liu
  - Weizhe Yuan
  - Jinlan Fu
  - Zhengbao Jiang
  - Hiroaki Hayashi
  - Graham Neubig
first_author: Pengfei Liu
introduces:
  - "[[prompt-engineering]]"
year: 2021
tags:
  - 2021
  - paper
  - survey
status: complete
importance: high
---

# Pre-train, Prompt, and Predict

**Source**: `sources/pre-train-prompt-predict-liu-2021.md` ([arXiv:2107.13586](https://arxiv.org/abs/2107.13586))
**Authors**: Pengfei Liu, Weizhe Yuan, Jinlan Fu, Zhengbao Jiang, Hiroaki Hayashi, Graham Neubig ([[cmu|Carnegie Mellon University]], National University of Singapore)
**Date**: **2021-07-28** (arXiv v1). The ACM Computing Surveys venue is from general knowledge.

## Summary

The survey that formalized **prompt-based learning** as an NLP paradigm. Instead of modelling P(y|x) directly:
- a *prompting function* rewrites the input x into a template with unfilled slots;
- a pre-trained LM fills the slots;
- an *answer mapping* turns the filled text into an output y.

This enables few-shot and zero-shot use of one pre-trained model across many tasks.

## Four paradigms of NLP (Table 1)

| Paradigm | When (per the survey's text) | What gets "engineered" |
|---|---|---|
| Fully supervised, non-neural | early NLP | **Feature** engineering |
| Fully supervised, neural | features learned jointly (Collobert 2011 onward) | **Architecture** engineering |
| Pre-train, fine-tune | the "sea change" of 2017–2019 | **Objective** engineering |
| **Pre-train, prompt, predict** | the "second sea change", under way "as of this writing in 2021" | **Prompt** engineering |

The "engineering" column is the useful part for lineage. It puts **[[prompt-engineering|prompt engineering]]** in a sequence of what practitioners hand-design, and the agent era extends the same sequence:
- context engineering ([[context-engineering]], 2025);
- harness engineering ([[agentic-harness-engineering]], 2026);
- arguably "loop engineering".

## Formal definition of prompt engineering

"**Prompt engineering** is the process of creating a prompting function f_prompt(x) that results in the most effective performance on the downstream task." The survey's typology:
- prompt shape: *cloze* vs *prefix*;
- manual templates vs *automated* ones, in discrete form (mined, paraphrased, gradient-searched) or continuous form (prefix-tuning, soft prompts);
- **answer engineering**: designing the answer space and how it maps to outputs;
- multi-prompt methods: ensembling, augmentation, composition, decomposition;
- training strategies: tuning-free prompting, fixed-LM prompt tuning, and so on.

> [!note] Is this the origin of "prompt engineering"?
> Probably not the first use of the phrase. The survey cites earlier work, and the term was in informal use around GPT-3 in 2020. It is the **first systematic, formal definition** in this wiki, and the paper most often cited for the paradigm. Treat it as the formalization point (July 2021), not the coinage.

## Lineage

- **Built on:** [[gpt-3|GPT-3]] [[in-context-learning]] (2020); cloze-style prompting such as LAMA and PET (see [[cloze-scoring]]); prefix-tuning and prompt tuning.
- **Fed into:**
  - instruction tuning ([[flan-wei-2021|FLAN]], Sep 2021), which trains models to follow prompts;
  - [[chain-of-thought-prompting|CoT]] (Jan 2022);
  - the prompt-engineering practice of 2022–23;
  - [[context-engineering]] (2025), which Anthropic frames as "the natural progression of prompt engineering".

## Concepts & entities

[[prompt-engineering]] · [[in-context-learning]] · [[cloze-scoring]] · [[pre-training]] · [[fine-tuning]] · [[context-engineering]] · [[gpt-3]] · [[cmu]]
