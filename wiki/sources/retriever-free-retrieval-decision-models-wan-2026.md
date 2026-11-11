---
title: "Retriever-Free Retrieval: Navigating Documents with Decision Models"
type: source
created: 2026-11-10
updated: 2026-11-11
sources:
  - retriever-free-retrieval-decision-models-wan-2026.md
arxiv_id: "2610.35868"
authors:
  - Chao Wan
  - Paul Jünger
  - Beixi Du
  - Bodong Liu
  - Navaneetha Krishnan
  - Kilian Q. Weinberger
year: 2026
venue: Preprint (arXiv cs.AI)
status: complete
importance: high
tags:
  - 2026
  - paper
---

# Retriever-Free Retrieval: Navigating Documents with Decision Models

**Source:** Chao Wan, Paul Jünger (equal contribution), Beixi Du, Bodong Liu, Navaneetha Krishnan, Kilian Q. Weinberger (Cornell University; ASAPP) — [arXiv:2610.35868](https://arxiv.org/abs/2610.35868), cs.AI, v1 26 Oct 2026. Preprint.

## Summary

The paper names a **capability asymmetry** in [[retrieval-augmented-generation|RAG]]: a frontier generator still sees only what a BM25 or embedding retriever found. **DecisionRAG** replaces the first-stage retriever with a [[system-one-model|decision model]] that *navigates* the corpus's own structure (topics → documents → sections → passages). At each level it scores all candidate branches in one shared context and one forward pass, with no generated tokens, and keeps the most promising ones. The default model is [[jev|Jev]] 1.13 with no task-specific training. A final multiple-choice step (up to 255 options for Jev) reorders the top 20 passages.

## Key points

- **Beats dense + 8B reranker without training.** Against Qwen3-Embedding-8B retrieval with an 8B reranker: +13.8 R@1 on MultiDoc2Dial and +17.4 Hit@1 on Natural Questions. The four benchmarks are MultiDoc2Dial, ConditionalQA, MTRAG and NQ over 2.68M Wikipedia paragraphs.
- **Shared context makes scores comparable.** Judging candidates together rather than one at a time adds up to 7.7 points R@1. A candidate that looks relevant alone gets a lower P(yes) when a better one sits beside it. Hit@1 rises from 47.8% (each scored alone) to 54.3% at 50 candidates per request and then plateaus, so large pools can be split into parallel 50-candidate requests.
- **Multiple choice sharpens the top rank.** Ranking all 200 candidates as one multiple-choice question adds 6.8 Hit@1 over binary judgments in one request, for 0.12–0.15 s and 4–33% more API cost.
- **Fast and cheap.** 0.7–1.8 s and 0.2–2.3¢ per query, against 3.3–14.3 s for dense + Qwen3-Reranker-8B on one GPU and 11.8–15.0 s for GPT-5 reranking.
- **Decisions vs. generation over the same hierarchy.** GPT-5 generating navigation choices token by token gives no consistent accuracy gain (+4.1 on ConditionalQA, −6.0 on NQ). It is 34–145× slower and 21–25× more expensive.
- **Navigation beats reranking.** Using Jev only to rerank the dense top 100 is 7.7 (ConditionalQA) and 7.0 (NQ) Hit@1 below navigating the hierarchy. So the gain is not only the decision model.
- **Budget trade-off.** Keeping about 15% of 1,000 top-level NQ clusters matches flat scoring of all 108,626 page titles (R@10 86.2 vs 84.5) at one fifth of the cost and one third of the latency. Below 5% the gold cluster is often pruned and R@10 drops to 63.7% at 1%.
- **Headroom with training.** A dedicated 8B decision model trained in-domain reaches 60.7 R@1 on MultiDoc2Dial, 5.7 above the best published task-trained retriever compared.

## Method details

- **Beam, not a single path.** Algorithm 1 keeps the top `kℓ` candidates per level by P(yes), expands their children, and repeats. Passages are then cut to the top r, and optionally the top m (20; 5 on NQ) are reordered by one multiple-choice question. A budget can also be a probability rule (keep the runner-up when the top score is below 0.8).
- **Yes/no inside the tree, multiple choice at the end.** Yes/no has no option limit, splits into parallel requests and favours recall. Multiple choice is used once, to sharpen the top rank.
- **Latency and cost.** Latency ≈ 0.2 s × ⌈log_b N⌉ (about 1 s for 10⁹ passages at b = 100). Cost grows with the number of judgments, at most k × b per level.
- **Split requests.** Pooled scores from separate requests are fine for screening (R@10 unchanged) but lower Hit@1. Gains from shared context stop at about 50 candidates per request.
- **Node representation.**
  - **Passages:** shown with their page and heading path.
  - **Pages:** shown by title and section headings.
  - **NQ clusters:** one ≤30-word sentence written by Qwen3-8B from up to 300 page titles.
  - **Structure:** taken from corpus metadata or HTML headings, or built by recursive k-means when absent. The tree need not be uniform: MTRAG adds a heading level only under pages with more than 60 passages.
- **Per-benchmark budgets.**
  - **ConditionalQA:** keep 3 of 652 guides.
  - **MTRAG:** 8,578 page titles → top 50 by pooled score → rescored together → 10.
  - **NQ:** 150 of 1,000 clusters → about 17,500 titles → paragraphs of the top 30 pages (about 230). Paragraph score is multiplied by page score.
- **Generator test.** Qwen3.5-27B (thinking off, greedy) answers from each system's top 5 passages, numbered in rank order, plus the query. No upper-level choices or scores are passed to it.
- **Prior hierarchical retrieval it cites:** MemWalker, PageIndex, LATTICE (LLM-generated branch choices over a corpus tree), RAPTOR, DocQN, dense hierarchical retrieval.

The paper never discusses graphs versus trees. For that discussion, and for what reaches the generator, see [[how-decisionrag-navigates-a-corpus-hierarchy]].

## Limitations

Quality depends on a corpus hierarchy with informative node descriptions. Pruning is irreversible: a passage whose ancestor is discarded can never be found. Larger budgets and adding dense candidates mitigate this. Most numbers rest on one proprietary model version.

## Concepts & entities

- [[system-one-model]] · [[jev]] · [[typesafe-ai]] · [[logit-scoring]] — the prefill-only decision interface used as the retriever.
- [[openai-decisions-api]] — cited with Jev as the other decision-model provider.
- [[retrieval-augmented-generation]] · [[dense-retrieval]] · [[bm25]] · [[open-domain-qa]] — the baselines being replaced.
- [[agentic-search]] — the token-generating navigation alternative this paper prices out.
- [[calibration]] — the shared-context effect is framed as calibrating relevance across candidates.
- Related: [[system-one-models-are-not-better-rerankers-dinzinger-2026]] (decision fine-tunes as rerankers).

## References

- [arXiv:2610.35868](https://arxiv.org/abs/2610.35868)
