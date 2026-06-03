---
title: "Memex(RL): Scaling Long-Horizon LLM Agents via Indexed Experience Memory"
type: source
created: 2026-06-03
updated: 2026-06-03
sources:
  - memex-rl-wang-2026.md
arxiv_id: "2603.04257"
authors:
  - Zhenting Wang
  - et al.
first_author: Zhenting Wang
year: 2026
tags:
  - 2026
status: complete
importance: medium
---

# Memex(RL): Scaling Long-Horizon LLM Agents via Indexed Experience Memory

[arXiv:2603.04257](https://arxiv.org/abs/2603.04257) — Zhenting Wang et al. (Center for Advanced AI, Accenture), 2026. Surfaced via the alphaXiv / NLP-newsletter digest.

## Summary

Long-horizon LLM agents are bottlenecked by finite context windows: as trajectories grow over dozens-to-hundreds of steps, tool outputs and reasoning traces fill the prompt and distant evidence becomes hard to use even when present. Truncation and running summaries are fundamentally lossy because they discard the evidence itself. **Memex** instead compresses context *without discarding evidence*: it keeps a compact working context of concise structured summaries plus stable indices, while archiving full-fidelity interactions in an external experience database keyed by those indices. The agent can later **dereference** an index to recover the exact past evidence needed for the current subgoal (via `CompressExperience` and `ReadExperience` operations).

**MemexRL** is the RL framework that trains both write and read behaviors — what to summarize, what to archive, how to index, and when to retrieve — using reward shaping tailored to indexed memory under a context budget plus compression-adaptive trajectory training. The paper adds a theoretical result: with a compact summary and a bounded number of dereferences per step, working context (and effective in-context computation) stays bounded even as full history grows without bound, while preserving decision quality. Empirically, on a harder modified ALFWorld with Qwen3-30B-A3B-Thinking, task success rises from 24.2% to 85.6% while peak working context stays small.

## Key points

- Indexed experience memory: compact summaries + stable indices in-context; full-fidelity artifacts in an external key-value store `D: index → content`.
- Lossless-by-reference vs. summary-only: evidence is archived, not discarded; agent dereferences on demand, keeping access precise and auditable.
- MemexRL trains write/read policies jointly with reward shaping for indexed memory usage under a fixed context budget + compression-adaptive trajectories.
- Theory: bounded working context (and bounded effective computation) with at most B dereferenced blocks per step, even as full history grows unbounded; compression ratio grows without bound.
- Backbone: Qwen3-30B-A3B-Thinking-2507 (MoE, ~3B active params).
- Environment: harder modified ALFWorld (hidden admissible commands to remove navigation triviality).
- Task success improves 24.2% → 85.6%; total penalty shrinks from −0.4 to ~−0.1, indicating learned peak-context squeezing.

## Concepts & entities

- [[memory-management]] — indexed write/read memory policy.
- [[virtual-context]] — externalize archive, page in evidence on demand (MemGPT-style).
- [[rl-memory-policy]] — RL-trained memory read/write behavior.
- [[long-context-llm]] — addresses finite-context bottleneck for long-horizon agents.
- [[agentic-rl]] — RL over agent trajectories with memory tools.
- [[llm-agent]] — long-horizon agent setting.

## References

- [[memgpt]] — virtual-context / paging precursor.
- [[memory-r1]], [[mem-alpha]] — RL-for-memory neighbors.
- [[rl-memory-policy]] — broader concept.
