---
title: "REALM: Retrieval-Augmented Language Model Pre-Training"
type: source
created: 2026-10-06
updated: 2026-10-07
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

## Method details

- **Retrieve-then-predict:** p(y|x) = Σ_z p(y|z,x)·p(z|x). There are two parts:
  - a **neural knowledge retriever**: a BERT dual encoder with inner-product relevance;
  - a **knowledge-augmented encoder**: BERT over [x; z].
- **Tractability:** a **MIPS** index over about 13M Wikipedia blocks gives the top-k documents. The index goes stale as the retriever trains, so it is **refreshed asynchronously** every few hundred steps by a separate re-indexing job.
- **Three pre-training tricks:**
  - **Salient span masking:** mask named entities and dates, such as "United Kingdom" or "July 1969", so that predicting them needs world knowledge.
  - A **null document**, so the model can assign credit when no retrieval is needed.
  - **Prohibiting trivial retrievals:** never retrieve the masked sentence's own source document.
- **Warm start:** the retriever starts from ICT (Inverse Cloze Task) pre-training, and the encoder from BERT.

## Results (Table 1, exact match)

| System | Params | NQ | WebQ | CuratedTrec |
|---|---|---|---|---|
| BERT baseline (sparse retrieval) | 110M | 26.5 | 17.7 | 21.3 |
| T5-11B (closed book) | 11,318M | 34.5 | 37.4 | — |
| ORQA (dense retrieval) | 330M | 33.3 | 36.4 | 30.1 |
| **REALM** (X = Wikipedia) | 330M | 39.2 | 40.2 | **46.8** |
| **REALM** (X = CC-News) | 330M | **40.4** | **40.7** | 42.9 |

REALM beats the closed-book T5-11B "while being 30 times smaller". Its gain over ORQA is "purely due to better pre-training", since the fine-tuning setup is the same.

## Ablations (Table 2, NQ dev: exact match / zero-shot retrieval recall@5)

| Setting | EM | R@5 |
|---|---|---|
| REALM | 38.2 | 38.5 |
| REALM retriever + baseline encoder | 37.4 | 38.5 |
| Baseline retriever + REALM encoder | 35.3 | 13.9 |
| Baseline (ORQA) | 31.3 | 13.9 |
| Random uniform masks | 32.3 | 24.2 |
| Random span masks | 35.3 | 26.1 |
| 30× stale MIPS index | 28.7 | 15.1 |

**Takeaways:**
- Most of the gain comes from the **better retriever**.
- **Salient span masking** matters a lot.
- A **stale index** hurts badly, which justifies the refresh machinery.

The paper also claims qualitative benefits: **interpretability** (you can see what was retrieved) and **modularity** (the knowledge corpus can be swapped).

## Lineage

- **Built on:** BERT-style masked language modelling, and ORQA (Lee et al., 2019), which had a learned retriever but no retrieval-augmented pre-training.
- **Fed into:**
  - [[dpr|DPR]] (2020-04-10), which simplified retriever training to supervised question–passage pairs, with no retrieval-augmented pre-training.
  - [[rag|RAG]] (2020-05-22), which couples a DPR retriever to a seq2seq generator and coined the term *retrieval-augmented generation*. REALM is the extractive, pre-training-time ancestor.
  - [[retro|RETRO]] (2021-12-08), which scaled retrieval to trillions of tokens during pre-training.
- **For the lineage graph:** REALM is the earliest of the four retrieval papers in this wiki. The order is REALM (Feb 2020) → DPR (Apr 2020) → RAG (May 2020) → RETRO (Dec 2021).

## Concepts & entities

[[realm]] · [[retrieval-augmented-generation]] · [[dpr]] · [[rag]] · [[retro]] · [[bert]] · [[pre-training]] · [[google-research]]
