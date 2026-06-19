---
title: "Meta-Harness: End-to-End Optimization of Model Harnesses"
type: source
created: 2026-06-03
updated: 2026-06-03
sources:
  - meta-harness-lee-2026.md
arxiv_id: "2603.28052"
authors:
  - Yoonho Lee
  - Roshen Nair
  - Qizheng Zhang
  - Omar Khattab
  - Kangwook Lee
  - Chelsea Finn
first_author: Yoonho Lee
year: 2026
tags:
  - 2026
status: complete
importance: medium
---

# Meta-Harness: End-to-End Optimization of Model Harnesses

Surfaced via the alphaXiv / NLP-newsletter digest.

## Summary

Meta-Harness is an outer-loop system that automatically searches over *harness code* for LLM applications — the code that determines what information to store, retrieve, and present to the model. The authors argue that harnesses are still designed largely by hand, and that existing text optimizers are poorly matched to harness search because they compress feedback too aggressively (they are memoryless, condition only on scalar scores, or restrict feedback to short templates). Meta-Harness instead uses an agentic proposer that accesses the full source code, scores, and execution traces of *all* prior candidates through a filesystem, so richer access to prior experience drives the search.

Empirically the discovered harnesses are strong across three regimes: on online text classification, Meta-Harness beats a state-of-the-art context-management system by 7.7 points while using 4x fewer context tokens; on retrieval-augmented math reasoning, a single discovered harness improves accuracy on 200 IMO-level problems by 4.7 points on average across five held-out models; and on agentic coding, discovered harnesses surpass the best hand-engineered baselines on TerminalBench-2. Baselines beaten in the harness-optimizer search include ACE, OpenEvolve, and TTT-Discover. The optimized TB2 harness (MHTBA) and an interactive demo are released.

## Key points

- Frames harness design as an outer-loop search problem over code, distinct from prompt/text optimization.
- Agentic proposer reads source code + scores + execution traces of all prior candidates via a filesystem — avoiding the lossy feedback compression of memoryless text optimizers.
- Text classification: +7.7 points over a SOTA context-management system with 4x fewer context tokens.
- RAG math reasoning: a single discovered harness gives +4.7 points on 200 IMO-level problems, averaged over five held-out models (transfers across models).
- Agentic coding: discovered harnesses beat the best hand-engineered baselines on TerminalBench-2; the artifact (MHTBA, with Claude Opus 4.6) is open-sourced.
- Outperforms ACE, OpenEvolve, and TTT-Discover harness-optimizer baselines.

## Concepts & entities

- [[agentic-harness-engineering]] — Meta-Harness is a concrete automated harness-optimization system
- [[code-as-harness]] — harness treated as searchable, optimizable code
- [[llm-agent]], [[reasoning]], [[code-generation]]
- [[terminal-bench]], retrieval-augmented generation, context management
- [[stanford-university]], [[mit]] (Omar Khattab), KRAFTON (Kangwook Lee)
- [[chelsea-finn]], [[omar-khattab]]

## References

- [arXiv:2603.28052](https://arxiv.org/abs/2603.28052) — Lee et al., 2026 (v1, 30 Mar 2026).
- Project page / demo: yoonholee.com/meta-harness; artifact: github.com/stanford-iris-lab/meta-harness-tbench2-artifact.
- Baselines: ACE ([[agentic-context-engineering-zhang-2025]]), OpenEvolve, TTT-Discover.
