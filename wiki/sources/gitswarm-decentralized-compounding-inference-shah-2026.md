---
title: "GitSwarm: Decentralized Compounding Inference"
type: source
created: 2026-10-29
updated: 2026-10-29
sources:
  - gitswarm-decentralized-compounding-inference-shah-2026.md
arxiv_id: "2610.04862"
authors:
  - Vedant Shah
  - Ankur Samanta
  - Paras Dahal
  - Mikhail Plekhanov
  - Carole-Jean Wu
  - Scott Yih
  - Remi Munos
  - Rob Fergus
  - Jakob Foerster
  - Ruslan Salakhutdinov
  - Sanjeev Arora
  - Jason Weston
  - Aaron Courville
  - Anirudh Goyal
year: 2026
status: complete
importance: high
tags:
  - 2026
  - paper
---

# GitSwarm: Decentralized Compounding Inference

**Vedant Shah, Ankur Samanta, Paras Dahal, Mikhail Plekhanov, Carole-Jean Wu, Scott Yih, Remi Munos, Rob Fergus, Jakob Foerster, Ruslan Salakhutdinov, Sanjeev Arora, Jason Weston, Aaron Courville, Anirudh Goyal** ([[meta|Meta Superintelligence Labs]]; also Mila / Université de Montréal, Columbia, [[cmu|CMU]], [[princeton-university|Princeton]]). [arXiv:2610.04862](https://arxiv.org/abs/2610.04862), v1, cs.AI, 4 Oct 2026.

## Summary

The paper names a paradigm, **compounding inference**: organize inference-time compute so that intermediate work (partial solutions, experiments, failures) persists and later compute can inspect, extend, combine or challenge it. GitSwarm implements it with **homogeneous asynchronous workers** sharing one branchable Git repository. Each contribution is an atomic commit. The key design choice is that a commit's **physical Git parent** (the state the worker started from) is kept separate from a **mandatory `informed_by=` list** of semantic dependencies. A worker can therefore extend one branch while citing discoveries from others. No planner assigns tasks. A harness handles scheduling and validation, and the final answer comes from **worker nominations plus a deterministic plurality readout**.

**Same idea as Agora, compared directly.** [[agora-git-shared-memory-collective-autoresearch-zhang-2026|Agora]] (NVIDIA, 2609.18094) is acknowledged as concurrent work. Both use a Git DAG as shared memory for self-directed agents. The differences:
- **Dependencies.** Agora encodes them as multi-parent Git edges. GitSwarm uses one physical parent plus `informed_by=`.
- **Selection.** Agora uses an evidence score plus diversity-UCB recommendations. GitSwarm uses nominations and plurality.
- **Evaluation.** Agora is one 11-day case study with no baseline. GitSwarm adds fixed-answer benchmarks with a "Forced" single-agent baseline and synthetic substrate tests.

The authors call the two "complementary evidence… rather than a direct performance comparison."

## Key points

- **ProgramBench** (fixed 50-of-200 task subset). With Codex (GPT-5.5-high) workers, GitSwarm scores **73.1% at a budget of 100 worker-episodes and 79.4% at 500**. With GPT-5.5 workers it scores 64.6% at 100 and 71.2% at 300. "Forced Codex", a single agent prompted repeatedly to continue, stays at **63.1–65.1%**. Gains diminish while input-token cost grows.
- **IMOProofBench-Advanced: 30/30 in a single run** with GPT-5.5-high, using slightly under 1M output tokens. With Gemini-3.1-Pro, mean fully correct proofs rise from 23.3 (budget 20) to 29.0 (budget 160), but not monotonically (there is a dip at 80).
- **Research tasks:**
  - **RMT.** Two 12-worker swarms reach held-out losses of 2.890 and 2.894, against 3.147 for the reference, with the same 1.3B tokens. Four of the six winning changes came from other branches; one was a failed idea revived 49.5 h later.
  - **Looped Transformer (Huginn).** Loss 3.187 vs 3.265; a parameter-matched control keeps two thirds of the gain.
  - **NanoChat.** Validation BPB falls from 0.993 to 0.905, but the model grows to about 1.72B parameters.
- **Accumulation metrics.** On ProgramBench, **94.7%** of contributions are later built on (44.4% on proofs). The selected solution's ancestry covers **81.6–92.6%** of the graph. In NanoChat, **70.1%** of negative-labelled commits are cited later; in Looped Transformer the figure is 74%.

> [!warning] Newsletter vs paper
> - **"Hundreds of agents share one Git repo."** The 100/500 figures are **budgets of charged worker episodes**. Concurrency is fixed at **5 workers** for problem-solving tasks, and the research swarms use 12–20 workers.
> - **The 70.1% figure is NanoChat-specific.** It refers to negative-labelled *research* commits in the NanoChat run, not failed commits in general.
> - **Not comparable with Agensh.** GitSwarm's ProgramBench 79.4 (50-task subset, mean score) cannot be compared with [[agensh-scaling-organizational-intelligence-1024-agents-zhan-2026|Agensh]]'s 28.78% (5 hardest tasks, test-pass rate).

## What it does not show

- **No causal attribution.** The authors say the dependency analyses "do not isolate the resulting performance gains." The paper never runs matched restarts with and without the history.
- **"Comparable compute" is only approximate.** Baselines are matched on token curves, not total compute or wall-clock time. The 30/30 is a single run, and some baselines also hit the ceiling.
- **Research comparisons are uneven.** In RMT the single-agent comparisons train on more tokens. The Looped Transformer runs were still in progress, and NanoChat runs differ in checkpoint and GPU allocation.
- **Overhead grows with the repository** (input tokens spent on inspection).

## Concepts & entities

- [[multi-agent]] · [[multi-agent-memory]]: Git as structured persistent memory. Compare [[agensh-scaling-organizational-intelligence-1024-agents-zhan-2026]] (Git + chat + findings log).
- [[inference-time-compute]] · [[test-time-compute]]: "compounding" as an alternative to independent sampling or longer chains.
- [[diversity-collapse-in-multi-agent-llm-systems-chen-2026]]: GitSwarm keeps competing branches alive rather than adding an explicit diversity mechanism.
- [[scaling-discovery-requires-forgetting-tan-2026]]: the opposite prescription; discarding accumulated history beats keeping it.
- [[looped-transformer]] · [[nanochat]] · [[agent-benchmark]] · [[meta]]
- [[programbench]]: benchmark page; scores across papers use different subsets and metrics.

## References

- [arXiv:2610.04862](https://arxiv.org/abs/2610.04862)
