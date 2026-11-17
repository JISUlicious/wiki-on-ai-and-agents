---
title: "LongCapHarness: Capability-Guided Auto-Harnessing for Long-Context Reasoning"
type: source
created: 2026-11-17
updated: 2026-11-17
sources:
  - longcapharness-zhang-2026.md
arxiv_id: "2611.10463"
venue: arXiv preprint (cs.CL)
authors:
  - Tao Zhang
  - Yuhang He
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# LongCapHarness: Capability-Guided Auto-Harnessing for Long-Context Reasoning

**Source:** Tao Zhang, Yuhang He ([[microsoft-research|Microsoft Research Asia]]) — [arXiv:2611.10463](https://arxiv.org/abs/2611.10463), cs.CL, v1 7 Nov 2026.

## Summary

The usual way to make a model good at long context is to synthesize long training data and post-train on it (NExtLong, PolicyLong, EntropyLong). LongCapHarness does neither. A strong harnessing agent (GPT-5.6-Sol) reads a development set, names the **capability** each task needs (for example "reranking"), splits it into **capability primitives** with typed inputs and outputs (relevance judgement, ordering), and implements each primitive as either code or a bounded LLM call. It then runs a recursive self-improvement loop that revises the primitive implementations, the control logic and the shared capability state until the harness's answers match the references. The finished harnesses are frozen into a library. At test time a small execution model (Llama3-8B with an 8K window, or Qwen2.5-3B) picks a harness by its text description and runs it. Results stay outside the model's context, so total input length is decoupled from any single call's budget.

## Key points

- **Main result (HELMET + RULER, Table 1).** Llama3-8B with the harness averages 79.32, against 70.15 for the 128K-trained NExtLong model, 39.67 for [[rrsi-regularized-recursive-self-improvement-agent-harnesses-xia-2026|RRSI]] and 15.14 for the bare 8K model. Qwen2.5-3B scores 78.73 vs. 64.10 for PolicyLong and 38.46 for RRSI.
- **Where it wins.** Recall reaches 100.00 with both models at every length from 8K to 128K, and [[ruler|RULER]] reaches 94.62 / 94.28. These tasks reduce to primitives with explicit rules (keyword matching, lookup, relation following, state tracking, counting) that code can execute at any length.
- **Where it doesn't.** On RAG it trails RRSI by 4.93 (Llama) and 0.55 (Qwen) points, and ICL and LongQA gains are mixed. The authors' reading: the harness organizes semantic steps but does not replace the model's judgement. On LongBench-v2, Qwen beats PolicyLong on the Medium subset (34.47 vs. 27.10) but loses on Long, 128K–2M tokens (27.85 vs. 31.50).
- **Primitive evolution.** For HELMET reranking, moving from BM25 + sort (V0, 40.35) to LLM scoring + sort (V1) to pairwise LLM comparison + group merge (V2) lifts Qwen to 56.28, +15.78 over V1 and +26.75 at 128K.
- **Cross-model transfer.** Swapping Llama3-8B for Qwen2.5-3B costs only 0.59 average points. Code primitives transfer unchanged; semantic primitives move with the execution model (reranking +16.61, LongQA −10.39).
- **Cost.** Development uses 814M input tokens, about $541. That is more than RRSI ($332) but below the estimated training cost of NExtLong ($3,696) and PolicyLong ($1,859). At inference, output tokens fall from 89.55M (RRSI) to 1.87M on Llama.

## Caveats

- The harness is built by a frontier model. When Llama3-8B or Qwen2.5-3B act as their own harnessing agent, performance drops sharply.
- Figure 1 reports 79.97 / 77.91, slightly different from the Table 1 averages (79.32 / 78.73). The numbers above follow Table 1.
- The dev split is drawn from every evaluated task type (2,242 dev vs. 59,826 test examples), so harnesses are task-specific rather than zero-shot.

## Concepts & entities

- [[long-context-llm]] · [[ruler]] · [[longbench]] · [[bm25]] — the problem setting and the benchmarks.
- [[agentic-harness-engineering]] · [[code-as-harness]] · [[recursive-self-improvement]] · [[recursive-agent-harnessing]] — how the harnesses are built.
- Related harness work: [[rrsi-regularized-recursive-self-improvement-agent-harnesses-xia-2026]] · [[meta-harness-lee-2026]] · [[autoharness-lou-2026]] · [[agentic-harness-engineering-lin-2026]] · [[longharness-stateful-evidence-management-huang-2026]].
- [[microsoft-research]] · [[llama-3]] · [[qwen-2-5]].

## References

- [arXiv:2611.10463](https://arxiv.org/abs/2611.10463)
