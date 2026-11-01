---
title: "LongHarness: Advancing Agentic Reasoning over 100M-Token Corpora via Stateful Evidence Management"
type: source
created: 2026-10-29
updated: 2026-10-29
sources:
  - longharness-stateful-evidence-management-huang-2026.md
arxiv_id: "2610.13075"
authors:
  - Songtao Huang
  - Zhijie Zhong
  - Xiaohan He
  - Shuaiyu Zhang
  - Zhiyin Yu
  - Li Kang
  - Heng Zhou
  - Yutao Fan
  - Xiangchao Yan
  - Bo Zhang
  - Shiyang Feng
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# LongHarness

**Songtao Huang, Zhijie Zhong, Xiaohan He, Shuaiyu Zhang, Zhiyin Yu, Li Kang, Heng Zhou, Yutao Fan, Xiangchao Yan, Bo Zhang, Shiyang Feng** (Shanghai AI Laboratory; [[fudan-university|Fudan]]; [[peking-university|PKU]]; [[shanghai-jiao-tong-university|SJTU]]; USTC; HIT) — [arXiv:2610.13075](https://arxiv.org/abs/2610.13075), v1, cs.AI, 9 Oct 2026. Preprint.

## Summary

Corpus-level QA means finding and combining evidence across a document collection far larger than the context window. The paper names two coupled failure modes of history-based agent harnesses:

- **Unbounded exploration**: retrieval keeps going without closing the decision-relevant gap.
- **Evidence-state drift**: acquired evidence or later corrections stop informing the current conclusion.

LongHarness makes the evidence–conclusion link explicit. **Evidence–Conclusion Coevolution** uses unresolved conclusions to steer the next search, and new evidence strengthens or revises the conclusions that asked for it. **Evidence Lifecycle Management** keeps the evidence store current and traceable.

## Key points

- **Benchmarks.**
  - OfficeQA Pro V1: 697 Treasury documents, about **100M tokens**.
  - OfficeQA Pro V2: 1,435 documents, with composite statistical questions.
  - AA-LCR (100 questions, context provided) and the LongBench v2 Long subset.
- **Backbones and budgets.** Two "Flash" backbones are used: GLM-5.3-Flash and Qwen3.8-Flash. LongHarness gets 200 steps, a 1M-token read budget and 60 searches. Codex and Claude Code get 200 turns and 200 tool calls.
- **Direct reasoning (BM25 into 256K context) caps out low**: the best direct model reaches 23.2% exact match on V1 and 5.6% on V2.
- **GLM-5.3-Flash, exact match.** V1 goes from **15.8% to 60.2%** and V2 from **4.4% to 61.1%**, measured against direct reasoning on the same backbone. Against the strongest competing harness, it leads Claude Code by **+12.0** on V1 and Codex by **+18.9** on V2.
- **Qwen3.8-Flash.** It reaches 61.7% on V1, +8.3 over Claude Code. On V2 it reaches 51.1%, **only +1.1 over Codex**.
- **Other benchmarks.** LongBench v2 Long: 75.9% and 76.9%, +6.5 and +7.4 over the next-best harness. AA-LCR: 84.0% and 87.0%, +5.0 and +4.0.
- **Stage breakdown (GLM).** Over Claude Code and Codex, gains are +11.1 in retrieval, +2.7 in evidence editing and +9.3 in transformation. Most of the lift is in finding evidence and turning it into answers, not in editing it.

> [!warning] Newsletter vs paper
> The numbers match. The "15.8→60.2 / 4.4→61.1" baseline is **direct BM25 reasoning, not a harness**. The harness-vs-harness margins are the +12.0 and +18.9. The V2 margin largely disappears with the second backbone (+1.1 with Qwen3.8-Flash).

## What it does not show

- **Single runs.** The main table is one run per setting because of compute limits. Three-run GLM variability appears only in Appendix D.
- **Uneven budgets.** LongHarness's read and search budgets differ in kind from the baselines' turn caps, and there is no cost or token comparison in the main results.
- **Only small backbones.** No frontier backbone is run through the harness.
- **Graded by an LLM.** AA-LCR harness answers are scored by GPT-5.6-Luna, while the direct AA-LCR scores come from Artificial Analysis.

## Concepts & entities

- [[long-context-llm]] · [[context-rot]] · [[agentic-search]] · [[longbench]] — the corpus-scale setting where a fixed window fails.
- [[agentic-harness-engineering]] · [[context-engineering]] · [[virtual-context]] — explicit evidence state as harness design.
- [[harness-1-state-externalizing-search-agents-jiang-2026]] — sibling idea: move bookkeeping state out of the transcript.
- [[harness-of-harness-multi-day-autonomous-development-yan-2026]] — evidence bundles of claim, record and status, the same principle in SWE.
- [[claude-code]] · [[bm25]] · [[retrieval-augmented-generation]]

## References

- [arXiv:2610.13075](https://arxiv.org/abs/2610.13075)
