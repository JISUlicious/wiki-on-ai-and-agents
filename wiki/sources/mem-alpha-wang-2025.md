---
title: "Mem-α: Learning Memory Construction via Reinforcement Learning"
type: source
created: 2026-05-16
updated: 2026-05-16
sources:
  - mem-alpha-wang-2025.md
arxiv_id: "2509.25911"
authors:
  - Yu Wang
  - Ryuichi Takanobu
  - Zhiqi Liang
  - Yuzhen Mao
  - Yuanzhe Hu
  - Julian McAuley
  - Xiaojian Wu
first_author: Yu Wang
year: 2025
introduces:
  - "[[mem-alpha]]"
  - "[[memory-construction]]"
tags:
  - 2025
status: complete
importance: high
---

# Mem-α: Learning Memory Construction via Reinforcement Learning

- **Source**: `sources/mem-alpha-wang-2025.md`
- **arXiv**: [arXiv:2509.25911](https://arxiv.org/abs/2509.25911)
- **Authors**: Yu Wang, Ryuichi Takanobu, Zhiqi Liang, Yuzhen Mao, Yuanzhe Hu, Julian McAuley, Xiaojian Wu (Anuttacon; UC San Diego; Stanford University — first author's work done during Anuttacon internship)
- **Note on author disambiguation**: This Yu Wang (yuw164@ucsd.edu, UCSD/Anuttacon) is *not* the same Yu Wang who co-authored MIRIX with Chen — that work is cited here as a baseline reference (`Wang & Chen, 2025`).

## Summary

Mem-α is a **reinforcement-learning framework that trains LLM agents to *construct* multi-component memory systems** from raw input streams. Where existing memory-augmented agents (MemGPT, Mem0, MIRIX) ship with elaborate tool sets but rely on the base model to figure out *what* to store, *how* to structure it, and *when* to update — capabilities the paper argues even GPT-4o lacks reliably — Mem-α treats memory construction as a sequential decision-making problem and directly optimizes it via downstream QA accuracy. The agent observes context chunks one at a time and emits structured `memory_insert` / `memory_update` / `memory_delete` tool calls against a memory store with **core (≤512-token persistent summary), semantic (factual entries), and episodic (timestamped events)** components.

Training uses GRPO with four reward terms: r1 correctness (BM25-RAG QA accuracy over the final memory), r2 tool-call format validity, r3 compression (1 − memory length / chunk length), and r4 memory-content quality judged by Qwen3-32B. The training corpus is 562 stratified instances (from a 4,139-instance pool) drawn from MemoryAgentBench's Accurate Retrieval / Test-Time Learning / Long-Range Understanding axes, with maximum input length 30k tokens. Backbone is Qwen3-4B; 32 H100 GPUs for three days, 205 training steps.

The headline result is **length generalization**: trained only on ≤30k-token sequences, Mem-α handles MemoryAgentBench instances up to **474k tokens — over 13× the training length** — while compressing memory to roughly 50% of long-context or RAG baselines and outperforming MemAgent and MEM1 across every axis. This is positioned as evidence that RL teaches *fundamental memory-management principles* rather than memorizing surface patterns, and complements work like Memory-R1 (which trains an *editor* over existing memory operations) by tackling the harder problem of learning *how to build structured memory from scratch*.

## Key Points

- **Memory construction as RL, not as prompting**: existing systems give models tools and hope; Mem-α trains the *policy* over those tools end-to-end, with no need for ground-truth construction traces.
- **Three-component memory architecture**: core (single ≤512-token summary, supports only `memory_update` requiring holistic rewrite), semantic (atomic factual statements), episodic (chronologically-organized timestamped events). Semantic and episodic both support insert / update / delete.
- **Four reward components** combined as r = r1 + r2,t + β·r3 + γ·r4,t with β=0.05, γ=1 fixed after tuning. r1 (QA correctness) and r3 (compression) are global; r2 (tool-call format) and r4 (semantic validity) are per-action.
- **Decoupled RAG evaluator**: the comprehensiveness of constructed memory is judged by a frozen BM25 retriever + frozen generator answering held-out questions; only the *write policy* is learnable. This sidesteps the need for memory ground truth.
- **GRPO with KL term dropped** to encourage exploration; group-relative advantages over 8 rollouts per prompt.
- **Length extrapolation 30k → 400k+ tokens (13×)**: on MemoryAgentBench Multi-Doc the average input is 474k tokens; Mem-α (Qwen3-4B) hits 0.68 F1 vs. RAG-Top2's 0.45 and MEM1's 0.18 while using ~50% the memory footprint of long-context / RAG.
- **Structured memory beats flat memory**: MEM1 and MemAgent (single-paragraph memories) score 0.071 and 0.198 average on MemoryAgentBench respectively; Mem-α scores 0.592. The gap validates the hierarchical (core/episodic/semantic) split as well as the RL optimization.
- **RL provides the lift, not just the architecture**: ablations show base Qwen3-4B with the same memory framework achieves only 0.389 average; gpt-4.1-mini with the same framework still trails the RL-tuned 4B model. Small models *can* manage complex memory if trained to.
- **Contrast with Memory-R1 / MEM1 / MemAgent**: those works train relatively flat memory representations (single paragraphs, fact lists) on LoCoMo-scale data (≤26k tokens). Mem-α trains a *substantially more capable* multi-component memory and demonstrates 13× length extrapolation beyond training distribution.
- **Modularity**: the memory architecture is decoupled from the RL framework — researchers can substitute alternative memory designs without changing the training methodology.

## Construction vs Editing

A useful framing for the 2025 RL-trained-memory cluster:

- **Memory-R1** (Yan et al., 2025) — learns to *edit* an existing memory of facts: when to add / merge / delete given a new observation. Pre-existing memory schema; the learned policy is over edit operations.
- **MEM1** (Zhou et al., 2025), **MemAgent** (Yu et al., 2025) — learn to maintain a single rewritable paragraph; closer to compressed-state RNN-style memory.
- **Mem-α** (this paper) — learns to *construct* a richer, multi-component memory from scratch given a stream, including deciding which component (core/semantic/episodic) a new piece of information belongs in. The harder problem; the larger model surface to learn.

## Entities Mentioned

- [[anuttacon]] (industry affiliation; sponsor of first author's internship)
- [[uc-san-diego]] (academic affiliation)
- [[stanford-university]] (co-author affiliation)
- [[memgpt]] (baseline architecture; source of core-memory pattern)
- [[mem0]] (cited baseline memory system)
- [[mirix]] (cited baseline; motivates the "even GPT-4o struggles with complex memory tools" argument)
- [[qwen-3]] (Qwen3-4B backbone, Qwen3-32B reward judge)
- [[gpt-4-1-mini]] (ablation comparison)

## Concepts Discussed

- [[mem-alpha]]
- [[memory-construction]]
- [[memory-management]]
- [[reinforcement-learning]]
- [[grpo]]
- [[long-context-llm]]
- [[semantic-memory]]
- [[episodic-memory-llm]]
- [[core-memory]]
- [[tool-use]]
- [[retrieval-augmented-generation]]
- [[bm25]]
- [[llm-agent]]
- [[length-generalization]]
- [[memory-r1]]

## Notable Quotes

> "Despite being trained exclusively on instances with a maximum length of 30k tokens, our agents exhibit remarkable generalization to sequences exceeding 400k tokens — over 13× the training length, highlighting the robustness of Mem-α."

> "Language models may lack the ability to determine which information to store, how to structure it, and when to update it — especially as memory systems become more complex. This results in suboptimal memory construction and information loss."

> "This exceptional length generalization suggests that reinforcement learning enables agents to learn fundamental memory management principles rather than merely memorizing specific patterns."

## References

_Original source: `sources/mem-alpha-wang-2025.md`_
