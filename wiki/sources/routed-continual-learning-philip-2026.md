---
title: "Routed Continual Learning"
type: source
created: 2026-11-10
updated: 2026-11-10
sources:
  - routed-continual-learning-philip-2026.md
tags:
  - 2026
  - paper
status: complete
importance: high
authors:
  - Michael Philip
  - Mahesh Balakrishnan
  - David Geraghty
  - David Mao
  - Vidhya Venkat
  - Ilya Mironov
year: 2026
venue: arXiv preprint
arxiv_id: "2610.38796"
---

# Routed Continual Learning

**Source:** Michael Philip, Mahesh Balakrishnan, David Geraghty, David Mao, Vidhya Venkat, Ilya Mironov ([[fair|FAIR at Meta]]) — [arXiv:2610.38796](https://arxiv.org/abs/2610.38796), cs.LG, v1 28 Oct 2026. Preprint.

## Summary

No single memory system suits every workload, and one shared history mixes experience that does not belong together. Routed CL treats [[continual-learning]] across a heterogeneous stream as a **routing problem with two decisions**: *how* to represent experience (which memory system) and *which* runs share state (which cluster). Before each instance, a prompted (not trained) LLM router either reuses an existing cluster or creates a new one and picks a memory system for it from a frozen library. The router sees only the visible prompt, the memory systems' declared behaviour, and each cluster's ID, type, description and trajectory count. It never sees raw trajectories or task labels. A **Context Memory Adapter** maps task-specific schemas onto generic memory state and injects a budget-bounded advisory into the agent's context. It never acts in the environment, so any memory effect passes only through context.

The library holds five systems in three families: retained trajectories ([[in-context-learning|ICL]]), distilled reasoning memory (ReasoningBank), and structured numerical state (Hierarchical Regression, Hierarchical Discrete-Event State, Persistent-Entity State, built with PyMC/River). The test stream interleaves all 301 CL-Bench instances (six tasks such as text-to-SQL and poker) with 97 [[tau-bench|τ³-Bench]] banking instances, giving 398 instances, five matched rollouts, and four models (Claude Opus 4.7, Opus 5, GPT-5.6 Sol, Gemini 3.7 Flash).

## Key points

- **Global ICL collapses under interleaving.** CL-Bench had found naive ICL strongest on isolated tasks. Interleaved, its mean gain over stateless is only **+0.015**, against 0.222 for Routed ICL and 0.280 for Routed CL.
- **Clustering helps ICL by +0.117 to +0.241** normalised reward. Choosing the memory system as well adds another +0.002 to +0.076, mainly on Cohort, τ³-Bench and Spectrum. Example: GPT-5.6 Sol scores 0.168 with Global ICL, 0.387 Routed ICL, 0.464 Routed CL.
- **Cost.** Routed CL reaches higher reward than Global ICL at **65–72% lower estimated API cost** and lies on the reward–cost Pareto frontier for every model. On Sales, Opus 5 holds reward at 0.34× the cost.
- **Routing ReasoningBank does not help consistently.** It already retrieves by request similarity, so clustering adds 8–29% cost for no steady gain.
- **The router is close to an oracle for strong models.** A ground-truth task→cluster assignment adds only 0.008 to 0.014 for Opus 5 and GPT-5.6 Sol, but about 0.07 for Opus 4.7 and Gemini. The router matches the oracle's memory system in 86% of cases and its partition in 91%.
- **Motivation.** The paper cites an OpenAI cyber evaluation in which independently started agents made an unplanned shared message board to pass findings to each other. The goal is to make that kind of sharing deliberate.

## Caveats

The memory library was chosen during exploratory work on CL-Bench and then frozen, so the paper tests *selection* within a library, not the synthesis of new memory systems. Instances run sequentially with one model per rollout. Workspace resets make the CL-Bench numbers not directly comparable to the original paper.

## Concepts & entities

- [[continual-learning]] · [[memory-management]] · [[multi-agent-memory]] · [[episodic-memory-llm]] · [[in-context-learning]] — the core framing.
- [[are-self-evolving-agents-continual-learners-chen-2026]] · [[reharness-recurrent-harness-agentic-continual-learning-xiao-2026]] — other CL-Bench / ReasoningBank work in the wiki.
- [[learned-orchestration]] · [[when-is-routing-meaningful-huot-2026]] — routing as the control decision.
- [[tau-bench]] · [[structured-outputs]].
- [[fair]] · [[meta]].

## References

- [arXiv:2610.38796](https://arxiv.org/abs/2610.38796)
