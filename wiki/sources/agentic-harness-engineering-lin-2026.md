---
title: "Agentic Harness Engineering: Observability-Driven Automatic Evolution of Coding-Agent Harnesses"
type: source
created: 2026-05-25
updated: 2026-05-25
sources:
  - agentic-harness-engineering-lin-2026.md
arxiv_id: "2604.25850"
authors:
  - Jiahang Lin
  - Shichun Liu
  - Chengjun Pan
  - Lizhi Lin
  - Shihan Dou
  - Zhiheng Xi
  - Xuanjing Huang
  - Hang Yan
  - Zhenhua Han
  - Tao Gui
  - Yu-Gang Jiang
first_author: Jiahang Lin
year: 2026
venue: arXiv preprint
introduces:
  - "[[agentic-harness-engineering]]"
tags:
  - 2026
  - paper
status: complete
importance: high
---

# Agentic Harness Engineering: Observability-Driven Automatic Evolution of Coding-Agent Harnesses

[arXiv:2604.25850](https://arxiv.org/abs/2604.25850) — Lin et al., April 2026 (v4: 18 May 2026). [[fudan-university]], [[peking-university]], Shanghai Qiji Zhifeng Co., Ltd.

## Summary

This paper formally names and operationalizes [[agentic-harness-engineering]] (AHE): a closed-loop system in which one agent continuously evolves another coding agent's harness — the model-external scaffold of system prompts, tools, middleware, skills, sub-agents, and long-term memory — driven by telemetry from rollouts. The authors argue that as base models advance rapidly, the manual loop of inspecting trajectories and hand-tuning harnesses cannot keep pace, creating a widening gap between model capability and the harness needed to realize it. AHE addresses this with three matched "observability pillars": component observability (every editable harness piece is a file at a fixed mount point), experience observability (millions of raw trajectory tokens distilled into a layered, drill-down evidence corpus via an Agent Debugger), and decision observability (every edit ships with a self-declared prediction that the next round verifies against task-level deltas, making each edit a falsifiable contract).

Empirically, ten AHE iterations starting from a deliberately minimal bash-only seed lift pass@1 on Terminal-Bench 2 from 69.7% to 77.0%, surpassing the human-designed Codex (71.9%) and self-evolving baselines ACE and Training-Free GRPO. The frozen evolved harness transfers without re-evolution: on SWE-bench-verified it tops aggregate success while using 12% fewer tokens than the seed, and cross-family it yields +5.1 to +10.1 pp gains across DeepSeek-V4, Qwen-3.6, and Gemini-3.1 bases. Ablations localize the gain to tools, middleware, and long-term memory — the system-prompt-only swap is the lone regression, evidence that "factual harness structure transfers across tasks and models whereas prose-level strategy does not."

This is the canonical reference work cited by the [[code-as-agent-harness-ning-2026]] survey's §3.5 chapter on harness evolution, and it represents the 2026 frontier of self-improving agent infrastructure: an empirical, harness-level descendant of the [[darwin-godel-machine]] paradigm. Where DGM evolves the agent itself, AHE freezes the base model and evolves only the surrounding scaffold — a more tractable and (the results suggest) more transferable target.

## Key Points

- **Observability is the bottleneck, not agent capability.** The paper's central claim: "once the evolution agent receives structured context over a clear action space, it can reliably converge on better harness designs." The three observability pillars are the contribution; the evolver itself is just a competent coding agent.
- **Deep telemetry as optimization substrate.** The Agent Debugger turns ~10M tokens of raw trajectories into ~10K tokens of layered evidence: per-task root-cause analyses, benchmark-level overviews, and progressively disclosed raw traces. This is what the [[code-as-agent-harness-ning-2026]] survey labels "deep telemetry."
- **Evolution Agent proposes harness mutations.** Each round, the Evolve Agent reads the evidence corpus, edits files in the NexAU workspace (seven decoupled component types: system prompt, tool description, tool implementation, middleware, skill, sub-agent config, long-term memory), and records a change manifest.
- **Governed mutation via falsifiable contracts.** Every edit is paired with a self-declared prediction (expected fixes + at-risk regressions). The next round's task-level deltas adjudicate the prediction; failed contracts are reverted at file granularity via git. The Evolve Agent is sandboxed — it cannot touch the verifier, tracer, or model config.
- **Empirical descendant of [[darwin-godel-machine]] applied at the harness level, not the agent level.** Like DGM, AHE is a self-improving system that mutates code based on benchmark feedback. Unlike DGM, the base model is held fixed; only the harness is mutable. This isolation makes attribution cleaner and (as the cross-model results show) the evolved artifact transferable.
- **Pairs naturally with [[self-improving-agent]].** AHE realizes the "self-improving infrastructure" arm of the self-improving-agent design space: the system that runs the agent learns from running it, even when the agent's weights do not.
- **The minimal seed matters.** H₀ is deliberately bash-only with no middleware, skills, or sub-agents. A seed pre-fitted to the benchmark would contaminate attribution: every gain must be earned against measured rollouts.
- **Where the gain lives.** Component-level ablation (Table 3): +memory-only 75.3%, +tool-only 73.0%, +middleware-only 71.9%, +system-prompt-only 67.4% (regresses). Single-component variants can overshoot full AHE — components interact non-additively, capping aggregate gain.
- **Cross-family transfer favors weaker bases.** Gains scale inversely with base saturation: +10.1 pp on DeepSeek-V4-flash, +6.3 on Qwen-3.6-plus, +5.1 on Gemini-3.1-flash-lite, vs. +2.3 on GPT-5.4 medium/xhigh. Less-saturated bases lean more on the coordination patterns the harness encodes.
- **Known limits.** Self-attribution is reliable for fixes but blind to regressions (the loop can spot what it improved, not what it silently broke); the timeout/step budget is fitted during evolution and creates a generalization hazard.

## Entities

- [[jiahang-lin]] (first author, Fudan University, intern at Shanghai Qiji Zhifeng)
- [[shichun-liu]], [[chengjun-pan]] (co-first authors)
- [[tao-gui]], [[zhenhua-han]] (corresponding authors)
- [[xuanjing-huang]], [[yu-gang-jiang]] (Fudan PIs)
- [[fudan-university]], [[peking-university]] (institutions)
- [[terminal-bench]] (primary evaluation, v2)
- [[swe-bench]] (transfer evaluation, verified split)
- [[gpt-5]] (GPT-5.4 high used as base for all three role agents)

## Concepts

- [[agentic-harness-engineering]] — this paper introduces and names the concept
- [[code-as-harness]] — the broader frame; AHE makes the harness itself agent-editable
- [[self-improving-agent]] — AHE is a concrete instance at the harness level
- [[darwin-godel-machine]] — direct intellectual ancestor; AHE narrows the mutation target from agent to harness
- [[agent-three-layer-model]] — AHE evolves all three layers (prompt = strategy, tools/middleware = harness, memory = state)
- [[observability]] — the paper's central organizing principle
- [[coding-agent]]
- [[long-term-memory]] (one of the four evolved components)
- [[middleware]]
- [[telemetry-driven-evolution]]

## Notable Quotes

> "Harnesses are now central to agent performance, mediating how models interact with tools and execution environments. Yet harness engineering remains a manual craft." (Abstract)

> "Our central insight is that this question is bottlenecked by observability, not by agent capability: once the evolution agent receives structured context over a clear action space, it can reliably converge on better harness designs." (§1)

> "Every edit becomes a falsifiable, file-level claim recorded in a versioned manifest, and the next round's verdict either confirms or reverts it." (§3.3)

> "Factual harness structure transfers across tasks and models whereas prose-level strategy does not." (§4, on the ablation result that system-prompt-only regresses while tools/middleware/memory each carry gains)

> "Bases further from saturation lean more on the coordination patterns AHE has fixed inside tools, middleware, and long-term memory, while a stronger base re-derives the same coordination from its prompt at low marginal cost." (§4.3)

> "Two limits of agent-driven evolution: harness components interact non-additively, so stacking effective edits caps the aggregate gain; and the loop's self-attribution is reliable for fixes but blind to regressions, pinpointing regression foresight as the clearest direction for future self-evolution loops." (§1, contributions)

## References

- **Survey context**: [[code-as-agent-harness-ning-2026]] — the 2026 survey of code-as-harness systems whose entire §3.5 chapter is built on this paper, positioning AHE as the canonical reference for harness-level self-evolution.
- **Direct intellectual ancestor**: [[darwin-godel-machine]] — Zhang et al.'s DGM evolves agents themselves; AHE narrows the same closed-loop-evolution principle to the harness substrate.
- **Sibling self-evolution methods (baselines beaten in the paper)**: ACE (Agentic Context Engineering, [[agentic-context-engineering-zhang-2025]]) — prompt/playbook-only evolution; Training-Free GRPO — trajectory-feedback variant. AHE's gain over both is attributed to evolving tools/middleware/memory, the layers ACE and TF-GRPO leave untouched.
- **Benchmarks**: Terminal-Bench 2 (primary), SWE-bench-verified (transfer).
- **Substrate**: NexAU harness framework (refs [23, 37] in the paper) — the file-level, decoupled component substrate that makes component observability possible.
- **Trajectory analysis**: Agent Debugger (ref [17]) — the framework AHE uses to turn raw rollouts into the layered evidence corpus.
