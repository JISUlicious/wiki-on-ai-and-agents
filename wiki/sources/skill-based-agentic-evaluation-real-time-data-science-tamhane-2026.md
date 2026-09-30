---
title: "Skill-based Agentic Evaluation for Real-time Data Science Tasks"
type: source
created: 2026-09-30
updated: 2026-09-30
sources:
  - skill-based-agentic-evaluation-real-time-data-science-tamhane-2026.md
arxiv_id: "2609.16487"
authors:
  - Aniruddha Tamhane
  - Raghavendra Addanki
  - Ayushi Aggarwal
  - Aditya Bansal
  - Rui Wang
  - Charles Menguy
  - Swati Jain
year: 2026
status: complete
importance: low
tags:
  - 2026
  - paper
---

# Skill-based Agentic Evaluation for Real-time Data Science Tasks

**Aniruddha Tamhane et al.** (Adobe) — [arXiv:2609.16487](https://arxiv.org/abs/2609.16487), cs.AI, 15 Sep 2026. Preprint; no venue given.

## Summary

Take a question like "what were last week's audience sizes?" Its correct answer changes as the warehouse ingests data, so any frozen reference answer goes stale, and an [[llm-as-a-judge|LLM judge]] comparing against a fixed string will be wrong. The paper's central idea is **ground-truth-as-code**: each test case stores a typed Python function `G_i`, not an answer. At evaluation time `G_i` runs against the same live data the agent sees, and so recomputes the reference. The function signature also acts as a schema contract that fails loudly when an upstream API changes.

The second idea is **format-agnostic factoid scoring**. Both the response and the computed ground truth are broken into atomic claims; the ground truth is filtered to what the query asks. The judge then scores precision (low = hallucination), recall (low = omission) and accuracy over one-to-one matches. It separately scores coverage, tone and conciseness with a rubric. The whole pipeline ships as a harness [[agent-skills|skill]] in [[claude-code|Claude Code]].

## Key points

- **Validation study.** 55 single-turn cases (53 comparable) test an in-house ML skill on two subtasks: ML-goal identification and dataset discovery. The data is a synthetic database of 353,330 events and 30,000 profiles across retail, financial services and healthcare, with injected decoy datasets and misleading column names. Three expert annotators agree well (Gwet's AC2 0.931 at factoid level, 0.883 overall).
- **Results (Table 3; MCC against the human PASS/FAIL reference):**

| Evaluator reference | MCC | F1 | Tokens/case |
|---|---|---|---|
| No ground truth (harness's skill-creator writes its own checks) | **−0.379** | 0.217 | ~70k |
| Natural-language ground truth | 0.331 | 0.697 | 29.2k |
| **Ground-truth-as-code** | **0.427** | 0.724 | ~24.6k |

- The headline "29% MCC improvement" is relative (0.427 vs 0.331). The "16% fewer tokens" matches the table (29.2k → 24.6k).
- **The self-directed baseline is anti-correlated with humans.** It graded the skill only against the skill's own declared final checks and never queried live data. This is a concrete case of an agent grading itself against its own assertions.
- The natural-language baseline called the right API but over-specified its parameters. The extra factoids that produced confused the judge and lowered precision.

## What it does not show

- **Drift resistance, the main motivation, is never tested.** The study runs on a synthetic database, not on data that changes between authoring and evaluation.
- An MCC of 0.427 is moderate agreement, and n = 53.
- Claude Sonnet 4.6 is both the skill under test and every judge operator. The authors argue that grounding in factoids limits self-preference but do not vary the judge.
- It assumes the reference code is correct and complete. Writing that code costs more expert effort than a text reference.

## Related

- [[agent-evaluation]] · [[llm-as-a-judge]] · [[verifier]] — executable references as a middle ground between code graders and LLM judges.
- [[skill-evaluation]] · [[evaluating-skills-not-just-agents-kevin-2026]] · [[agent-skills]] — evaluating a skill, and packaging the evaluator itself as a skill.
- [[autodata-agentic-data-scientist-kulikov-2026]] — the data-science-agent setting.
- [[hallucination]] — factoid precision as a direct hallucination rate.
- [[benchmark-contamination]] — the static-benchmark failure that live recomputation also sidesteps.

## References

- [arXiv:2609.16487](https://arxiv.org/abs/2609.16487)
