---
title: "From Static Templates to Dynamic Runtime Graphs: A Survey of Workflow Optimization for LLM Agents"
type: source
created: 2026-06-03
updated: 2026-06-03
sources:
  - survey-workflow-optimization-llm-agents-yue-2026.md
arxiv_id: "2603.22386"
authors:
  - Ling Yue
  - Kushal Raj Bhandari
  - Ching-Yun Ko
  - Dhaval Patel
  - Shuxin Lin
  - Nianjun Zhou
  - Jianxi Gao
  - Pin-Yu Chen
  - Shaowu Pan
first_author: Ling Yue
year: 2026
tags:
  - 2026
status: complete
importance: medium
---

# From Static Templates to Dynamic Runtime Graphs: A Survey of Workflow Optimization for LLM Agents

Surfaced via the alphaXiv / NLP-newsletter digest.

[arXiv:2603.22386](https://arxiv.org/abs/2603.22386) — Yue, Bhandari, Ko, Patel, Lin, Zhou, Gao, Chen, Pan (RPI & IBM Research), 2026.

## Summary

This survey reviews methods for designing and optimizing the executable workflows that LLM-based systems build to solve tasks — workflows that interleave LLM calls, retrieval, tool use, code execution, memory updates, and verification. The authors frame all of these as **agentic computation graphs (ACGs)** and organize the literature primarily by *when* workflow structure is determined: **static** methods fix a reusable scaffold before deployment, while **dynamic** methods select, generate, or revise the workflow for a particular run, before or during execution.

Beyond the static/dynamic axis, the survey adds two more organizing dimensions — *what* part of the workflow is optimized, and *which evaluation signals* guide optimization (task metrics, verifier signals, preferences, or trace-derived feedback) — and draws a sharp distinction between reusable **templates**, run-specific **realized graphs**, and **execution traces**. It closes with a structure-aware evaluation perspective that supplements downstream task metrics with graph-level properties, execution cost, robustness, and structural variation across inputs, aiming to give the field a common vocabulary and a more reproducible evaluation standard.

## Key points

- Treats LLM-agent workflows uniformly as **agentic computation graphs (ACGs)**: who is present, how components depend on each other, and how information flows.
- Primary taxonomy axis is **when structure is decided** — static (pre-deployment scaffold) vs. dynamic (per-run selection/generation/revision).
- Two further axes: *what* is optimized within the workflow, and *which evaluation signals* drive optimization.
- Separates three artifacts: reusable **template**, run-specific **realized graph**, and runtime **execution trace** — to disentangle design choices from deployed structure and actual behavior.
- Advocates **structure-aware evaluation**: graph-level properties, cost, robustness, and input-dependent structural variation alongside task accuracy.
- Companion repo: `github.com/IBM/awesome-agentic-workflow-optimization`.

## Concepts & entities

- [[llm-agent]] — Survey scope is workflow optimization for LLM agents.
- [[multi-agent-memory|multi-agent]] — ACGs include multi-agent topologies and inter-agent information flow.
- [[code-as-harness]] — Workflows interleave code execution and verification as graph nodes.
- [[agentic-context-engineering]] — Related to optimizing what flows through the agent's workflow.
- [[pin-yu-chen|Pin-Yu Chen]] — Corresponding author (IBM Research).

## References

- IBM awesome-agentic-workflow-optimization repository (curated bibliography).
- Surveys prior static workflow scaffolds and dynamic graph-generation/revision methods.
