---
title: "REALM: Retrieval-Augmented Language Model Pre-Training"
type: source
created: 2026-10-06
updated: 2026-10-06
sources:
  - realm-guu-2020.md
arxiv_id: "2002.08909"
venue: "ICML 2020"
authors:
  - Kelvin Guu
  - Kenton Lee
  - Zora Tung
  - Panupong Pasupat
  - Ming-Wei Chang
first_author: Kelvin Guu
introduces:
  - "[[realm]]"
year: 2020
tags:
  - 2020
  - paper
status: complete
importance: high
---

# REALM: Retrieval-Augmented Language Model Pre-Training

**Source**: `sources/realm-guu-2020.md` ([arXiv:2002.08909](https://arxiv.org/abs/2002.08909))
**Authors**: Kelvin Guu, Kenton Lee, Zora Tung, Panupong Pasupat, Ming-Wei Chang ([[google-research|Google Research]])
**Date**: **2020-02-10** (arXiv v1). The ICML 2020 venue is from general knowledge.

## Summary

Pre-trained models like [[bert|BERT]] and [[t5|T5]] store world knowledge *implicitly in their parameters*, so covering more facts means ever-larger networks. **[[realm|REALM]]** instead adds a **latent knowledge retriever** to pre-training:
- before predicting a masked token, the model retrieves documents from a large corpus (Wikipedia) and attends over them;
- the retriever is used during pre-training, fine-tuning and inference.

**The first-time claim:** REALM shows how to **pre-train the retriever without supervision**. The masked-language-modelling signal is backpropagated *through a retrieval step over millions of documents*. MIPS search and an asynchronously refreshed document index make this tractable.

## Results

- On three Open-QA benchmarks (NaturalQuestions-Open, WebQuestions, CuratedTrec), REALM beats all previous methods by **4–16 points of absolute accuracy**. This includes ORQA, its closest predecessor, and the much larger implicit-knowledge T5.
- The authors also claim qualitative benefits: **interpretability** (you can see what was retrieved) and **modularity** (the knowledge corpus can be swapped).

## Lineage

- **Built on:** BERT-style masked language modelling, and ORQA (Lee et al., 2019), which had a learned retriever but no retrieval-augmented pre-training.
- **Fed into:**
  - [[dpr|DPR]] (2020-04-10), which simplified retriever training to supervised question–passage pairs and sits in REALM's comparison table.
  - [[rag|RAG]] (2020-05-22), which couples a DPR retriever to a seq2seq generator and coined the term *retrieval-augmented generation*. REALM is the extractive, pre-training-time ancestor.
  - [[retro|RETRO]] (2021-12-08), which scaled retrieval to trillions of tokens during pre-training.
- **For the lineage graph:** REALM is the earliest of the four retrieval papers in this wiki. The order is REALM (Feb 2020) → DPR (Apr 2020) → RAG (May 2020) → RETRO (Dec 2021).

## Concepts & entities

[[realm]] · [[retrieval-augmented-generation]] · [[dpr]] · [[rag]] · [[retro]] · [[bert]] · [[pre-training]] · [[google-research]]
