---
title: "Qwen-AgentWorld: Language World Models for General Agents"
type: source
created: 2026-07-06
updated: 2026-07-06
sources:
  - qwen-agentworld-language-world-models-for-general-agents-qwen-2026.md
arxiv_id: "2606.24597"
authors:
  - Qwen Team
first_author: Qwen Team
year: 2026
tags: [2026]
status: complete
importance: medium
---

# Qwen-AgentWorld: Language World Models for General Agents

*Credit: alphaXiv digest 2026-07-01.*

## Summary

Qwen-AgentWorld is a family of **language world models** — LLMs trained to predict environment dynamics (the next state given an observation and action) so that agents can reason and plan over simulated agentic environments. The team releases two MoE checkpoints (Qwen-AgentWorld-35B-A3B and 397B-A17B), described as the first language world models that simulate agentic environments across 7 domains via long chain-of-thought reasoning, trained on 10M+ real-world interaction trajectories through a three-stage pipeline (CPT → SFT → RL).

Beyond the foundation models, the paper argues world modeling helps general agents in two complementary ways: as a decoupled, scalable and controllable environment simulator that supplies training signal for [[agentic-rl]] (surpassing real-environment training alone), and as a unified agent foundation model where world-model pretraining acts as an effective warm-up that lifts downstream agent performance.

## Key points

- Two language world models released: **35B-A3B** and **397B-A17B** MoE models covering **7 domains** (SWE, terminal, search, MCP, web, OS, Android).
- Trained on **10M+** environment interaction trajectories via a three-stage pipeline: CPT (world-modeling capability), SFT (activates next-state-prediction reasoning), RL (sharpens simulation fidelity with hybrid rubric-and-rule rewards).
- Introduces **AgentWorldBench**, built from real interactions of **5 frontier models** on **9 established benchmarks** (Tool Decathlon, Terminal-Bench 1.0 & 2.0, OSWorld-Verified, etc.), judging world-modeling quality across **5 dimensions** with ground-truth-grounded rubrics.
- Reported to significantly outperform existing frontier LLMs on AgentWorldBench.
- As a decoupled simulator, enables scalable, controllable simulation of thousands of real-world environments for [[agentic-rl]], with gains beyond real-environment training alone.
- As a warm-up, world-model training improves downstream results across 7 agentic benchmarks.

## Concepts & entities

- [[world-model]] — the core paradigm: predicting environment dynamics for reasoning and planning.
- [[agent-benchmark]] — AgentWorldBench is a new benchmark for evaluating world-model simulation quality.
- Related: [[agentic-rl]] (training agents against the simulator), [[llm-agent]].
- Qwen (model family / team) — no dedicated wiki page yet.

## References

- [arXiv:2606.24597](https://arxiv.org/abs/2606.24597) — "Qwen-AgentWorld: Language World Models for General Agents", Qwen Team, 2026.
