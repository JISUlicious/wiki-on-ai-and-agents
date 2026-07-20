---
title: "AgentBench: Evaluating LLMs as Agents"
type: source
created: 2026-07-07
updated: 2026-07-07
sources: []
arxiv_id: "2308.03688"
venue: ICLR 2024
year: 2023
authors:
  - Xiao Liu
  - Hao Yu
  - Hanchen Zhang
  - Yuxiao Dong
  - Jie Tang
tags:
  - 2023
  - paper
status: complete
importance: high
---

# AgentBench: Evaluating LLMs as Agents

**Liu et al.** (Tsinghua/THUDM, Ohio State, UC Berkeley) — [arXiv:2308.03688](https://arxiv.org/abs/2308.03688), **ICLR 2024**.

The first broad multi-environment benchmark for LLMs *as agents*, and still one of the most-cited reference points in [[agent-evaluation]].

> [!note] Version caveat
> The arXiv abs page now serves **v3 (2025-10-04)**, which includes models postdating the ICLR 2024 camera-ready. Numbers below are v3; cite the version alongside the venue.

## The eight environments

| Family | Environment | Metric |
|---|---|---|
| Code-grounded | Operating System (bash) | Success Rate |
| Code-grounded | Database (SQL) | Success Rate |
| Code-grounded | Knowledge Graph | F1 |
| Game-grounded | Digital Card Game | Reward |
| Game-grounded | Lateral Thinking Puzzles | Game Progress |
| Game-grounded | House-Holding (ALFWorld-style) | Success Rate |
| Web-grounded | Web Shopping | Reward |
| Web-grounded | Web Browsing | Step Success Rate |

Five were newly constructed; three recompiled from existing datasets. **29 LLMs** evaluated (10 commercial, 19 open-source capped at <=70B), ~11k test-set inference calls.

## The commercial/open gap

GPT-4 scores **4.01** overall vs the best open model (codellama-34b) at **0.96** — roughly **4x**, and qualitative in the hardest environments: House-Holding 78.0 vs 4.0; Digital Card Game 74.5 vs 8.4. Open models are competitive only on Web Shopping, the shallowest environment.

## Why agents fail — the useful part

Execution-outcome analysis names three failure classes that map to three distinct deficits:

| Failure | Meaning | Where it dominates |
|---|---|---|
| **Invalid Format** | cannot emit required action syntax | Database **53.3%**, Card Game 38.5% |
| **Invalid Action** | valid syntax, inadmissible action | House-Holding **64.1%** |
| **Task Limit Exceeded** | runs out of turns unsolved | Knowledge Graph **67.9%**, Puzzles **82.5%** |

These correspond to **instruction-following**, **grounding**, and **long-horizon reasoning** failures respectively — a taxonomy that maps cleanly onto verification targets ([[verifier]]). The paper's own framing: *"poor long-term reasoning, decision-making, and instruction following abilities are the main obstacles."*

Two secondary findings: alignment data matters (Vicuna-13b beats llama-2-13b, read as evidence for high-quality **multi-round** alignment data), and **code training is ambivalent** — codellama-34b is the best open model overall yet near-zero on House-Holding and Puzzles.

## Limitations

Uses **primitive CoT prompting only** — no scaffolds, no multiple trials — so it measures the *base model*, not the best achievable agent. This is the key caveat for anyone reading it as an agent benchmark: it is closer to a model benchmark under an agentic interface. Open models capped at 70B, so the commercial gap is partly a scale gap. The composite score is normalized against the model pool, so it is only comparable within a fixed pool.

## Concepts

- [[agent-evaluation]] · [[verifier]] · [[terminal-bench]] · [[swe-bench]]

## References

- [arXiv:2308.03688](https://arxiv.org/abs/2308.03688)
