---
title: "Single-Agent LLMs Outperform Multi-Agent Systems on Multi-Hop Reasoning Under Equal Thinking Token Budgets"
type: source
created: 2026-06-03
updated: 2026-06-03
sources:
  - single-agent-vs-multi-agent-tran-2026.md
arxiv_id: "2604.02460"
authors:
  - Dat Tran
  - Douwe Kiela
first_author: Dat Tran
year: 2026
tags:
  - 2026
status: complete
importance: medium
---

# Single-Agent LLMs Outperform Multi-Agent Systems on Multi-Hop Reasoning Under Equal Thinking Token Budgets

Surfaced via the alphaXiv / NLP-newsletter digest.

## Summary

Tran and Kiela (Stanford) argue that the much-reported gains of multi-agent LLM systems (MAS) over single-agent systems (SAS) are largely an artifact of unaccounted test-time computation. They give an information-theoretic argument grounded in the **Data Processing Inequality**: under a fixed reasoning-token budget and perfect context utilization, a single agent is more information-efficient, because multi-agent decomposition introduces communication bottlenecks that lose information. The framing also predicts *when* MAS should win — when a single agent's effective context utilization is degraded (long/noisy contexts) or when MAS quietly spend more compute.

Their controlled study spans three model families (Qwen3, DeepSeek-R1-Distill-Llama, Gemini 2.5) and several MAS architectures, all matched on **thinking-token budget** (intermediate reasoning tokens, excluding prompts/answers). SAS consistently match or beat MAS on multi-hop reasoning under equal budgets. A diagnostic pass also surfaces measurement artifacts: API-based budget control (notably Gemini 2.5) and standard benchmark quirks can both inflate apparent MAS gains. Conclusion: many reported MAS advantages reflect compute and context effects, not inherent architectural benefit.

## Key points

- Core claim: at equal thinking-token budgets, SAS match or outperform MAS on multi-hop reasoning across three model families.
- Theory: Data Processing Inequality implies multi-agent decomposition adds communication bottlenecks → information loss under fixed budgets.
- Predicts MAS competitiveness only when single-agent context utilization degrades or when MAS use extra (uncontrolled) compute.
- Identifies artifacts in API budget control (Gemini 2.5) and in benchmarks that inflate MAS results.
- Defines "thinking token budget" as intermediate reasoning tokens, excluding prompts and final answers — a fairer normalization axis.
- Calls for explicitly controlling the compute / context / coordination trade-offs when evaluating agentic systems.

## Concepts & entities

[[multi-agent-memory]] · [[llm-agent]] · [[chain-of-thought]] · [[inference-time-compute]] · [[long-context-llm]] · [[deepseek-r1]] · [[deepseek]] · [[stanford-university]]

## References

- arXiv: [arXiv:2604.02460](https://arxiv.org/abs/2604.02460)
