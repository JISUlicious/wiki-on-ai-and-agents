---
title: "Scaling Discovery Requires Forgetting"
type: source
created: 2026-10-29
updated: 2026-10-29
sources:
  - scaling-discovery-requires-forgetting-tan-2026.md
arxiv_id: "2610.20563"
authors:
  - Shangyin Tan
  - Ben Zhou
  - Lakshya A Agrawal
  - Jialin Zhang
  - Yilin Chen
  - Qiuyang Mang
  - Donghyun Lee
  - Dennis Jacob
  - Koushik Sen
  - Alex Dimakis
  - Matei Zaharia
year: 2026
status: complete
importance: high
tags:
  - 2026
  - paper
---

# Scaling Discovery Requires Forgetting

**Shangyin Tan, Ben Zhou, Lakshya A Agrawal, Jialin Zhang, Yilin Chen, Qiuyang Mang, Donghyun Lee, Dennis Jacob, Koushik Sen, Alex Dimakis, Matei Zaharia** ([[uc-berkeley|UC Berkeley]], [[stanford-university|Stanford]], [[bespoke-labs|Bespoke Labs]]). [arXiv:2610.20563](https://arxiv.org/abs/2610.20563), v1, cs.AI, 14 Oct 2026.

## Summary

Autoresearch can be scaled three ways: run one agent longer, run several in parallel, or let parallel agents share candidates. The paper argues that **all three plateau for the same reason**. As session context and search history grow, each agent anchors on its own past attempts and makes smaller, more conservative edits. The fix is **forgetting**: periodically discard every agent's session *and* the whole candidate history, keep only the best candidate, and restart fresh agents from it. This is stronger than Fork-and-Flush, which keeps the workspace. The best configuration is a **team of 4 that shares candidates as scored commits** (asynchronous, no chat) and **forgets 3 times** (4 rounds). All arms use the **Pi agent + DeepSeek V4.1 Flash** at matched total dollar cost, with 3 replicates per task.

## Key points

- **Table 1 (final best; mean ± s.d. over 3 runs):**
  - **AHC014:** single agent 795 ± 210; 4 parallel 964 ± 57; team with forgetting **1,103 ± 50**. That is **+39% vs single** and about +14% vs parallel.
  - **VLIW kernel (cycles, lower is better):** best run single 1,262, parallel 1,172, **team 1,077**. Means are 1,426 / 1,220 / **1,097**.
  - **Frontier-CS Q171:** means 64.0 / 70.2 / **80.8**, **+15% over parallel**.
  - **nanochat-autoresearch BPB:** 0.980 / 0.980 / **0.961**, only a 2% gain.
- On average the team with forgetting beats parallel agents by **10%** and a single agent by **22%**. **32 of 36 forgets** produce a new best within the next round. Most of the gain comes from the first forget (AHC014: +11.2%, then +4.8%, then +1.7%).
- **Continue vs forget from the same plateau (AHC014).** Continuing gains 1.6% (single), 1.9% (parallel) and 1.4% (team). Forgetting gains **4.5%, 7.0% and 9.2%** at about half the cost, because contexts are shorter.
- **Mechanism.** Before the first reset, team edits shrink from about 150 lines to 30 and reasoning from about 28k to 7k tokens. After a forget, edits go back to about 50 lines with at least 2× the reasoning. On VLIW, single agents spent **58–70% of their spend on empty turns** that declared a plateau. One stopped at 1,710 cycles, saying "practical limit"; the team later reached 1,077.

The newsletter's +15%/+39% and 1,077/1,172/1,262 match the paper. The cycle counts are best-run values; the means are 1,097/1,220/1,426.

## What it does not show

- **One model and harness** (DeepSeek V4.1 Flash, a cheap model, with Pi). Frontier models with longer effective context might plateau later or less.
- **Only 3 replicates.** The single-agent AHC014 s.d. is ±210, about 26% of its mean.
- **The forget schedule is fixed** (3 forgets, even rounds). There is no adaptive trigger, and the paper does not test forgetting the search state alone versus the session alone.
- **Communication is candidate-sharing only.** The claim that team plus forgetting is best is not tested against richer coordination such as Agora- or GitSwarm-style dependency graphs.

## Concepts & entities

- [[context-rot]] · [[catastrophic-remembering]]: accumulated history as a liability. Here it shows up as shrinking edit sizes and reasoning budgets, not as lost recall.
- [[ebbinghaus-forgetting]] · [[memory-management]]: deliberate forgetting as a memory policy.
- [[diversity-collapse-in-multi-agent-llm-systems-chen-2026]] · [[quality-diversity]]: exploration collapse in iterative loops.
- Direct counterpoint to [[gitswarm-decentralized-compounding-inference-shah-2026]] and [[agora-git-shared-memory-collective-autoresearch-zhang-2026]], which treat accumulated history as the asset.
- [[gepa-agrawal-2025]] (shared author) · [[pi-mono]] · [[deepseek-v4.1-flash-kv-cache-compression-deepseek-2026]] · [[nanochat]] · [[andrej-karpathy]] (autoresearch)
- [[self-improving-agent]] · [[multi-agent]]

## References

- [arXiv:2610.20563](https://arxiv.org/abs/2610.20563)
