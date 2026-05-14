---
title: "Exploring the Limits of Transfer Learning with a Unified Text-to-Text Transformer (T5)"
type: source
created: 2026-05-14
updated: 2026-05-14
sources:
  - raffel-t5-2019.md
arxiv_id: "1910.10683"
venue: JMLR 2020
authors:
  - "[[colin-raffel]]"
  - "[[noam-shazeer]]"
  - Adam Roberts
  - Katherine Lee
  - Sharan Narang
  - Michael Matena
  - Yanqi Zhou
  - Wei Li
  - Peter J. Liu
year: 2019
tags:
  - 2019
status: complete
importance: high
introduces:
  - "[[t5]]"
  - "[[c4]]"
---

# Exploring the Limits of Transfer Learning with a Unified Text-to-Text Transformer (T5)

**Source**: `sources/raffel-t5-2019.md` (extracted from `raffel-t5-2019.pdf`)
**arXiv**: [1910.10683](https://arxiv.org/abs/1910.10683)
**Venue**: JMLR 2020
**Authors**: [[colin-raffel]] (equal), [[noam-shazeer]] (equal), Adam Roberts (equal), Katherine Lee (equal), and others ([[google-brain|Google]])

## Summary

T5 ("Text-to-Text Transfer Transformer") is a [[transformer-architecture|Transformer]] **encoder-decoder** model and an exhaustive empirical study of NLP transfer-learning design choices. Its defining framing: **every NLP task is recast as a text-to-text problem**. Translation, summarization, classification, regression, span extraction — they all become "given this input text, produce this output text."

This unified framing means a *single* model and a *single* training recipe can serve heterogeneous downstream tasks; only the input string changes. Examples (from the paper):
- `"translate English to German: That is good."` → `"Das ist gut."`
- `"cola sentence: The course is jumping well."` → `"unacceptable"`
- `"stsb sentence1: The rhino grazed on the grass. sentence2: A rhino is grazing in a field."` → `"3.8"` (regression, rendered as text)

Beyond the framing, the paper is a *systematic ablation study* of pre-training objectives, architectures, datasets, fine-tuning approaches, and scaling. It compares span-corruption pretraining vs. autoregressive LM vs. BERT-style MLM; encoder-decoder vs. decoder-only vs. encoder-only; and so on. The paper's recommendations became the default recipe for the encoder-decoder seq2seq line for several years.

Side contributions: the **C4 (Colossal Clean Crawled Corpus)** — 750GB of cleaned Common Crawl text — became one of the most-used public pre-training corpora. T5 itself is released at sizes from 60M to 11B parameters, all open-weight.

The text-to-text framing is the conceptual ancestor of instruction tuning ([[flan-wei-2021|FLAN]], [[instructgpt-ouyang-2022|InstructGPT]]) — once you've decided every task is text-to-text, the next step is "let me write a natural-language instruction telling the model which task." That step ships ~2 years later.

## Key Points

- **Text-to-text framing**: every task becomes `(input text, output text)`. Classification labels, regression numbers, structured outputs all serialized as strings.
- **Architecture**: encoder-decoder Transformer (similar to the original [[transformer]]). 5 sizes from 60M to 11B parameters.
- **Pre-training objective**: **span corruption** (variant of MLM where contiguous spans are masked and the model must predict the masked content as a sequence). Empirically beats both autoregressive LM and standard MLM for this encoder-decoder setup.
- **C4 corpus**: ~750GB / ~156B tokens, derived from Common Crawl with quality filtering and deduplication. Released publicly.
- **Systematic study**: tests pre-training objectives, architectures (encoder-decoder vs. decoder-only vs. encoder-only), unlabeled datasets, transfer approaches (fine-tuning vs. multi-task vs. adapter), and scaling. Hundreds of pages of ablations.
- **State of the art** (at release) on GLUE, SuperGLUE, SQuAD, WMT translation, and several summarization benchmarks.

## Entities Mentioned

- [[colin-raffel]] — co-first author (later at UNC; one of the most-cited NLP researchers of his cohort)
- [[noam-shazeer]] — co-first author (also Transformer paper, MoE, etc.)
- [[google-brain]] — author affiliation
- [[t5]] — the model

## Concepts Discussed

- [[t5]] — the model family
- [[c4]] — the dataset
- [[transformer-architecture]] (encoder-decoder variant)
- [[pre-training]] — span corruption pre-training objective
- [[fine-tuning]] / multi-task learning

## Notable Quotes

> "We explore the landscape of transfer learning techniques for NLP by introducing a unified framework that converts all text-based language problems into a text-to-text format." — Abstract

## References

_Original source: `sources/raffel-t5-2019.md`_
_Code & checkpoints: https://github.com/google-research/text-to-text-transfer-transformer_
