---
title: "Self Improvement via Fast Tree-search (SIFT)"
type: source
created: 2026-09-30
updated: 2026-09-30
sources:
  - self-improvement-via-fast-tree-search-fu-2026.md
arxiv_id: "2609.19526"
authors:
  - Xinghong Fu
  - Aravinth Kulanthaivelu
  - Yutaro Yamada
year: 2026
status: complete
importance: high
tags:
  - 2026
  - paper
---

# Self Improvement via Fast Tree-search (SIFT)

**Xinghong Fu** ([[mit|MIT]]), **Aravinth Kulanthaivelu, Yutaro Yamada** ([[sakana-ai|Sakana AI]]) — [arXiv:2609.19526](https://arxiv.org/abs/2609.19526), v1, cs.AI, 17 Sep 2026. Preprint; no venue.

## Summary

DGM-style self-improving coding agents are expensive because **every candidate self-modification is benchmarked before it can guide search**. SIFT puts a cheap signal in front of that step. Each new child agent is compared pairwise by an [[llm-as-a-judge|LLM judge]] against up to 10 top archive nodes. The judge sees the two agents' **full source files**, never tasks or outcomes. Wins and losses are pooled into a regularized Bradley–Terry strength score. Parents are sampled by rank, P(i) ∝ exp(−α·r_BT − β·r_acc − η·log(1+visits)), with α = β = η = 1. Expansion and evaluation run in a disaggregated, non-blocking pipeline, so promising unevaluated nodes can be expanded speculatively. It is built on the [[darwin-godel-machine]] harness: a 4-task gate, then a fixed 50-task subset (Polyglot-50) as the search signal, with the full 225 tasks held out.

A single pairwise call costs about $0.044 and 0.004 CPU-h with gpt-5.4. A Polyglot-50 evaluation costs about $6.0 and 2.6 CPU-h.

## Key points

- **Polyglot-225, Qwen3-Coder-30B coding model** (Table 2): base agent 20.0; SICA 25.1; DGM 27.1; HGM 30.5 (347 CPU-h); **SIFT 31.1** (Qwen3-480B judge; $34.3, 224 CPU-h, 6.7 h) and **32.0** (gpt-5.4 judge). The budget is capped at 30 expansions to match HGM's 800-evaluation constraint.
- **o3-mini coding model:** base 14.2 → DGM 30.7 (after 80 nodes) → **SIFT 35.1** (gpt-5.4 judge). Three repeat runs landed at 32.0–35.6. With gpt-5-mini as the judge it reaches 31.6, and a no-judge SIFT gets 29.8. Discovered harnesses also lift gpt-5-mini and gpt-5.4-mini.
- **Terminal-Bench 2.1** (gpt-5-mini coder; each figure is the mean of 3 full evaluations): start 29.2%. With the judge, the **judge's #1 pick scores 36.7%** but the **search-accuracy #1 pick scores 28.1%**. The no-judge ablation's best reaches 29.2%, i.e. no gain. Ranking by search accuracy alone would have chosen a node whose "local verifier" was switched off by default and whose bash rewrite silently dropped shell state. The judge caught both defects by reading the code.
- **Judge quality** (Table 5): gpt-5.4-high gives Spearman ρ = 0.72 against full-benchmark score, Recall@5 = 4/5, and pairwise agreement within the top 5 of 1.00. The weaker gpt-5 matches it globally (ρ = 0.71) but falls to **0.50 within the top 5**. Weak judges steer the search, and strong ones are needed for the final pick.
- **Judge input matters more than position debiasing** (Table 6, 50 nodes): diffs only, ρ = 0.40; full files, **ρ = 0.68**. Swap-order passes double the cost and add ~0.
- **SWE-60** (the 60-task SWE-bench Verified subset HGM uses; appendix): start 40.0%. Judge-guided runs reach 52.1% pooled and no-judge runs 47.5%.

> [!note] Abstract/intro vs body
> The intro claims o3-mini runs finish "under 50 CPU hours, 5 hours of wall clock", and the text claims "42 CPU hours and 150 USD" within 30 steps. **Table 3 records that run as 20 steps, $86.8, 59 CPU-h and 2.1 h.** Treat the per-run cost figures as approximate. The Table 2 accuracies are internally consistent.

## What it does not show

- **No matched-compute plain-search baseline.** The comparators are other self-evolution frameworks (DGM, HGM, SICA) at a matched *evaluation* budget, plus a same-budget no-judge ablation. There is no parallel-sampling or best-of-k arm on the base agent, the control [[rethinking-evaluation-harness-evolution-wang-2026]] asks for. Note, though, that on Terminal-Bench the no-judge search itself produced **zero gain** over the starting agent, consistent with that paper.
- In the strongest runs the judge (gpt-5.4) is stronger than the coder, so this is **not pure self-judged improvement**, as the authors state. Each Terminal-Bench condition is a single search run.

## Concepts & entities

- [[self-improving-agent]] · [[darwin-godel-machine]] · [[godel-machine]] — the lineage. SIFT changes only how the tree is searched.
- [[llm-as-a-judge]] — pairwise, rank-only use. Compare the *pre-execution* judge in [[rehearse-confidence-cliff-autoresearch-ji-2026|Rehearse]], which also judges candidates before paying for a run but *requires* both-order agreement. SIFT measured swap-order and found it adds ~0 to rank correlation. Unlike Rehearse, SIFT does not track whether judge accuracy decays with tree depth.
- [[ai-research-preference-models-foster-2026]] — the same move with a trained preference model instead of a prompted judge.
- [[red-queen-godel-machine-iacob-2026]] — cited sibling: a reviewer agent supplies auxiliary signal before full Polyglot evaluation.
- [[cedar-agent-orchestrated-tree-search-complex-systems-tian-2026]] — another Sakana tree search with an LLM judge as value function.
- [[verifier]] · [[reward-hacking]] — the judge caught an off-by-default verifier that accuracy could not see. Patches that edit benchmark or harness code are rejected.
- [[terminal-bench]] · [[swe-bench-verified]] — evaluation suites.

## References

- [arXiv:2609.19526](https://arxiv.org/abs/2609.19526)
- Baselines: DGM ([[darwin-godel-machine-zhang-2025]]), HGM (Wang et al. 2025), SICA (Robeyns et al. 2025).
