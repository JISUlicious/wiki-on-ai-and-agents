---
title: "Follow the Entities: A Corpus Map for Agentic Search (CorpusMap)"
type: source
created: 2026-10-29
updated: 2026-10-29
sources:
  - corpusmap-follow-the-entities-jeong-2026.md
arxiv_id: "2609.37226"
authors:
  - Soyeong Jeong
  - Sujay Kumar Jauhar
  - Sung Ju Hwang
  - Andrew Joohun Nam
year: 2026
status: complete
importance: high
tags:
  - 2026
  - paper
---

# Follow the Entities: A Corpus Map for Agentic Search

**Soyeong Jeong, Sujay Kumar Jauhar, Sung Ju Hwang, Andrew Joohun Nam** (KAIST, [[microsoft|Microsoft]]; first author's work done as a Microsoft intern) — [arXiv:2609.37226](https://arxiv.org/abs/2609.37226), v1, cs.CL, 29 Sep 2026. Preprint.

## Summary

[[agentic-search|Agentic search]] over a flat folder of files makes the agent rediscover, for every query, how documents relate: who owns the project mentioned in this email, which contract covers that vendor. **CorpusMap** builds a navigation layer offline around **recurring entities**. Mentions are extracted per document and resolved against a shared registry. Each resolved entity becomes an **Entity Page** that lists what each document says about it (attributed to the source) and links to every document that mentions it. The result is a bipartite entity–document graph that the agent walks with shell commands; the original documents stay available. Links are built once and shared across queries, and the map can be updated incrementally as documents arrive.

## Key points

- **Setup.** Multi-document questions from EnterpriseRAG-Bench (80 questions, 2,819 docs), WixQA (79, 6,221) and HERB (238, 6,365). The main results use four GPT models (GPT-5.5, GPT-5.6 Luna/Terra/Sol); each model both builds its own layer and answers. The judge is GPT-5.6 Sol; a DeepSeek-V4-Pro re-judge preserves the ranking.
- **Headline.** Against raw-corpus search, Overall Quality is +6.45 / +11.74 / +6.76 / +7.00 points (GPT-5.5 / Luna / Terra / Sol). Average input tokens fall 34–57%, and the one-time construction cost is recovered as the map is reused across queries. All gains are significant (paired bootstrap, Holm-corrected, p < 10⁻⁴).
- **Other navigation layers often make things worse.** For GPT-5.5, Overall Quality is: raw corpus 66.11, Document Page 66.41, Group Page 61.07, **LLM Wiki 64.49**, Corpus2Skill 45.49, **CorpusMap 72.55**. Relative input tokens: 1.00×, 0.94×, 1.65×, 0.63×, 0.31×, 0.43×.
- **No LLM needed to build it.** A version built with GLiNER-based entity linking and deterministic page rendering is about as effective as the LLM-built map and still beats raw corpus on every metric.
- It also holds for DeepSeek-V4-Pro, MAI-Thinking-1 and open-weight Qwen3.8-27B, and as distractor documents are added.

## The "LLM Wiki" baseline — this vault's own pattern

The **LLM Wiki** baseline is [[andrej-karpathy|Karpathy]]'s `llm-wiki` gist, the layout this vault follows (immutable sources, an LLM-maintained cross-linked wiki, a schema file). The paper implements it as "the LLM freely writes cross-linked pages over the corpus". Its result is sobering. CorpusMap beats LLM Wiki by +8.07 / +11.20 / +13.79 / +9.54 points across the four models. Subtracting CorpusMap's gain over raw corpus, **LLM Wiki scores below raw-corpus search for three of four models** (about −1.6, −7.0 and −2.5 points for GPT-5.5, Terra and Sol; about +0.5 for Luna), although it uses fewer tokens (0.63× for GPT-5.5). The authors' explanation: a free-form wiki makes the model decide what becomes a page and how content is merged, and those decisions can degrade as the corpus grows. CorpusMap fixes the page unit (one per resolved entity), attributes every fact to its source document, and always links back to the raw documents.

For this vault, the lessons are concrete. Entity pages that link to every source mentioning them, per-fact source attribution, and keeping raw sources one hop away are what the paper credits for the gain. The setting differs, though: enterprise QA with **one-shot automated wiki builds** by the answering model, not a curated, human-reviewed research wiki.

The newsletter's "Microsoft", "+6.4–11.7 points", "−34–57% tokens" and "beats an LLM Wiki layer" all match. One nuance: the 6.4–11.7 range is over the four GPT models, while the "7 models" includes three more models reported separately.

## What it does not show

- No comparison with graph-RAG retrievers such as GraphRAG or [[hipporag-2|HippoRAG 2]] as navigation layers; they are discussed only in related work.
- Questions are filtered to multi-document ones, which is where an entity map should help most.
- The LLM Wiki baseline is the authors' own implementation of a loosely specified gist pattern. A curated or lint-maintained wiki was not tested.

## Concepts & entities

- [[agentic-search]] · [[retrieval-augmented-generation]] · [[knowledge-graph]] · [[knowledge-graph-extraction]] · [[graph-memory]].
- [[andrej-karpathy]] · [[wfm-wiki-foundation-model-agentic-reasoning-dong-2026]] · [[wikiskill-compiling-agent-experience-tang-2026]] — other papers built on the LLM-wiki pattern.
- [[memorytax-bitter-lesson-agent-memory-peng-2026]] — the opposite finding at small scale: raw search beats designed memory when the history fits in context.
- [[hipporag-2]] · [[bm25]] · [[microsoft]] · [[mai-thinking-1]] · [[qwen]].

## References

- [arXiv:2609.37226](https://arxiv.org/abs/2609.37226)
- [Karpathy, llm-wiki gist](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) — the LLM Wiki baseline
