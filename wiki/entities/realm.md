---
title: REALM
type: entity
created: 2026-10-06
updated: 2026-10-07
sources:
  - realm-guu-2020.md
family: bert
predecessor: "[[bert]]"
successors:
  - "[[dpr]]"
  - "[[rag]]"
builds-on:
  - "[[bert]]"
parameters: 330M
release_date: 2020-02-10
status: complete
importance: high
tags:
  - model
---

# REALM

**Retrieval-Augmented Language Model pre-training**. [[google-research|Google Research]]: Kelvin Guu, Kenton Lee, Zora Tung, Panupong Pasupat, Ming-Wei Chang. [arXiv:2002.08909](https://arxiv.org/abs/2002.08909), 2020-02-10; ICML 2020.

REALM is a 330M-parameter, BERT-based masked language model with a **learned latent retriever** over about 13M Wikipedia blocks. The retriever is trained during **pre-training** with no retrieval labels: the masked-LM loss is backpropagated *through* the choice of document. The same retriever is then used for fine-tuning and inference on open-domain QA.

## How it works

1. **Retrieve:** a BERT dual encoder scores documents by inner product, and MIPS returns the top k. A **null document** is always included for masks that need no lookup.
2. **Predict:** a knowledge-augmented BERT encoder reads [input; document] and predicts the masked span, marginalizing over the retrieved documents.
3. **Keep the index fresh:** document embeddings go stale as the retriever learns, so a parallel job **re-indexes asynchronously** every few hundred steps.
4. **Make retrieval necessary:** **salient span masking** masks entities and dates rather than random tokens, and trivial self-retrieval is prohibited.

## Results

| | NQ | WebQ | CuratedTrec | Params |
|---|---|---|---|---|
| ORQA | 33.3 | 36.4 | 30.1 | 330M |
| T5-11B (closed book) | 34.5 | 37.4 | — | 11B |
| **REALM** | **40.4** | **40.7** | **46.8** | 330M |

REALM outperforms a model 30× larger. The REALM row takes the best of its two pre-training corpora (CC-News for NQ and WebQ, Wikipedia for CuratedTrec).

**Ablations:**
- The improved **retriever** accounts for most of the gain.
- Salient-span masking beats random masks (38.2 vs 32.3 exact match).
- A 30× staler index collapses performance (28.7).

## Lineage

- **Inherited:** BERT's masked-LM objective. From ORQA (Lee et al. 2019), the idea of a learned dense retriever with ICT warm-start.
- **Changed:** retrieval moves into **pre-training**, learned end to end, with the engineering to make that tractable (MIPS plus asynchronous refresh).
- **What came after simplified or scaled it:**
  - [[dpr|DPR]] (Apr 2020): retriever training with plain supervised question–passage pairs, and no expensive retrieval-in-pre-training.
  - [[rag|RAG]] (May 2020): a DPR retriever plus a BART *generator*; coins "retrieval-augmented generation".
  - [[retro|RETRO]] (Dec 2021): retrieval over trillions of tokens with a frozen retriever.
- **Why it matters:** REALM made the case for **non-parametric memory**. Knowledge lives in a swappable corpus rather than only in the weights, which gives interpretability (you see what was retrieved) and modularity. The case runs through [[retrieval-augmented-generation]] to agent [[memory-management]].

## References

- [[realm-guu-2020]]
