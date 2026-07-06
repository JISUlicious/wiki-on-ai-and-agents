---
title: "The Hitchhiker's Guide to Agentic AI: From Foundations to Systems"
type: source
created: 2026-07-06
updated: 2026-07-06
sources:
  - hitchhikers-guide-to-agentic-ai-roitman-2026.md
arxiv_id: "2606.24937"
authors:
  - Haggai Roitman
first_author: Haggai Roitman
year: 2026
tags: [2026, survey]
status: complete
importance: medium
---

# The Hitchhiker's Guide to Agentic AI: From Foundations to Systems

*Credit: alphaXiv digest 2026-07-01.*

## Summary

A single-author survey/guide (Haggai Roitman, v1.2.2) that synthesizes the modern agentic-AI stack into one practical reference. It moves "from foundations to systems": starting with LLM architecture and optimization, through the reinforcement-learning methods used to align and train models, into the systems-engineering and infrastructure needed to run agents at scale, and finally the agentic architectures, patterns, and production concerns that tie them together.

The guide is organized as ~19+ chapters spanning three arcs. The **Foundations** arc covers LLM architecture/optimization, systems foundations, and a deep RL sequence (PPO, DPO, GRPO, preference-optimization variants, reward-model training, SFT best practices) plus RL for large reasoning models and RLVR. The **Systems/agentic** arc covers agentic training, evaluation, introduction to agentic AI, RAG, agentic memory systems, the agent harness (context management and orchestration, including MCP and multi-agent orchestration), and agent design patterns. It is pitched as a unified, hands-on map rather than a novel-result paper.

## Key points

- Single-author guide by **Haggai Roitman** (v1.2.2, 2026); explicitly framed "From Foundations to Systems".
- **Foundations** arc: LLM architecture & optimization, systems foundations, and a full RL track — PPO, DPO, GRPO, preference-optimization variants, reward-model training, SFT best practices.
- Covers **RL for large reasoning models** including Reinforcement Learning from Verifiable Rewards (RLVR).
- **Systems & agentic** arc: agentic training (incl. Reflexion), LLM evaluation, RAG (BM25/TF-IDF, DPR, hybrid RRF, SPLADE, RAFT), agentic memory, and the agent harness.
- Agent-harness chapter details context management, orchestration, **Model Context Protocol (MCP)**, and multi-agent orchestration; a dedicated chapter catalogs **agent design patterns**.
- Positioned as a unified practical reference / textbook-style map of the agentic-AI landscape, not a single-experiment paper.

## Concepts & entities

- [[llm-agent]] — the central subject; the guide surveys how to build and operate LLM agents end to end.
- Touches on [[reinforcement-learning]] / [[agentic-rl]], [[mcp]] (via the agent-harness chapter), and RAG — linked only where existing pages apply.

## References

- [arXiv:2606.24937](https://arxiv.org/abs/2606.24937) — "The Hitchhiker's Guide to Agentic AI: From Foundations to Systems", Haggai Roitman, 2026.
