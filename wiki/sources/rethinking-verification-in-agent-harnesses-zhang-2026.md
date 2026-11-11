---
title: "Rethinking Verification in Agent Harnesses"
type: source
created: 2026-11-10
updated: 2026-11-10
sources:
  - rethinking-verification-in-agent-harnesses-zhang-2026.md
arxiv_id: "2610.34577"
venue: arXiv preprint (cs.AI)
authors:
  - Zhengxin Zhang
  - Erkang Zhu
  - Yin Lin
  - Claire Cardie
  - Bolin Ding
  - Jingren Zhou
year: 2026
status: complete
importance: high
tags:
  - 2026
  - paper
---

# Rethinking Verification in Agent Harnesses

**Source:** Zhengxin Zhang, Erkang Zhu, Yin Lin, Claire Cardie, Bolin Ding, Jingren Zhou (Cornell University; [[alibaba|Alibaba Group]]) — [arXiv:2610.34577](https://arxiv.org/abs/2610.34577), cs.AI, v1 25 Oct 2026. Preprint.

## Summary

Harnesses such as [[claude-code|Claude Code]], [[codex-cli|Codex CLI]] and [[gemini-cli|Gemini CLI]] all add some form of "check your work". This paper treats that checking as an axis of [[test-time-compute|test-time scaling]] and asks *when* to intervene, *who* checks, and *how* findings lead to revision. It defines four layers along the agent lifecycle:

- **L0, initial instructions**: verification guidance in the prompt.
- **L1, execution-time reminders**: periodic or after-edit nudges.
- **L2, completion gates**: when the agent stops, its work is reopened for self-checking.
- **L3, independent review**: a separate critic starts in a fresh context with access to the live sandbox, and its findings go back to the agent.

It compares 17 designs across seven frontier coding models: Qwen3.7-Max, Qwen3.8-Max, [[glm-5-2|GLM-5.2]], DeepSeek V4-Pro and V4-Flash, Kimi-K2.7-Code and MiniMax-M2.5. Qwen3.7-Plus is the critic throughout. Benchmarks are [[terminal-bench|Terminal-Bench]] 2.1 (TB) and [[swe-bench-verified|SWE-bench Verified]] (SWE), plus a new **VisualBench**: 18 visual and interactive artifact tasks (games, 3D configurators, data explorers) judged by blinded expert raters.

## Key points

- **L0/L1 are cheap but small.** They give modest gains that depend on the model (about +1 to +5 points on average) and often *reduce* output tokens. After-edit reminders that also check the rendered artifact are the recommended form.
- **L2 is poor value.** Asking the main agent to recheck its own work can help (up to +6.0 on TB), but costs up to 1.56× on SWE.
- **L3 wins on accuracy per dollar.** It adds **+5.9 to +9.5 points on TB and +3.9 to +6.6 on SWE, at 1.13–1.25× and 1.00–1.04× baseline cost.** The best design is task-first review with the agent's trajectory. On VisualBench, all four L3 designs beat the baseline by +13.2 to +23.8 net-preference points, ahead of every L0–L2 design.
- **Review scales.** More rounds (depth) or more critics (width) raise TB accuracy from 62.7% to about 66%. Four parallel critics come close to four rounds at about 19% lower total cost.
- **Practical rule:** keep system-prompt verification guidance lightweight, skip completion-time self-checking, and use a separate low-cost critic.

## Caveats

- The task subsets are selected, and there are three trials per model–task pair.
- The critic is always the same Qwen model, and the coding models are all open or Chinese-lab models served through Alibaba's DashScope.
- VisualBench is internal, with raters from the company's own data team.

## Concepts & entities

- [[agentic-harness-engineering]] · [[verifier]] · [[agent-as-a-judge]] · [[llm-as-a-judge]] — critic review as a harness layer.
- [[self-reflection]] · [[reflection-mechanism]] · [[plan-execute-verify-loop]] · [[false-success]] — self-checking (L2) vs independent checking (L3).
- [[test-time-compute]] · [[inference-time-compute]] · [[subagents]] — verification as a scaling axis.
- [[terminal-bench]] · [[swe-bench-verified]] · [[agent-evaluation]].
- [[alibaba]] · [[qwen]] · [[glm-5-2]] · [[deepseek]] · [[moonshot-ai]] · [[minimax]].

## References

- [arXiv:2610.34577](https://arxiv.org/abs/2610.34577)
