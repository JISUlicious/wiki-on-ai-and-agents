---
title: "Is Complex Agentic Memory Always Needed? Naive RAG Is Stronger Than We Thought"
type: source
created: 2026-11-10
updated: 2026-11-10
sources:
  - is-complex-agentic-memory-always-needed-naive-rag-bao-2026.md
arxiv_id: "2610.30755"
venue: arXiv preprint (cs.AI)
authors:
  - Zhengye Bao
  - Shengtao Zhang
  - Haokai Zhang
  - Shiqi Lou
  - Muning Wen
  - Minnan Luo
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# Is Complex Agentic Memory Always Needed? Naive RAG Is Stronger Than We Thought

**Source:** Zhengye Bao et al. (Xi'an Jiaotong, [[shanghai-jiao-tong-university|SJTU]], [[zhejiang-university|Zhejiang]], [[chinese-university-of-hong-kong|CUHK]]) — [arXiv:2610.30755](https://arxiv.org/abs/2610.30755), cs.AI, v1 22 Oct 2026. Code: github.com/Litmeb/RAW-RAG.

## Summary

A baseline-correction paper for [[long-term-memory|long-term conversational memory]]. **RAW-RAG** is plain [[retrieval-augmented-generation|RAG]] over raw chat history: 256-token windows (32 overlap) with timestamps and image captions, [[dense-retrieval]] of the top 128, a cross-encoder reranker keeping the top 15, and a reader LLM that works out state changes itself. There is no extraction, summarisation, graph or mutation. Across [[locomo|LoCoMo]], [[longmemeval|LongMemEval]], MEME and MemoryAgentBench it **ranks first among compared memory systems in 6 of 7 benchmark–reader settings**, beating the strongest agentic system by 0.13–5.53 points. It is also far cheaper to ingest.

## Key points

- **Mutation is rare and doesn't help.** An audit of [[mem0|Mem0]] finds UPDATE+DELETE in only 4.39% (LoCoMo) and 2.46% (LongMemEval) of write decisions. Effective, persisted mutations are 1.90% and 1.34%. Swapping to append-only ADD/NONE *raises* LoCoMo accuracy from 67.36% to 69.42%.
- **Readers can resolve state from raw evidence.** On MEME with oracle phase-visible evidence, DeepSeek-V4-Flash reaches 82.32% on cascades and 91.54% on invalidation.
- **Prior baselines were weak.** A review of 14 papers finds coarse retrieval units, shallow depth or no reranking in their RAG baselines.
- **Ablations.** Removing the reranker costs 3.92 points. Final-k from 5 to 60 stays within 1.3 points. Adding [[memory-construction|memory management]] on top (atomic-fact extraction, summary augmentation, [[graph-memory|graph]] expansion) *lowers* accuracy by 5.8–19.6 points. Fact extraction wrecks test-time learning (47.68% → 15.37%) because it destroys demonstrations.
- **Cost.** About 5.1× faster than the next system on a MEME slice and 30.4× faster than EverMemOS. Streaming a ~1M-token BEAM conversation takes 0.33 h to ingest vs 3.25 h (MemOS) to 26.95 h (Memora).
- **Why it works: coverage over utilisation.** Systems differ more in whether sufficient evidence is retrieved (coverage 19–45% on MemoryAgentBench) than in using it once retrieved (utilisation 80–90%). RAW-RAG leads coverage, and it keeps the *dependency paths* needed to derive the current value: P(path | result) is 62.34%, vs 9.62% for MemU.
- **Lost information stays lost.** Even oracle access to each system's complete transformed store leaves a gap of 7.6–29.4 points to raw evidence. Source-preserving stores (RAW-RAG, MemMachine) recover best.

## Limitations

Offline public conversation benchmarks only; not interactive, multimodal or operational. Raw storage raises privacy and deletion questions. Results depend on reader strength: weaker readers might still gain from explicit state. Full-context still beats RAW-RAG by ~20 points on DeepSeek MEME.

## Concepts & entities

- [[retrieval-augmented-generation]] · [[dense-retrieval]] · [[long-term-memory]] · [[memory-management]] · [[memory-construction]] · [[memory-evolution]] · [[graph-memory]] · [[episodic-memory-llm]]
- Benchmarks: [[locomo]] · [[longmemeval]]
- Compared systems with pages: [[mem0]] · [[a-mem]]
- Context: [[context-rot]] (lost-in-the-middle is named as a residual cause)

## References

- [arXiv:2610.30755](https://arxiv.org/abs/2610.30755)
