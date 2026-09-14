---
title: "PARSER: Read in Parallel, Reason in Depth for Long-Context LLM Agents"
type: source
created: 2026-09-15
updated: 2026-09-15
sources:
  - parser-read-in-parallel-reason-in-depth-li-2026.md
arxiv_id: "2609.06702"
authors:
  - Kun Li
  - Zexuan Qiu
  - Tianhua Zhang
  - Irwin King
  - Helen Meng
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# PARSER: Read in Parallel, Reason in Depth for Long-Context LLM Agents

**Kun Li\*, Zexuan Qiu\*, Tianhua Zhang\*, Irwin King, Helen Meng** (\*equal contribution; The Chinese University of Hong Kong). [arXiv:2609.06702](https://arxiv.org/abs/2609.06702), cs.CL, v1 6 Sep 2026. No venue stated. Project page `cuhk-parser.github.io`. The paper writes the name as ParSer, for "Parallel Reading, Sequential Reasoning".

## Summary

**Sequential memory agents** (MemAgent, ReMemR1, GRU-Mem) read a long document chunk by chunk and rewrite a fixed-size textual memory after each chunk. T chunks therefore mean T dependent steps, and each chunk's relevance is judged before the rest of the document has been seen. PARSER decouples reading from reasoning. Each chunk is bound to a frozen **subagent** (Qwen3.5-4B, non-thinking) that answers a query from its chunk alone, with a verbatim quote as evidence, or returns "Unknown". A **lead agent** never sees a document token and runs a [[react]] loop. Each round it broadcasts one or more queries to all subagents in parallel, gathers the non-abstaining findings, and writes its next query conditioned on what came back. Sequential depth is therefore the number of reasoning rounds (about 4, flat with length) rather than the number of chunks. Only the lead agent is trained, with [[grpo]] on a binary exact-match reward and no format reward, over 32,768 synthetic HotpotQA samples of ~28K tokens each. Training uses 512-token chunks and ≤9 turns; inference uses 4,096-token chunks and ≤12 turns.

## Key results

All scores are Sub_EM on 128 questions × 8 lengths (7K to 896K tokens), averaged over 3 runs. DeepSeek-V4-Pro reading the full context (think-max) scores 80.47 on average and 78.91 at 896K.

| HotpotQA: average / at 896K | Qwen3.5-4B | Qwen3.5-9B |
|---|---|---|
| Full context (think) | 65.82 / 31.25 | 68.85 / 46.88 |
| MemAgent | 78.58 / 72.92 | 80.05 / 75.00 |
| ReMemR1 | 78.91 / 73.44 | 78.39 / 76.04 |
| **PARSER** | **84.57 / 85.42** | **86.79 / 85.94** |

- **Against the strongest sequential baseline:** +5.7 (4B) and +6.7 (9B) on average. **At 896K the gap is +12.0 for 4B (vs ReMemR1's 73.44) and +9.9 for 9B.** PARSER-4B stays within 83.07–85.68 across all eight lengths, while 4B full-context reading falls from 80.47 to 31.25.
- **Out of distribution (2WikiMultiHopQA):** PARSER scores 87.04 (4B) and 88.48 (9B), against 60.61 / 73.73 for MemAgent and 77.41 / 79.27 for ReMemR1. At 896K, PARSER-4B scores 86.98 vs 60.68 for ReMemR1.
- **Evidence placement** (894K-token documents, 512 questions per test). MemAgent dips when the evidence sits between the 50th and 70th percentiles, and both sequential agents degrade as the distance between two evidence paragraphs grows. Reversing the evidence order, per the figure's bar labels: MemAgent 66.5 → 23.3, ReMemR1 75.5 → 61.7, PARSER 96.4 → 97.1. The MemAgent trace in App. F.3 shows why: death-date paragraphs arrive before their link to the question is known, so they are never written to memory.
- **Latency** (amortized seconds per sample at 896K): MemAgent 876.2 vs PARSER 78.2 at concurrency 1 (**11.2×**), 101.9 vs 58.9 at concurrency 16 (1.7×), 75.0 vs 58.8 at concurrency 32 (1.3×). At 896K, PARSER's subagents generate 804 output tokens (plus 541 from the lead), against 187,058 for MemAgent.
- **What drives the gain:**
  - *Chunking.* A single subagent reading the full document averages 73.76 (53.13 at 896K), and accuracy falls steadily as chunks grow from 4K to 128K tokens (84.57 → 80.34).
  - *Subagent size saturates at 4B:* 78.26 for 2B, 84.57 for 4B, 84.77 for 9B.
  - *RL adds about 10 points* (74.42 → 84.57 at 4B), but the untrained workflow is already flat across lengths.

## What it does not show

Both benchmarks pad HotpotQA or 2WikiMultiHopQA questions with distractor paragraphs. The evidence therefore sits in a few paragraphs, and every hop is a lookup a single chunk can answer, which is the favorable case for scatter–gather. Nothing tests tasks that need aggregation across many chunks (summarization, counting) or facts that no single chunk states. The metric is the lenient bidirectional-substring Sub_EM from MemAgent's code, and every length reuses the same 128 questions. The DeepSeek margin holds only on HotpotQA: on 2WikiMultiHopQA, DeepSeek-V4-Pro think-max (88.18) beats PARSER-4B (87.04) and trails PARSER-9B by just 0.3. The latency lead narrows under load, because chunk KV caches get evicted and each round re-prefills; the appendix puts prefill at K× (about 4×) MemAgent's in that regime. PARSER also ran on two GPUs (an H100 plus an RTX 3090) against one H100 per baseline. Context isolation has a cost too: in the documented failure, a subagent holding a chunk about a similarly named Elena reports her husband, and the lead agent trusts it over identity-grounded evidence from another subagent. Training took about 312 h (4B) and 400 h (9B) on 6 H100s, plus 10 more H100s serving the subagents.

## Related

- [[long-context-llm]]: context rot inside 1M-token windows. Qwen3.5-4B reading the full context drops from about 80 to about 31 at 896K.
- [[memory-management]] · [[compaction-cliff-long-running-agent-memory-zerhoudi-2026]] · [[lost-in-compaction-side-constraint-loss-wang-2026]]: sequential memory evicts content before its relevance is known, the same failure as lossy compaction. PARSER avoids it by keeping no memory and re-reading everything under each new query.
- [[learned-orchestration]] · [[multi-agent]]: only the orchestrator is trained and the workers stay frozen. Here the workers own disjoint parts of the input, so coverage is guaranteed by construction.
- [[retrieval-augmented-generation]] · [[agentic-search]] · [[search-r1]] · [[coding-agents-effective-long-context-processors-cao-2026]]: PARSER reads every chunk for every query rather than a retriever's top-k. Plugged in as subagents, grep-style DCI searchers score 84.67, against 75.61 for a standalone DCI agent.
- [[react]] · [[grpo]] · [[reinforcement-learning]] · [[llm-agent]]
- [[prefix-caching]] · [[kv-cache]] · [[sglang-radixattention-zheng-2023]]: each subagent prompt is instructions, then chunk, then query, so the chunk's KV cache is reused across rounds through SGLang's cache-aware router.
- [[qwen]] · [[deepseek]]

## References

- [arXiv:2609.06702](https://arxiv.org/abs/2609.06702) (v1, 6 Sep 2026).
- Sequential memory agents: MemAgent (Yu et al., ICLR 2026); ReMemR1 (Shi et al., ICLR 2026); GRU-Mem (Sheng et al. 2026). Training-free: ReadAgent (Lee et al., ICML 2024); MemWalker (Chen et al. 2023); Chain-of-Agents (Zhang et al., NeurIPS 2024).
- Parallel and multi-agent reading: LLM×MapReduce (Zhou et al. 2024); LongAgent (Zhao et al. 2024); XpandA (Xiao et al. 2025). DCI: Li et al. 2026, arXiv:2605.05242.
- Context rot (Hong et al., Chroma 2025); Lost in the Middle (Liu et al., TACL 2024); DeepSeek-V4 (DeepSeek-AI 2026); Qwen3.5 (Qwen 2026).
