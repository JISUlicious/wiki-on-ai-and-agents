---
title: "STAIR (STructure Aware Information Retriever): A novel dataset and LLM based retriever for document structure augmentation"
type: source
created: 2026-09-15
updated: 2026-09-15
sources:
  - stair-structure-aware-retriever-kumar-2026.md
arxiv_id: "2609.03874"
authors:
  - Vineet Kumar
  - Meghanadh Pulivarthi
  - Vishwajeet Kumar
  - Jaydeep Sen
  - Riyaz Ahmad Bhat
  - Sachindra Joshi
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# STAIR: Structure-Aware Information Retriever

**Vineet Kumar, Meghanadh Pulivarthi, Vishwajeet Kumar, Jaydeep Sen, Riyaz Ahmad Bhat, Sachindra Joshi** (IBM; first author now at Amazon Books Science). [arXiv:2609.03874](https://arxiv.org/abs/2609.03874), cs.AI, v1 3 Sep 2026. No venue stated. The code link is an anonymous 4open.science repo, which suggests the paper is under review.

> [!important] What this paper is, in one paragraph
> STAIR is a **generative retriever**: it stores a book in model weights and answers a query by *writing out* the right section title. It is a variant of the Differentiable Search Index (DSI, Tay et al. 2022; no wiki page). The one change is that the **whole Table of Contents goes in the prompt** during fine-tuning and inference (up to 14k input tokens). The model then picks a ToC leaf instead of having to memorise the book's structure from query–label pairs. On the new **SearchTome** benchmark, STAIR gets **82.6% Recall@1** against 76.9% for fine-tuned DSI. The pitch is that ToC sections are semantically coherent retrieval units, unlike the length-based chunks of standard [[retrieval-augmented-generation]] pipelines.

## Setup

- **SearchTome:** 18 open textbooks, 3 each from 6 domains (Education, Finance, Law, Medicine, Natural Sciences, Social Sciences). The books run 179–1,327 pages, with 70–429 ToC leaves each (2,880 in total). Mixtral-8x7B generates questions for each paragraph, and the gold label is the leaf the paragraph belongs to. Summed from Table 1: 62,458 train / 24,502 dev / 103,249 test queries. Most generated questions go to the test split on purpose, to cover the whole book.
- **Model:** Mistral-7B-Instruct v0.2, [[lora]] r=16, α=32, up to 200 epochs with early stopping at patience 20 (dev R@1). Max input is 14k tokens for STAIR and 512 for DSI; max output is 64. STAIR and DSI are both fine-tuned **per book**. Inference uses [[constrained-decoding]] restricted to valid ToC leaves (Fig. 2).
- **Baselines:** out-of-the-box Mistral with beam search; [[bm25]] on Elasticsearch 8.11.2; "DPR", which is actually **NV-Embed-v2 used zero-shot**, a Mistral-7B-based embedder, not Karpukhin's [[dpr]]; Claude Haiku 4.5, which appears only in Table 4; and fine-tuned DSI.

## Key results (Table 4, R@1 / R@3 / nDCG@3, macro-averaged over the 6 domains)

| System | R@1 | R@3 | nDCG@3 |
|---|---|---|---|
| Mistral (no tuning) | 13.8 | 16.4 | 15.4 |
| Claude Haiku 4.5 | 45.6 | 64.1 | 56.4 |
| BM25 | 59.5 | 77.6 | 70.1 |
| DPR (NV-Embed-v2) | 68.7 | 85.4 | 78.6 |
| DSI (fine-tuned) | 76.9 | 85.3 | 81.9 |
| **STAIR** | **82.6** | **90.8** | **87.5** |

- STAIR is best in every domain, with R@1 from 80.6 (NatSci) to 86.1 (Med). The headline "+7.4%" is a **relative** gain; the absolute gap to DSI is 5.7 points. A randomization test (Smucker et al. 2007) finds the DSI–STAIR gap significant at p < 0.05 in each of the 6 domains.
- **Hallucination** here means generating a string that is not a valid ToC leaf. STAIR's error rate is 18.67%, and only **0.05%** of its predictions are non-leaf. DSI's error rate is 24.31% with 3.25% non-leaf; untuned Mistral's is 86.20%, and 26.81% of its mistakes are non-leaf. DSI hallucinates more, and loses more R@1 to STAIR, on leaves with few training queries (Figs. 3–4). The authors read this as DSI having to memorise the book structure that STAIR is simply given.

## What it does not show

The queries are synthetic and in-distribution: one LLM wrote the train and test questions from the same paragraphs, and the index is one book at a time. So this is closer to "memorise this book" than to open-corpus retrieval. The comparison is also uneven. STAIR and DSI are fine-tuned per book, while the dense baseline is a zero-shot embedder, and nobody tests a fine-tuned [[dense-retrieval]] model or ToC-aware chunking. The hallucination headline is partly built into the method, because inference is constrained to leaf strings. The paper also doesn't say whether DSI got the same constraint, and the abstract's "less than 0.05%" is written as "only 0.05%" in the body. There is no end-to-end RAG answer-quality result and no cost for 14k-token prompts. The authors admit the method needs a corpus that already has a ToC. Their own Conclusion even lists generating a ToC-like structure dynamically for an unseen corpus as future work.

## Related

- [[retrieval-augmented-generation]] · [[rag]] · [[rag-lewis-2020]]: the downstream setting. The paper argues that fixed-length chunking breaks semantic boundaries (the wiki has no chunking page).
- [[dense-retrieval]] · [[dpr]] · [[dpr-karpukhin-2020]] · [[bm25]]: baseline families. The table's "DPR" row is an NV-Embed-v2 stand-in.
- [[hallucination]] · [[constrained-decoding]]: the <0.05% invalid-identifier rate is a narrow, retrieval-side definition of hallucination.
- [[long-context-llm]] · [[longbench]]: "lost in the middle" is the stated motivation for precise retrieval over just using long context.
- [[lora]] · [[fine-tuning]] · [[understanding-lora-as-knowledge-memory-back-2026]]: STAIR stores the corpus in a LoRA adapter, like DSI's parametric index.
- [[agentic-search]]: the authors' stated next step is iterative, zero-shot ToC navigation for multi-hop agentic retrieval.
- [[claude]]: Claude Haiku 4.5 is a baseline (45.6 R@1, below BM25).

## References

- [arXiv:2609.03874](https://arxiv.org/abs/2609.03874) (v1, 3 Sep 2026).
- DSI: Tay et al. 2022; model-based indexing: Metzler et al. 2021. RAPTOR: Sarthi et al. 2024, a dense-retrieval tree, contrasted with STAIR's model-based index.
- DPR: Karpukhin et al. 2020; ColBERT: Khattab & Zaharia 2020; SPLADE v2: Formal et al. 2021; BM25: Robertson & Zaragoza 2009; NV-Embed: Lee et al. 2024.
- Lost in the Middle: Liu et al., TACL 2024; LongBench: Bai et al. 2024; LoRA: Hu et al. 2021; Mistral 7B: Jiang et al. 2023; Mixtral: Jiang et al. 2024.
- Significance test: Smucker et al. 2007. Long-context retrieval benchmark LoCoV1: Saad-Falcon et al. 2024.
