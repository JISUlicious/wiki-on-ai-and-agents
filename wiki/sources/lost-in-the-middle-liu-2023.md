---
title: "Lost in the Middle: How Language Models Use Long Contexts"
type: source
created: 2026-10-06
updated: 2026-10-06
sources:
  - lost-in-the-middle-liu-2023.md
arxiv_id: "2307.03172"
venue: "TACL 2024 (arXiv preprint July 2023)"
authors:
  - Nelson F. Liu
  - Kevin Lin
  - John Hewitt
  - Ashwin Paranjape
  - Michele Bevilacqua
  - Fabio Petroni
  - "[[percy-liang]]"
first_author: Nelson F. Liu
introduces:
  - "[[context-rot]]"
year: 2023
tags:
  - 2023
  - paper
status: complete
importance: high
---

# Lost in the Middle: How Language Models Use Long Contexts

**Source**: `sources/lost-in-the-middle-liu-2023.md` ([arXiv:2307.03172](https://arxiv.org/abs/2307.03172))
**Authors**: Nelson F. Liu, Kevin Lin, John Hewitt, Ashwin Paranjape, Michele Bevilacqua, Fabio Petroni, [[percy-liang|Percy Liang]] ([[stanford-university|Stanford]], [[uc-berkeley|UC Berkeley]], Samaya AI)
**Date**: arXiv v1 July 2023 (from the arXiv ID); v3 dated 2023-11-20. Published in *TACL* in 2024.

> [!note] Dating
> Pages in this wiki cite it as "Liu et al., TACL 2024", which is correct for the journal. For lineage, the finding dates from **July 2023**.

## Summary

The paper asks whether models that *accept* long contexts actually *use* them. It tests two tasks:
- **multi-document QA**: one answer-bearing passage among k−1 distractors, which mirrors a RAG setup;
- **synthetic key-value retrieval** over JSON.

Models tested: MPT-30B-Instruct, LongChat-13B (16K), GPT-3.5-Turbo (4K and 16K), Claude-1.3 (8K and 100K).

## Key findings

- **The U-shaped curve.** Accuracy is highest when the relevant information comes at the **beginning** (primacy bias) or **end** (recency bias), and drops sharply in the **middle**.
- **Worse than no documents.** With the answer in the middle of the context, GPT-3.5-Turbo's multi-document QA accuracy falls *below* its closed-book score of 56.1%.
- **Longer windows don't help.** Extended-context models often score the same as their shorter-context versions.
- **Plain lookup degrades too.** Even simple key-value retrieval shows the U-shape for some models.
- **Factors studied:**
  - Encoder-decoder models are robust to position only within their training length.
  - *Query-aware contextualization* (putting the query both before and after the documents) nearly fixes key-value retrieval but barely helps multi-document QA.
  - Instruction tuning is not the cause: base models show the U-shape too.
- **The RAG case study** (NaturalQuestions-Open): "model performance saturates long before retriever recall saturates." Going from 20 to 50 retrieved documents improved accuracy only marginally (about 1.5% for GPT-3.5-Turbo and about 1% for Claude-1.3).

## Why it matters for the wiki

This is the standard citation for the finding that an advertised context length does not mean effective context. Its lineage:
- [[needle-in-a-haystack]]-style evaluation and Gemini 1.5's long-context claims ([[gemini-1-5-google-2024]], Feb 2024).
- "Context rot" (Chroma, 2025) and Anthropic's "attention budget" framing in [[effective-context-engineering-anthropic-2025]]. See [[context-rot]].
- The 2026 compaction-loss literature, which finds a *monotonic* pattern rather than a U-shape. See [[lost-in-compaction-side-constraint-loss-wang-2026]].
- The practical rules of [[context-engineering]]: put critical instructions at the edges and retrieve fewer, better documents.

## Concepts & entities

[[long-context-llm]] · [[context-rot]] · [[context-engineering]] · [[retrieval-augmented-generation]] · [[needle-in-a-haystack]] · [[percy-liang]] · [[stanford-university]]
