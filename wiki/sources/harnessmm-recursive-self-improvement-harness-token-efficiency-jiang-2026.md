---
title: "HarnessMM: Recursive Self-Improvement for Harness Token Efficiency"
type: source
created: 2026-11-10
updated: 2026-11-10
sources:
  - harnessmm-recursive-self-improvement-harness-token-efficiency-jiang-2026.md
arxiv_id: "2610.32813"
authors:
  - Yuhua Jiang
  - Chang Xu
  - Feifei Gao
  - Li Zhao
year: 2026
venue: Technical report (arXiv cs.AI)
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# HarnessMM: Recursive Self-Improvement for Harness Token Efficiency

**Source:** Yuhua Jiang, Chang Xu, Feifei Gao, Li Zhao ([[tsinghua-university|Tsinghua University]]; [[microsoft-research|Microsoft Research Asia]]) — [arXiv:2610.32813](https://arxiv.org/abs/2610.32813), cs.AI, v2 6 Nov 2026. The PDF is titled "HarnessMinusMinus: A Technical Report on Efficient Agent Harnesses"; the arXiv listing uses the title above. Code: github.com/YuhuaJiang2002/HarnessMM.

## Summary

Agent harnesses usually call the reasoning model after every tool result, even when earlier reasoning already fixed the next action (for example, "after compiling, run the tests"). **HarnessMM** stores such *resolved* decisions and executes them when feedback shows they are ready, without calling the model again. Model weights, tools and task interfaces stay unchanged. Three parts:

- **Executable Decision Graphs (EDGs)** store literal actions, arguments, dependencies and public execution guards.
- **Guarded Decision Reuse (GDR)** keeps those decisions across tool transitions and activates one when an observed receipt satisfies its guard.
- **Certified Callback Substitution (CCS)** dispatches the eligible action through the native tool interface and drops the reasoning callback it replaces.

A "Jev-motivated" typed selector picks an eligible candidate ID or abstains, decoding one token when a model call is needed. Empty sets and abstentions hand control back to the slow actor. Offline, **recursive self-improvement** tunes author prompts, context settings and adapter rules on training feedback and picks the deployed configuration on validation.

## Key points

- **HumanEval (98 test tasks).** With Codex, success holds at 90.82% while tokens per task fall from 66.72k to 47.02k (−29.54%). With DeepSeek Harness, success rises from 71.43% to 85.71% and tokens fall from 96.28k to 11.62k (−87.93%).
- **MBPP ablation.** Persistent reuse costs 26.91% less than single-use graphs and 19.52% less than keeping the callbacks.
- **The Substitution Paradox.** Adding a selector does not remove the original callback by itself. Computation is saved only when the reasoning removed exceeds the added authorship, selection and recovery. The paper therefore counts all of these in complete-task accounting.
- **Negative results are reported.** On Terminal-Bench development tasks, savings come with lower success (Codex 77.78% → 55.56%). A full CoT/Jev test comparison ran verified fast actions in 94.44% of episodes but cost 127.07% more than the original harness and lost 7.29 success points, because input tokens dominate the bill (98.53%).
- A companion visual-agent study (HarnessMM-V) is summarized.

## Limitations

Savings depend on provider input, cache and context pricing and on how often actions are already resolved. New semantic decisions still need slow reasoning. Public-state adapters are task-specific, and small development sets leave quality differences uncertain. The decision-model link is loose: Jev's typed interface motivates the selector, but the headline gains come from guarded reuse, not from a prefill-only scorer.

## Concepts & entities

- [[agentic-harness-engineering]] · [[recursive-self-improvement]] · [[recursive-agent-harnessing]] — the harness-optimization line this belongs to.
- [[system-one-model]] · [[jev]] — the typed-decision interface that motivates the selector.
- [[react]] · [[tool-use]] · [[inference-time-compute]] · [[context-engineering]] — the callback overhead being removed.
- [[humaneval]] · [[terminal-bench]] · [[openai-codex]] · [[deepseek]] — evaluation and integrated harnesses.

## References

- [arXiv:2610.32813](https://arxiv.org/abs/2610.32813)
