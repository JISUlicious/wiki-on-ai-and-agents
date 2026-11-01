---
title: "Cache Engineering for Agents"
type: source
created: 2026-10-29
updated: 2026-10-29
sources:
  - cache-engineering-for-agents-fu-2026.md
arxiv_id: "2610.28577"
authors:
  - Tianyu Fu
  - Ruoxi Wang
  - Zihan Min
  - Zhongqi Ren
  - Zhihao Li
  - Tengxuan Liu
  - Shengxu Mu
  - Hanling Zhang
  - Yu Wang
year: 2026
status: complete
importance: high
tags:
  - 2026
  - paper
---

# Cache Engineering for Agents

**Tianyu Fu, Ruoxi Wang, Zihan Min, Zhongqi Ren, Zhihao Li, Tengxuan Liu, Shengxu Mu, Hanling Zhang, Yu Wang** ([[tsinghua-university|Tsinghua University]], [[chinese-university-of-hong-kong|CUHK]]) — [arXiv:2610.28577](https://arxiv.org/abs/2610.28577), v1, cs.CL, 20 Oct 2026. Code: `thu-nics/AKV`.

## Summary

[[context-engineering|Context engineering]] shrinks an agent's history by trimming or summarizing text. Any edit to the trajectory prefix invalidates the cached KV after it, so the retained suffix must be re-prefilled. Harnesses therefore wait until a length threshold and then cut hard, which loses intermediate information. **Cache engineering** leaves the text alone and lets the agent operate on its active [[kv-cache]] directly, with two new operations beside `append`:

- **drop** — exclude selected KV entries from future attention;
- **repos** — rewrite the position encoding of retained keys (keeping positions inside the range the model handles well).

Retained KV is reused without recomputation, even though it was computed while the dropped tokens were visible. This creates **history dependence**: the same token sequence can yield different cache states under different operation histories, so plain prefix matching is no longer correct for reuse. The **AKV** serving engine, built on [[sglang|SGLang]], keeps a stateless API outward while tracking stateful cache versions, eviction and restoration inside.

## Key points

- **Motivation.** In the authors' accounting, KV-cache reads are 82.4% of total agent API cost.
- **Setup.** MiniMax-M2.7, GPT-OSS-120B and MiroThinker-1.7-Mini on BrowseComp-Plus, xbench-DeepSearch, GAIA-Text (103 tasks) and SWE-Verified. Baselines: "Standard" summarization at 96K (192K for MiroThinker) and "Text Engineering" at 48K. AKV uses a rolling drop that keeps the KV of the latest 12 tool responses.
- **Accuracy and cost.** Average success +1.3–2.1 points over Standard and +0.9–8.7 over Text Engineering (abstract: +1.7 on average). Cost −27.7% vs Standard and −21.9% vs Text Engineering, averaged over 3 models × 4 benchmarks. Repos tokens are charged at the cache-hit rate, which overstates their cost.
- **Serving.** At peak-throughput settings, 1.36× mean and 1.51× P95 latency speedup; input and output throughput 1.53× and 1.18× of Standard.
- **Recursive self-improvement of policies.** MiniMax-M2.7 refined cache policies on 20 BrowseComp-Plus tasks. On 100 held-out tasks, the iteration-3 and iteration-6 policies cut cost 15.6% and 17.8% vs Standard at matched accuracy, a further 7.1% below the hand-designed rolling drop.
- **Cache management as a tool.** Exposed as an agent tool, it gives the best BCP pass rates (61.0% GPT-OSS-120B, 64.0% MiniMax-M2.7). Cost falls for GPT-OSS but rises for MiniMax, which runs more turns.

The newsletter's numbers (+1.7, −27.7%, 1.2×, 1.5× P95, RSI −7.1%) match the abstract; "1.2×" is output throughput only.

## What it does not show

- Dropped-then-reused KV still "remembers" the dropped tokens, so behaviour depends on operation history. Accuracy is shown empirically on four benchmarks, not guaranteed.
- No comparison with KV-compression methods (e.g. [[kv-cache-eviction]] heuristics such as H2O/SnapKV) as agent-level policies; the baselines are text-summarization harnesses.
- Serving tests use eight A800s with BF16 weights; the RSI loop is run with one model on one benchmark.

## Concepts & entities

- [[kv-cache]] · [[kv-cache-eviction]] · [[prefix-caching]] · [[positional-encoding]] · [[paged-attention]] — the serving substrate.
- [[context-engineering]] · [[context-rot]] · [[memory-management]] — the text-level alternative it replaces.
- [[context-language-models-shao-2026]] — the opposite design: edit text freely, then recover cache reuse with Suffix Cache Reuse.
- [[self-improving-agent]] — RSI over cache policies.
- [[sglang]] · [[minimax]] · [[swe-bench-verified]] · [[gaia]].
- [[context-compaction]]: concept page on when, what and who compacts agent context, and what compaction loses.

## References

- [arXiv:2610.28577](https://arxiv.org/abs/2610.28577)
