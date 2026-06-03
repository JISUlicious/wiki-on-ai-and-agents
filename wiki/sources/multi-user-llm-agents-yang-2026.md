---
title: "Multi-User Large Language Model Agents"
type: source
created: 2026-06-03
updated: 2026-06-03
sources:
  - multi-user-llm-agents-yang-2026.md
arxiv_id: "2604.08567"
authors:
  - Shu Yang
  - Shenzhe Zhu
  - Hao Zhu
  - José Ramón Enríquez
  - Di Wang
  - Alex Pentland
  - Michiel A. Bakker
  - Jiaxin Pei
first_author: Shu Yang
year: 2026
tags:
  - 2026
status: complete
importance: medium
---

# Multi-User Large Language Model Agents

[arXiv:2604.08567](https://arxiv.org/abs/2604.08567) — Yang, Zhu, Zhu, Enríquez, Wang, Pentland, Bakker, Pei (Stanford, KAUST, UT Austin, MIT), 2026. Surfaced via the alphaXiv / NLP-newsletter digest.

## Summary

Most LLM agents are implicitly built for a **single-principal** paradigm: one dominant user whose instructions are the sole source of authority and utility. As agents move into team workflows and organizational tools, they must serve **multiple users simultaneously** — each with distinct roles, preferences, and authority levels — creating multi-principal settings with unavoidable conflicts, information asymmetry, and privacy constraints. This paper presents the first systematic study of multi-user LLM agents, formalizing the interaction as a **multi-principal decision problem** where a single agent must reconcile potentially conflicting interests.

The authors introduce a unified multi-user interaction protocol and three targeted stress-test scenarios probing instruction following, privacy preservation, and coordination. Results reveal systematic gaps: frontier LLMs fail to keep stable prioritization under conflicting objectives, exhibit increasing privacy violations over multi-turn interactions, and hit efficiency bottlenecks when coordination requires iterative information gathering.

## Key points

- **Single- to multi-principal shift.** Recasts user–agent interaction from optimizing one fixed objective to role-aware reasoning, selective context sharing, and cross-user coordination among many principals.
- **Formalization.** Multi-user interaction is framed as a multi-principal decision problem with conflicting interests, information asymmetry, and privacy as first-class constraints.
- **Unified interaction protocol** plus three stress-test scenarios targeting instruction following, privacy preservation, and coordination.
- **Unstable prioritization.** Frontier models fail to maintain consistent prioritization when users' objectives conflict.
- **Privacy erosion over turns.** Privacy violations *increase* across multi-turn interactions as the agent leaks one user's information to another.
- **Coordination bottlenecks.** Efficiency degrades when serving a group requires iterative back-and-forth information gathering.

## Concepts & entities

- [[llm-agent]] — the core unit being extended from single- to multi-user service.
- [[multi-agent-memory]] — related to managing per-principal state, though here a single agent juggles many users.
- [[stanford-university]] — co-affiliation of several authors.
- [[mit]] — co-affiliation (Pentland, Bakker).

## References

- [arXiv:2604.08567](https://arxiv.org/abs/2604.08567)
