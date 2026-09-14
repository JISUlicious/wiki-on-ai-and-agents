---
title: "Codebook Agent: Amortized Topology Design for LLM Multi-Agent Systems"
type: source
created: 2026-09-15
updated: 2026-09-15
sources:
  - codebook-agent-amortized-topology-design-yu-2026.md
arxiv_id: "2609.02264"
authors:
  - Jinxi Yu
  - Yubei Li
  - Eric Hanchen Jiang
  - Zhi Zhang
  - Dong Liu
  - Wenxiao Zhao
  - Levina Li
  - Kai-Wei Chang
  - Ying Nian Wu
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# Codebook Agent: Amortized Topology Design for LLM Multi-Agent Systems

**Jinxi Yu\*, Yubei Li\*, Eric Hanchen Jiang\*, Zhi Zhang, Dong Liu, Wenxiao Zhao, Levina Li, Kai-Wei Chang, Ying Nian Wu** (\*equal contribution; all UCLA). [arXiv:2609.02264](https://arxiv.org/abs/2609.02264), cs.AI, v1 2 Sep 2026. No venue stated. Code: `github.com/jinxiy1104/CodebookAgent`. Jiang, Chang and Wu are also authors of GTD, the strongest baseline.

## Summary

Query-conditioned topology designers for [[multi-agent]] LLM teams (G-Designer, ARG-Designer, GTD) generate an N×N communication adjacency with a variational, autoregressive or diffusion decoder. They then rank the samples with a GNN proxy trained on utility plus edge count |E|. The paper argues that three measurements break this recipe: (1) successful topologies collapse to **about six codes** even as codebook capacity grows from 8 to 64; (2) **edge count correlates negatively with measured tokens** (r ≈ −0.4); (3) on homogeneous teams (identical agent profiles, the benchmark default) a message-passing scorer is **adjacency-invariant**, giving every candidate the same score. Codebook Agent therefore replaces generation with selection. A VQ autoencoder compresses successful topologies (u > 0.5) into a query-independent 16-entry codebook, and a reward-weighted MLP maps the query embedding to a distribution over codes. The top 5 codes are decoded, deduplicated and scored in one batched pass by an MLP over the flattened adjacency, regressed on measured utility and per-task-normalized token cost; the winner maximizes û − 0.1·ĉ. Design makes no LLM calls and does no message passing. Training data per benchmark is 50 tasks × 6 fixed topologies (complete, chain, star, three Erdős–Rényi samples) = 300 executed records.

## Key results

- **Accuracy** (Table 1; gpt-4o-mini, 4-agent homogeneous teams, 14 baselines). Codebook Agent is best on all six benchmarks: **84.62 average** vs 83.02 for GTD (+1.60), 80.18 for DyLAN and 77.63 for Vanilla. The margins over GTD are small and spread out: +1.3 GSM8K, +1.0 MATH, +1.2 MultiArith, +2.4 SVAMP, +3.1 MBPP, +0.6 HumanEval. On the Qwen-3-8B transfer (GSM8K/MATH/MMLU) it scores 74.0 vs 72.7 (GTD) and 72.1 (G-Designer).
- **Latency.** 2.4–2.5 ms per topology vs 301–396 ms for iterative generators (125–158×). Topology design was already under 0.1% of end-to-end latency, though, so this is not where the savings come from.
- **Tokens.** 21.9–33.2% fewer than the full incumbent pipeline (iterative decoder + edge-count GNN): GSM8K 1,239→927, MATH 2,304→1,611, homogeneous HumanEval 699→546, heterogeneous HumanEval 624→417. MATH wall clock drops from 43.7 to 27.4 min.
- **The scorer is where the savings come from** (Fig. 5c, same candidate set). Tokens per query on GSM8K/HumanEval: MLP 927/546, random 1,249/750, edge-count GNN 1,711/918, with accuracy within 1.5 points across all three. The incumbent GNN is **worse than random selection**: its adjacency-invariant utility head gives every candidate the same score, which leaves the |E| head to steer toward expensive graphs. The one exception is the heterogeneous HumanEval team, where the GNN wins on accuracy (78.8 vs 78.1).
- **The design space is small.** Growing K from 4 to 64 moves accuracy by at most 1.5 points (GSM8K) and 2.5 (HumanEval), and for K ≥ 8 no more than six codes are ever used. Across seven settings the best *fixed* topology stays within 1.4 points of every generated one, and on MATH the fully connected graph beats every generated configuration. Which fixed graph wins varies: fully connected on MATH, star on homogeneous HumanEval, chain on MMLU. The gaps between them reach 4.3 points and 2.9× in tokens.

## Sparser graphs cost more tokens

> [!warning] Contradiction
> The learned-topology literature treats edge count as the cost to minimize. GPTSwarm, AgentPrune ("Cut the Crap", Zhang et al. 2025b), G-Designer and GTD all train or rank with an |E| cost term, and pruning methods such as AgentPrune and AgentDropout sparsify graphs specifically to save tokens. This paper measures edge count against actual token spend and finds **r ≈ −0.4: fewer edges, more tokens**. The paper's explanation is that sparse communication yields longer completions. The chain, the sparsest connected graph, is the most token-expensive fixed topology on every math-format benchmark. The evidence has limits. It is one pooled correlation over the 300 training records per benchmark, where |E| takes only about six values (six graph families). There is no per-benchmark r, no confidence interval, and every run is single, on 4-agent gpt-4o-mini teams. It does not show that pruning an already-dense graph never saves tokens. It is also a different axis from [[when-agents-coordinate-destefanis-2026]], where messages per run grow from 6.1 to 71.3 as teams grow from 2 to 8 agents. That result varies team size; this one varies edges at fixed N.

## What it does not show

The codebook is trained on records from exactly six fixed topologies. The ER graphs at eight densities are used only for an auxiliary proxy head. "Six used codes" is therefore also the number of distinct training graphs, and the paper does not show that a richer candidate pool would collapse the same way. In practice, Codebook Agent selects among those six graphs per query, which is the paper's own "short list" framing. Because the best fixed graph sits within 1.4 points of every generated one, the main table is missing its most natural baseline: the best fixed topology for each benchmark. Every result is a single run with no confidence intervals. The benchmarks are math and code only. Teams are homogeneous with up to 10 agents, apart from one heterogeneous setting, and the backbones are gpt-4o-mini and Qwen-3-8B.

## Related

- [[multi-agent]] · [[learned-orchestration]] · [[agent-as-a-router-zhou-2026]]: per-query structure selection. Codebook Agent is the topology-level case, turning the decision into one cheap learned selector.
- [[when-agents-coordinate-destefanis-2026]] · [[drop-the-hierarchy-and-roles-dochkina-2026]] · [[brain-inspired-graph-multi-agent-systems-hao-2026]] · [[hyperagents-zhang-2026]]: communication structure that is measured, emergent or designed.
- [[single-agent-vs-multi-agent-tran-2026]] · [[price-reversal-phenomenon-chen-2026]] · [[test-time-compute]]: measured tokens, not a nominal proxy, as the real cost unit.
- [[adas]] · [[autogen]] · [[metagpt]] · [[chatdev]]: learned-design baselines and fixed-topology frameworks cited.
- [[chain-of-thought]] · [[self-consistency]]: the single-agent baselines.
- [[humaneval]] · [[mmlu]] · [[qwen]]: benchmarks and models with pages. GSM8K, MATH, MBPP and gpt-4o-mini have none.

## References

- [arXiv:2609.02264](https://arxiv.org/abs/2609.02264) (v1, 2 Sep 2026).
- Topology designers: GPTSwarm (Zhuge et al., ICML 2024); G-Designer (Zhang et al., ICML 2025); AgentPrune / "Cut the Crap" (Zhang et al., ICLR 2025); AgentDropout (Wang et al., ACL 2025); ARG-Designer (Li et al., AAAI 2026); TopoDIM (Sun et al., ACL Findings 2026); GTD (Jiang et al., ACL 2026).
- Other baselines: ADAS (Hu et al., ICLR 2025); AFlow (Zhang et al., ICLR 2025); MaAS (Zhang et al., ICML 2025); DyLAN (Liu et al., COLM 2024); LLM-Debate (Du et al., ICML 2024).
- VQ-VAE (van den Oord et al. 2017); VQGraph (Yang et al., ICLR 2024); Cemri et al., "Why Do Multi-Agent LLM Systems Fail?" (NeurIPS 2025 D&B).
