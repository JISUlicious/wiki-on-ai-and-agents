---
title: "Agentic Hardware Design as Repository-Level Code Evolution"
type: source
created: 2026-07-06
updated: 2026-07-06
sources:
  - agentic-hardware-design-repository-level-code-evolution-yu-2026.pdf
arxiv_id: "2606.28279"
authors:
  - Cunxi Yu
  - Chenhui Deng
  - Nathaniel Pinckney
  - Brucek Khailany
first_author: Cunxi Yu
year: 2026
tags: [2026]
status: complete
importance: medium
---

# Agentic Hardware Design as Repository-Level Code Evolution

Surfaced via the NLP newsletter (Top AI Papers of the Week, 2026-06-28–07-05).

## Summary

HORIZON (NVIDIA Research) is a self-evolving agent framework that treats **hardware design as repository-level code evolution**. A structured **Markdown harness** is compiled into a **project pack** containing domain knowledge, an **executable evaluator**, an **acceptance predicate**, and a **git/runtime policy**. A hands-free agent loop then evolves an **isolated git worktree**, using repository operations (diffs, commits, logs, history) for state management, tracing, and replay. This extends prior repository-scale self-evolution (AlphaEvolve, SATLUTION, ABCEvo) from EDA *software* to hardware-design *artifacts* themselves — the missing step where the agent changes the hardware engineers create, not the programs they run.

## Key points

- Git is load-bearing, not incidental: it provides the isolated evolving environment and the trace substrate; commits define accepted checkpoints and history becomes a replayable record of the agent's search.
- RTL is used as a sharp, executable proxy: correctness depends on cycle-level behavior, reset/interface conventions, and bit widths — plausible Verilog is not enough.
- Evaluated on **ChipBench, RTLLM, Verilog-Eval, and nine CVDP categories** (completion, modification, reuse, testbench stimulus, checker/assertion generation, debugging).
- Achieves **100% benchmark completion (pass rate)** across all suites with a fully hands-free agentic loop; the only residual failure traced to a known specification-harness mismatch.
- Trace analysis (token consumption, test-generation coverage) shows that once executable feedback makes correctness converge, the main bottleneck becomes **convergence efficiency and verification quality**.
- Authors explicitly caution these controlled benchmarks are proxies and do **not** claim agentic AI for hardware design is solved.

## Concepts & entities

[[code-as-harness]] · [[self-improving-agent]]

## References

- [arXiv:2606.28279](https://arxiv.org/abs/2606.28279)
