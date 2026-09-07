---
title: "Model or Harness? An Interaction-Centric Taxonomy for Localizing Agent Failures"
type: source
created: 2026-08-17
updated: 2026-08-17
sources: []
arxiv_id: "2607.28802"
year: 2026
authors:
  - Harsh Raj
  - Vipul Gupta
  - Anas Mahmoud
  - Razvan-Gabriel Dumitru
  - Darvin Yi
  - Aakash Sabharwal
  - Yunzhong He
tags:
  - 2026
  - paper
status: complete
importance: high
---

# Model or Harness? An Interaction-Centric Taxonomy for Localizing Agent Failures

**Harsh Raj, Vipul Gupta, Anas Mahmoud, Razvan-Gabriel Dumitru, Darvin Yi, Aakash Sabharwal, Yunzhong He** (Scale AI) — [arXiv:2607.28802](https://arxiv.org/abs/2607.28802), cs.AI, 30 Jul 2026. Preprint; no venue stated.

## Summary

The paper names a problem the wiki has circled repeatedly under [[harness-vs-model-attribution]]: outcome-level evaluation tells you an [[llm-agent|agent]] failed but not **which component to repair**. The authors call this the **repair-assignment problem** — "the same visible failure may call for model post-training, harness engineering, environment redesign, or benchmark repair depending on where it originated." Their running example is exact: in a long Claude Code session the agent ignores an earlier user instruction either because the harness's context compaction removed it, or because the instruction was still present and the model failed to follow it. *Identical observed behavior, opposite fixes.*

Their answer is to make the **interaction**, not the module, the unit of analysis. Every failure is written as an **edge** between two components plus a **fault side** naming the endpoint responsible:

```
comp₁ — comp₂ · fault: side
```

So a tool call that silently fails is `tool — model · fault: tool` when the wrapper suppresses the error (the model never observes it), and `tool — model · fault: model` when the wrapper surfaces the error and the model ignores it. Same edge, different repair. This is the paper's whole contribution in one line, and it is what distinguishes it from module-based taxonomies (memory / reflection / planning / action), which cannot express *who is at fault on a boundary*.

The taxonomy has **41 role-specific failure modes** and is validated with an [[agent-as-a-judge]] protocol on **40 worked examples** drawn from public benchmarks, model system cards, published reports, and logged trajectories. Four frontier judges recover the human labels well above chance; the strongest, **GPT-5.5, reaches Cohen's κ=0.76** on category labels. Crucially, the judges agree *with each other* about as strongly as with the human (peak pairwise **κ=0.84**), which is the paper's evidence that the categories capture shared structure rather than one annotator's taste.

## Key points

**Component vocabulary (Table 1)** — nine components, grouped into three families around a focal **Model** (the LLM policy):

| Family | Components | Definition highlights |
|---|---|---|
| User | Owner, Grader, Third party | Owner sets the task and defines success; **grader is deliberately separate** (the model can fail against the evaluator independently of following instructions); a third party is an actor met during execution that does *not* act for the owner |
| Harness | Context, Memory, Tool, Model (peer/subagent) | Context = information in the current interaction; Memory = persistent store outliving context; Tool = the bidirectional interface (calls, channels, wrappers) |
| Environment | Local env., External env. | OS/shell/filesystem vs. remote services, websites, APIs, model-provider infrastructure |

Two boundary decisions are worth noting. **Multi-agent is not a separate component** — since a model reaches another model through its own harness, peer/subagent interactions live on the `model — model` edge with *role* as an annotation, and are filed under the Harness family. And **third party vs. external environment** splits on actor-vs-channel: a stale response is the environment's fault, a manipulation attempt is the third party's.

**The 41 modes, by edge** (from Figure 2; the model is always one endpoint):

| Edge | # | Modes |
|---|---|---|
| model — owner | 10 | Over-initiative, Under-initiative, Satisficing, Instruction-Following Failure, Reasoning Failure, Unauthorized Irreversible Action, Sycophancy, Domain Knowledge Deficit, Value Misalignment, **Instruction–Grader Mismatch** (owner-side) |
| model — grader | 2 | Specification Gaming, Evaluation Awareness |
| model — third party | 2 | Indirect Prompt Injection, Contextual Sycophancy |
| model — context | 3 | State Tracking Failure, Goal Drift, Context Rationale Erosion |
| model — memory | 8 | Missed Write, State Staleness, Overgeneralization, Memory Rationale Erosion, Pollution, Redundancy, Missed Read, Memory Following Failure |
| model — tool | 7 | Incorrect Tool Selection, Tool Hallucination, Tool Feedback Neglect, Tool Recovery Failure, Malformed Arguments, Suboptimal Arguments, **Mistranslation** (tool-side) |
| model — model (peer / subagent) | 2 ×2 roles | Delegation Failure, Communication Failure |
| model — external env. | 3 | Recovery Failure, **Service Failure**, **Stale State Delivery** |
| model — local env. | 2 | Observation Failure, Recovery Failure |

The named modes number 39; the paper's **41** counts `model — model` modes once per role (peer and subagent), which it labels "role-specific failure modes."

**The distribution is lopsided by construction:** "Of the 41 role-specific failure modes, **36 are assigned to a model and five to surrounding components**." The authors are upfront that this reflects their attribution rule — *a failure is model-side when a more capable model could have prevented it or recovered from it* — not a measured base rate. Two modes are **dual-attribution**: Context Rationale Erosion goes to the harness when compaction is harness-driven and to the model when model-driven; external-environment Recovery Failure is model-side when recovery was still possible and environment-side when it was not.

**Root-cause rule.** A single trajectory contains cascading failures, so labels would be inconsistent without a fixed selection rule: start at the observed system-level failure, trace backward, and label **the earliest failure from which execution does not recover** — not its downstream symptoms. This is the same "critical failure / first unrecoverable event" convention used in the trajectory-localization literature (cf. `t_lock` in [[failure-as-a-process-cli-agent-trajectories-zhao-2026]]).

**Validation protocol.** Each judge gets the frozen taxonomy definitions plus a *reference to the original source* (GitHub issue, blog post, system-card section, arXiv paper, or a logged trajectory on Hugging Face / Docent) — **never the human label**. Judgment runs in three turns in one session: (1) evidence reconstruction into a neutral chronology, (2) classification of edge + fault side + mode, (3) reflection against disambiguation rules. Human labels came from a single annotator against the frozen definitions.

**Table 2 — agreement with human labels (40 worked examples):**

| Judge | Category Acc | Category F₁ | Failure-mode Acc | Failure-mode F₁ |
|---|---|---|---|---|
| GPT-5.5 | **0.80** | **0.69** | **0.72** | **0.64** |
| Claude Opus 4.6 | 0.75 | 0.61 | 0.70 | 0.57 |
| Claude Opus 4.7 | 0.75 | 0.63 | 0.62 | 0.53 |
| [[claude-opus-4-8\|Claude Opus 4.8]] | 0.75 | 0.62 | 0.68 | 0.58 |

Cohen's κ on category labels: **GPT-5.5 0.76**, Opus 4.6 and 4.7 **0.71** each, Opus 4.8 **0.70**; highest judge-judge pair **κ=0.84** (Opus 4.6 ↔ 4.8). Failure-mode κ is lower across all pairs.

**Table 3 — how much of the mode error is really a category error:**

| Judge | Predicted cat. Acc / F₁ | Gold cat. Acc / F₁ |
|---|---|---|
| GPT-5.5 | 0.72 / 0.64 | 0.72 / 0.62 |
| Claude Opus 4.6 | 0.70 / 0.57 | **0.80 / 0.70** |
| Claude Opus 4.7 | 0.62 / 0.53 | 0.70 / 0.58 |
| Claude Opus 4.8 | 0.68 / 0.58 | 0.78 / 0.69 |

Handing the Opus judges the gold category lifts mode accuracy by **8–10 points**, so a large share of their mode errors originate upstream at the category stage rather than from confusion among modes inside the right category. GPT-5.5 is flat (0.72 → 0.72) — its errors are genuinely intra-category.

**Table 4 — selective-voting ensemble** (assign a category only when ≥k of 4 judges agree, else abstain; then majority-vote the mode among the supporting judges):

| Agreement | Coverage | Cat. P | Cat. R | Cat. F₁ | Mode P | Mode R | Mode F₁ |
|---|---|---|---|---|---|---|---|
| ≥2 of 4 | 1.00 | 0.78 | 0.78 | 0.78 | 0.70 | 0.70 | 0.70 |
| ≥3 of 4 | 0.90 | 0.83 | 0.75 | 0.79 | 0.75 | 0.68 | 0.71 |
| 4 of 4 | 0.68 | **0.96** | 0.65 | 0.78 | **0.89** | 0.60 | 0.72 |

Unanimity buys **0.96 category precision at 68% coverage**. Note recall is computed over the full set while precision is over labeled examples, so F₁ is nearly flat (0.78–0.79) across thresholds — the ensemble trades coverage for precision, it does not get *better*.

**Where the judges break.** Two named sources of disagreement. First, **evidence heterogeneity**: in example E4, a public incident report attributes an agent's deletion of **more than 200 emails** to context compaction dropping a "do not act" instruction, but publishes no trajectory — from the source alone it reads equally well as a context-side failure or a model-side unauthorized action. Second, **causal reconstruction is genuinely hard**: in the Appendix A.2 case study, a scripted reply email never arrives because of an *evaluation-environment bug*, and the judge blames the model for not checking rather than tracing back to the undelivered email. The authors connect this to OpenRCA 2.0's finding that frontier models often fail to reconstruct a verified causal path and produce an "ungrounded diagnosis."

**Stated limitations.** (1) The taxonomy is **descriptive, not quantitative** — it assigns responsibility but estimates no frequencies, and the 40 examples are explicitly "illustrative rather than exhaustive and should not be used to estimate the prevalence of individual failure modes." (2) It is derived from the cases reviewed and will need to expand as harnesses evolve. (3) Labels depend on available evidence; brief reports and system cards often omit what is needed for a unique root cause. (4) The agent-as-a-judge validation "may be difficult to deploy in production because judge accuracy remains limited, especially for failure-mode labels" — and ensembling buys precision only by abstaining exactly where attribution is most uncertain.

> [!note] Scope note
> Every edge in this taxonomy has the *model* as one endpoint. Harness-internal failures with no model involvement (e.g. a memory-compaction job corrupting a store the model never reads) have no home here, and the model-centric framing is part of why 36 of 41 modes land on the model side.

## Related

- [[harness-vs-model-attribution]] — the wiki's standing comparison page on separating scaffold from model; this paper supplies the *per-failure* attribution vocabulary that the factorial leaderboard studies only supply in aggregate.
- [[agentic-harness-engineering]] — the taxonomy's practical payoff: harness-side labels are precisely the input an evolver loop needs to decide what to edit.
- [[fundamental-components-of-harness]] — an adjacent component decomposition; compare Context / Memory / Tool / sub-agent against this paper's Harness family.
- [[agent-three-layer-model]] — a coarser model/harness/environment stratification of the same system.
- [[agent-as-a-judge]] and [[llm-as-a-judge]] — the validation instrument; note the three-turn evidence-reconstruction pipeline rather than dropping the trace in the judge's context.
- [[agent-evaluation]] — the motivating gap: outcome-level scoring is insufficient for improvement.
- [[failure-as-a-process-cli-agent-trajectories-zhao-2026]] — complementary *temporal* decomposition (decisive error, lock point, first observable sign); shares the "earliest unrecoverable failure" convention.
- [[tracing-agentic-failure-flow-of-success-yeh-2026]] — failure *attribution within a trajectory* (which step); this paper answers the orthogonal question of which component and which side.
- [[critique-of-agent-model-xing-2026]] — related argument that model-level framing obscures system-level causes.
- [[agent-harness-survey-meng-2026]] — survey context for the harness component vocabulary.
- [[stop-comparing-llm-agents-without-disclosing-the-harness-zhang-2026]] — the benchmarking-side consequence of unattributed failures.
- [[memory-management]] — the `model — memory` edge is the largest non-owner edge (8 modes), split into write failures and read failures.
- [[tool-use]] — the `model — tool` edge (7 modes), with **Mistranslation** the sole tool-side fault.
- [[indirect-prompt-injection]] and [[sycophancy]] — both appear as named modes on the `model — third party` edge (the latter as *Contextual Sycophancy*).
- [[reward-hacking]] — the paper's *Specification Gaming* mode on the `model — grader` edge; grounded in the chess-engine case (E12) where the agent edited board state until the opponent resigned.
- [[multi-agent]] — Delegation and Communication Failure, distinguished by peer vs. subagent role.
- [[claude-code]], [[openclaw]], [[hermes-agent]] — cited as the reference architectures the schema is claimed to cover.

## References

- [arXiv:2607.28802](https://arxiv.org/abs/2607.28802)
