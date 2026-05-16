---
title: MemoryBank
type: concept
created: 2026-05-16
updated: 2026-05-16
sources:
  - memorybank-zhong-2023.md
status: complete
importance: high
tags:
  - 2023
---

# MemoryBank

MemoryBank is a long-term memory mechanism for [[llm-agent|LLM agents]] proposed by Zhong et al. (2023, AAAI 2024), motivated by the inability of base ChatGPT/ChatGLM to sustain state across the kinds of extended interactions required by AI companions or counselors. Its distinctive contribution is the first principled, time-dependent **forgetting policy** for LLM memory, drawn from the [[ebbinghaus-forgetting]] curve.

## Core mechanism

- **Hierarchical storage**: raw timestamped turns → per-day event summaries → global event summary; in parallel, per-day personality insights → an evolving global user portrait. Summaries are produced by prompting the underlying LLM.
- **Dual-tower dense retrieval**: each memory piece is pre-encoded; current context is encoded into `h_c`; FAISS does nearest-neighbor search ([[dpr|dense passage retrieval]] style). Encoder is interchangeable (MiniLM for English, Text2vec for Chinese).
- **Ebbinghaus forgetting formula**: `R = e^(-t/S)` where `S` is a discretized strength initialized to 1; `t` is time since last interaction.
- **Recall reinforces**: when the retriever surfaces a memory, the system increments `S` by 1 and resets `t = 0` — recalled items decay more slowly, mirroring the spacing effect.
- **Backbone-agnostic**: same mechanism plugs into ChatGPT, ChatGLM-6B, BELLE-7B; demonstrated in the SiliconFriend chatbot (English + Chinese).

## What it advances

Primarily **evict** in the [[memory-management]] vocabulary: MemoryBank introduces the first time-dependent, psychologically-motivated decay policy over LLM memory items, replacing naive fixed-size windows or pure similarity-based eviction. It also exercises **consolidate** (daily-to-global hierarchical summarization, including the evolving user portrait) and standard **retrieve** (dual-tower dense), but the load-bearing novelty is the forget+reinforce rule.

## Relation to other systems

- **vs. [[memgpt]]** (concurrent, 2023): MemGPT contributes tier hierarchy and function-call paging; MemoryBank contributes a *temporal* eviction policy. The two are orthogonal and composable.
- **vs. [[titans]]** (2025): Titans gates forgetting on a learned surprise signal inside the model's weights; MemoryBank gates it on an exponential decay over wall-clock time in an external store.
- **vs. [[memory-r1]]** (2025): Memory-R1 learns when to forget via RL over an explicit `DELETE` action; MemoryBank uses a fixed analytic decay law.

## References

- [[memorybank-zhong-2023]]
