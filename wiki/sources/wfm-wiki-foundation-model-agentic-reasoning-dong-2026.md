---
title: "WFM: Wiki Foundation Model for Complex Agentic Reasoning"
type: source
created: 2026-09-30
updated: 2026-09-30
sources:
  - wfm-wiki-foundation-model-agentic-reasoning-dong-2026.md
arxiv_id: "2609.18182"
authors:
  - Junnan Dong
  - Linhao Luo
  - Senlei Zhang
  - Gong Chen
  - Taian Guo
  - Yifei Yu
  - Rong Tao
  - Tao Guo
  - Qian-wen Zhang
  - Siyu An
  - Ruizhi Qiao
  - Xing Sun
year: 2026
tags:
  - 2026
  - paper
status: complete
importance: medium
---

# WFM: Wiki Foundation Model for Complex Agentic Reasoning

**Junnan Dong†, Linhao Luo†, Senlei Zhang†, … Siyu An (corresponding), Ruizhi Qiao, Xing Sun.** Tencent Youtu Lab, Monash, HKBU and Shenzhen University. [arXiv:2609.18182](https://arxiv.org/abs/2609.18182), cs.AI, 16 Sep 2026. The PDF still carries the ACM template placeholders ("Conference acronym 'XX", Woodstock 2018), so no venue is known.

> [!warning] Some reported numbers are self-described as not measured
> §4.4 says: "The newly completed cells and diagnostic sweeps are constructed planning values and require validation with measured runs before external use." The paper does not say which cells or sweeps this covers. Figure 4 is labelled "using reported results". **Treat every number below as provisional.** This is the paper's own caveat, not an inference.

## Summary

WFM cites [[andrej-karpathy|Karpathy]]'s `llm-wiki` gist as the target representation: dense text coupled with linked entity structure, the same pattern as this vault. It argues that such wikis should be *encoded by a trained graph foundation model* rather than retrieved training-free. The "Wiki Graph" is in fact a hybrid of **entity nodes** (typed entity–relation edges) and **passage nodes** (text-encoder embeddings), joined by cross-layer entity→passage links. It is *not* a set of markdown pages linked to each other, and the paper never describes how the wiki is built from its QA corpora. A 3-layer relation-aware attention GNN propagates over the query-induced subgraph. Training combines a TransE ranking loss, InfoNCE entity–passage alignment and a hinge **attention-variance regularizer** against "uniform attention collapse", with a warm-start curriculum. Inference wraps retrieval in a **self-reflection loop**: up to B = 4 rounds, accumulating evidence, with an early-stop flag. A systems contribution, NCCL-native boundary exchange, cuts training step time from 2.40 s to 0.23 s (10.5×).

## Key points

- **Multi-hop QA (Table 2; 1,000 questions each, DeepSeek-V4-Flash generator, V4-Pro judge).** Open/Reject accuracy is HotpotQA **89.6/84.3**, 2Wiki **90.2/82.4** and MuSiQue **69.8/52.6**. That is +2.8/4.1, +3.2/4.8 and +4.1/5.1 over Youtu-GraphRAG, from the same lab. [[hipporag-2|HippoRAG 2]] scores 86.6/74.4, 84.5/76.5 and 62.4/38.9.
- **Recall@20 (Table 1).** HotpotQA 93.20 and 2Wiki 90.15. On MuSiQue it gets 75.24 against Youtu-GraphRAG's 75.90, so WFM is *not* best there. Its margin over GFM-RAG is +11.8/+17.5/+26.7.
- **Long-horizon memory (Table 3).** PersonaMem-1M overall is **58.49** (A-mem 50.0, MemoryOS 50.1) and RHELM **52.17** (A-mem 37.8). The body says WFM wins "every category". Without reflection it scores 54.12 / 48.90.
- **Ablations (Fig. 5).** Removing passage nodes to leave an entity-only "ordinary graph" costs **−7.48 recall@20 and −7.68 memory accuracy**. Removing self-reflection takes memory accuracy from 55.33 to 51.51 at 0.69× cost. Forcing all 4 rounds costs 1.58×. The average number of executed rounds at B = 4 is 2.58.

## Is the comparison budget-matched?

**No, only partly.** All methods share the embedding model (all-MiniLM-L6-v2), the retrieval depth of 20 and the generator. WFM, however, runs an *iterative* loop that **accumulates** top-20 sets over an average of 2.58 rounds, with up to 4 LLM calls. Every baseline except HippoRAG-IRCoT is single-pass, and no LLM-call, token or latency budget is equalised. Cost is reported only relative to WFM's own variants. The no-reflection variant still leads, which is the closest thing to a matched comparison, but its margin is 4.02 / 2.10 points rather than 8.39 / 5.37. This does **not** contradict [[graph-memory]]'s budget-matched loss for *extraction-based* graphs. WFM keeps passages as first-class nodes, like HippoRAG 2, and its own entity-only ablation *loses* 7–8 points. That is a second data point for that page's synthesis: "indexing text with entities is fine; replacing it is not."

## Concepts & entities

- [[andrej-karpathy]] — the cited LLM-wiki pattern. [[wikiskill-compiling-agent-experience-tang-2026]] is the other paper here built on that gist.
- [[graph-memory]] · [[knowledge-graph]] · [[hipporag-2]] · [[hipporag2-jimenez-gutierrez-2025]] — the graph-retrieval lineage and its main baseline.
- [[long-term-memory]] · [[a-mem]] · [[a-mem-xu-2025]] — the memory baselines.
- [[retrieval-augmented-generation]] · [[self-reflection]] — the Native-RAG baseline and the iterative loop.
- [[selective-forgetting-graph-based-memory-rusu-2026]] — the budget-matched counter-evidence discussed above.

## What it does not show

- **Some cells are unmeasured planning values** (see the warning above). There are no seeds or variance.
- Wiki construction from HotpotQA, 2Wiki, MuSiQue and the memory histories is never specified. It is unclear who writes the "wiki", or whether it resembles a human- or LLM-curated markdown vault at all.
- On RHELM the full-context zero-shot baseline (46.8) beats every retrieval baseline, and Native RAG manages only 28.7. The baselines look weak in this setup, which inflates the margins.

## References

- [arXiv:2609.18182](https://arxiv.org/abs/2609.18182)
