---
title: "Lost in Compaction: Evaluating Side-Constraint Loss under Context Compaction"
type: source
created: 2026-08-17
updated: 2026-08-17
sources: []
arxiv_id: "2608.11242"
authors:
  - Zhiqi Wang
  - Yichi Zhang
  - Dongwon Lee
  - Yuchen Yang
year: 2026
status: complete
importance: high
tags:
  - 2026
---

# Lost in Compaction: Evaluating Side-Constraint Loss under Context Compaction

**Zhiqi Wang, Yichi Zhang, Dongwon Lee, Yuchen Yang** (The Pennsylvania State University) — [arXiv:2608.11242](https://arxiv.org/abs/2608.11242), cs.CL, dated 31 Jul 2026 on the v1 stamp. License CC BY 4.0. Code and suite at `github.com/ZhiqiEliWang/compaction-integrity`.

> [!note] Terminology
> The paper uses two names for the same object. The title and §3 say **Side Constraints**; the abstract and introduction say **Session Constraints**. Both abbreviate to SC and refer to the identical construct. This page uses "side constraint."

## Summary

Every agent harness compacts context when the window fills, and every compactor prompt is written around **task continuity** — preserve the objective, the working state, the next steps, so the dominant task can resume. The paper identifies what that design silently discards: instructions that constrain *how* the task is carried out rather than *what* it is. "Confirm with me before taking any action." "Do not delete any emails until I confirm." "Show me the draft before sending anything from now on."

The formal definition is precise and does real work. A side constraint `s` is a clause in a user prompt such that (1) `s` is not part of the task, (2) `s` is meant to constrain decoding **within the same session**, and (3) `s` has no intended use outside that session. Linguistically it is a **generic** directive (targeting a *kind* of action) rather than an **episodic** one (targeting one specific action this turn). In "Email Sarah and let her know I'll be late, but show me the draft before sending anything from now on," the first clause is the task and the second is the SC.

Those three qualifiers are exactly what makes SCs fragile. An SC is not the task, so a task-centric compactor prompt has no explicit reason to keep it. It is often stated once, phrased as a soft preference by users who do not think in prompt-engineering terms, and buried inside a task-focused turn. Unlike a system prompt it is not durable; unlike memory it is not stored anywhere. **Its survival depends entirely on the compactor.** When it is dropped, the result is a *silent integrity failure*: the agent keeps working on the right task while violating how the user asked it to proceed, and nothing in the transcript signals that anything went wrong.

**CompInt** measures this across four orthogonal axes — SC content (a five-category taxonomy: **Action**, **Information**, **Process**, **Preference**, **Output**, binding respectively what the agent does, what it emits, how it reaches an answer, which of several task-equivalent answers it picks, and textual properties of the response), framing, injection condition, and long-context environment. 15 hand-crafted SCs (3 per category) × 50 filler contexts per dataset = **N=750 instances per compaction condition**.

Three environments, each stitched/truncated to 100K tokens (~80% of a 128K window, to simulate a near-limit compaction trigger):

| Dataset | Scenario | # Tokens | # Turns | # User turns |
|---|---|---|---|---|
| WildChat | multi-turn chat | 101,121 | 519.34 | 257.22 |
| Hermes Agent | agentic tool-calling trajectory | 100,142 | 119.66 | 9.04 |
| OpenResearcher | single-turn long-horizon research | 100,481 | 310.46 | 1.00 |

Metrics: **Retention** (is the SC present in the compacted summary, LLM-as-a-judge), **Compliance** (does the model pick the compliant option on a probing MCQ), and **Effect Retention** (compaction's compliance gain normalized by the upper-bound gain over an SC-free baseline).

## Key points

**Main result** (N=750; the shaded baselines are compliance *without* compaction — "w/ SC" means the full uncompacted context including the SC, "w/o SC" means no constraint at all):

| Compactor | Retention | Compaction compliance | Upper-bound compliance | Effect retention |
|---|---|---|---|---|
| **Hermes Agent** — *w/ SC 65.1%, w/o SC 46.1%* | | | | |
| Recent-5 | 0.4% | 48.4% | 95.7% | 4.6% |
| LLMLingua-2 T500 | 0.0% | 50.1% | 99.7% | 7.5% |
| Gemma-4-E4B (Anthropic prompt) | 12.5% | 55.0% | 99.1% | 16.8% |
| Qwen3-30B (Anthropic) | 2.1% | 53.6% | 99.1% | 14.1% |
| gpt-oss-120b (Anthropic) | 18.5% | 58.5% | 98.3% | 23.8% |
| gpt-oss-120b (pi-mono) | 36.3% | 60.7% | 98.4% | 27.8% |
| GPT-5.4-mini (Anthropic) | 53.7% | 72.3% | – | – |
| GPT-5.4-mini (pi-mono) | 88.4% | 87.7% | – | – |
| **OpenResearcher** — *w/ SC 70.7%, w/o SC 49.1%* | | | | |
| Recent-5 | 0.0% | 52.0% | 95.5% | 6.3% |
| LLMLingua-2 T500 | 0.0% | 52.8% | 96.9% | 7.8% |
| Gemma-4-E4B (Anthropic) | 3.6% | 54.5% | 96.7% | 11.5% |
| Qwen3-30B (Anthropic) | 1.1% | 53.1% | 96.4% | 8.5% |
| gpt-oss-120b (Anthropic) | 8.4% | 53.9% | 94.5% | 10.6% |
| gpt-oss-120b (pi-mono) | 19.6% | 56.2% | 96.1% | 15.2% |
| GPT-5.4-mini (Anthropic) | 26.0% | 60.3% | – | – |
| GPT-5.4-mini (pi-mono) | **98.0%** | 85.7% | – | – |
| **WildChat** — *w/ SC 59.2%, w/o SC 51.2%* | | | | |
| Recent-5 | 0.0% | 53.3% | 93.5% | 5.1% |
| LLMLingua-2 T500 | 0.0% | 52.4% | 94.9% | 2.7% |
| Gemma-4-E4B (Anthropic) | 2.0% | 55.7% | 94.5% | 10.5% |
| Qwen3-30B (Anthropic) | 3.3% | 56.2% | 95.1% | 11.5% |
| gpt-oss-120b (Anthropic) | 1.3% | 54.6% | 93.6% | 8.1% |
| gpt-oss-120b (pi-mono) | 0.0% | 51.2% | 95.1% | **0.0%** |
| GPT-5.4-mini (Anthropic) | 28.3% | 64.9% | – | – |
| GPT-5.4-mini (pi-mono) | 6.7% | 56.8% | – | – |

The **17% average** is over all 24 compactor×dataset cells including both GPT-5.4-mini configurations; over the six open/non-LLM compactors alone it is closer to 6%. Several observations that matter more than the headline:

- **Non-LLM compaction is a total loss.** Recent-5 and LLMLingua-2 retain 0.0% on essentially every cell. Truncation and token-pruning have no concept of a constraint.
- **The upper bound is 93–99.7%**, so the constraints themselves are trivially followable when present. This is purely a preservation failure, not a following failure.
- **Compaction usually makes things worse than not compacting.** Compare each compaction-compliance figure against its "w/ SC" baseline: only GPT-5.4-mini ever exceeds it. Most compactors land within a few points of the **w/o SC** baseline, i.e. the constraint has almost no residual effect. gpt-oss-120b (pi-mono) on WildChat hits 51.2% compliance against a 51.2% no-constraint baseline — **0.0% effect retention**, the constraint is functionally gone.
- **Nothing is stable.** GPT-5.4-mini ranges from **98.0%** (OpenResearcher, pi-mono) to **6.7%** (WildChat, pi-mono). Which compaction prompt wins is dataset-dependent: pi-mono beats Anthropic's prompt by an average 14.5% on the lower-turn datasets, but Anthropic's wins on WildChat. There is no configuration a practitioner can pick and trust.
- Source–compactor mismatch is ruled out: when the OpenResearcher trajectory generator and the compactor are both gpt-oss-120b, retention is still 8.4%.

**Compaction geometry.** At 100K, compaction is on average **182× shorter**. Compactor output length is nearly invariant to input: a 10× input growth (10K→100K) grows the output by only **0.84×–1.28×**. Retention tracks this directly — ~90% at 10K context on Hermes Agent, decaying as context grows. Compactor ranking is stable across lengths. The obvious mitigation (compact earlier, at shorter contexts) directly defeats the purpose of compaction.

**Position and phrasing.** Retention rises **monotonically** with injection position — later is better — which the authors attribute to proximity between the SC and the compaction instruction that immediately follows the context. This **diverges from the U-shaped lost-in-the-middle pattern** reported for retrieval, and matches instead the finding that instruction-following improves when the query sits near the relevant statement. A 50K control supports the proximity hypothesis: retention improves at top and middle positions but not at bottom. Repeating the SC helps but saturates after ~30 turns.

**Framing** (Hermes Agent retention, strength × explicitness):

| Compactor | Preferential | Strict | Implicit | Direct |
|---|---|---|---|---|
| Recent-5 | 0.0% | 0.0% | 0.4% | 0.0% |
| LLMLingua-2 T500 | 0.1% | 0.0% | 0.0% | 0.0% |
| Qwen3 (Anthropic) | 0.9% | 2.0% | 2.1% | 2.5% |
| gpt-oss (Anthropic) | 9.1% | **22.0%** | 18.5% | **24.1%** |
| gpt-oss (pi-mono) | 24.4% | **38.3%** | 36.3% | 37.1% |

Framing does nothing for non-LLM compactors. For LLM compactors, "strict" (prepending "This is an important constraint:") helps more than "explicit," and combining both adds only ~1.3 pp. So user-side prompt engineering is a real but weak lever — you cannot phrase your way out of this.

**By constraint type**, **Process SCs are retained worst across every compactor**, and **no compactor exceeds 36% average retention on any type** (best: Preference under gpt-oss pi-mono, at 36% — still losing nearly two-thirds).

**The fix — an SC-aware extractor.** A training-free Qwen3.5-9B SLM reads **only user turns**, decides per prompt whether an SC is present, and maintains a running list `S^t`. At deploy time the compacted history becomes `H̃^t = C(H^t) ⊕ S^t` — the compactor is untouched, the SC is appended verbatim, reproducing the upper-bound condition.

| Dataset | Retention | Latency (s / 100K context) |
|---|---|---|
| Hermes Agent | 95.6% | 0.45 |
| OpenResearcher | 95.1% | 0.03 |
| WildChat | 90.3% | 12.93 |

Over 90% everywhere, beating GPT-5.4-mini by 62 pp on WildChat. Latency tracks user-turn count, not context size: OpenResearcher has one user turn (0.03s), WildChat has ~257 (12.93s). The paper's framing is that this requires **architectural separation** — a strategy-layer module — not a better compactor prompt. That is supported: adding an explicit SC-preservation instruction to the Anthropic prompt improves things but still leaves WildChat retention **below 40%** vs 90.3% for the extractor.

## Where it breaks

Stated limitations:

- **Proprietary context pressure untested at scale.** They could not run GPT-5.4-mini at 320K (~80% of its 400K window) because OpenAI's API errored consistently; the 220K run alone cost ~**800 USD**. Flagship models now offer 1,000K windows, and extending there is cost-prohibitive. So the closed-model column — the only one that ever performs well — is the least representative of real near-limit compaction.
- **One probing model.** Compliance is measured with a single `LLM_prob`, so Effect Retention may carry model-specific bias. (Appendix D robustness checks across three downstream models and a tool-using free-generation harness reportedly reproduce the pattern.)

Further gaps visible from the outside:

- **Constraints are injected, not naturally occurring.** 15 hand-crafted SCs concatenated onto user turns in stitched contexts. WildChat contexts are built by splicing ~128 unrelated conversations to reach 100K tokens, so "topic drift" is an artifact of the construction as much as a property of real sessions. The paper acknowledges the stitching and argues the three datasets span different coherence levels.
- **Compliance is mostly measured by MCQ probe**, choosing between a compliant and a violating option — an easier and more legible test than open-ended agentic behavior.
- **The extractor is evaluated on retention and latency, not downstream compliance or false positives.** No number is given for how often it extracts a non-constraint, nor what an ever-growing `S^t` costs in context budget over a long session — which is somewhat pointed, given the paper is about context pressure.
- **The proposed fix reintroduces the thing compaction removes.** `C(H^t) ⊕ S^t` appends uncompacted text; over a long enough session the SC registry itself becomes context that needs managing.

**Safety framing** (from Ethical Considerations, and it is the right frame): dropped session constraints mean agents "may take actions the user explicitly forbidden, including unauthorized tool calls, disclosure of withheld information, or bypassing user-required verification steps." This is a confirmation-gate failure in exactly the systems where confirmation gates matter most.

## Related

- [[prompt-compression]] — the technique family under evaluation; this paper is the negative result about what compression drops.
- [[llmlingua-2-pan-2024]] · [[llmlingua-jiang-2023]] — LLMLingua-2 is a compactor baseline here and retains **0.0%** of SCs across all three datasets.
- [[acm-agentic-context-management-li-2026]] — complementary critique of the same pipeline: ACM attacks *when* compaction fires, this paper attacks *what* it keeps.
- [[context-assembly]] · [[agentic-context-engineering]] — the layer the extractor intervenes in; the paper's thesis is that constraint preservation needs its own strategy-layer module.
- [[virtual-context]] · [[long-context-llm]] — compaction is the standard answer to finite windows; 100K contexts here sit at ~80% of a 128K window.
- [[agentic-harness-engineering]] — compaction prompts are harness artifacts, and the Anthropic vs. pi-mono gap shows harness choice is a correctness variable.
- [[openclaw]] · [[pi-mono]] · [[hermes-agent]] · [[nous-research]] · [[anthropic]] — the pi-mono compaction prompt is OpenClaw's; Hermes Agent trajectories come from the Nous Research harness; the Anthropic prompt is from the Claude API compaction docs.
- [[agent-benchmark]] · [[agent-evaluation]] — CompInt is the contribution; four controlled axes, N=750 per condition.
- [[llm-as-a-judge]] — retention is scored by LLM judge (with human-agreement/Cohen's κ validation in the appendix).
- [[guardrailing]] · [[agentic-misalignment]] — a dropped confirmation gate is a silent guardrail failure, framed by the authors as a safety issue.
- [[instruction-tuning]] — the upper bound (93–99.7% compliance when the SC is present) shows models follow these fine; the failure is upstream of following.
- [[memory-management]] · [[long-term-memory]] — the negative space of this paper: SCs are explicitly *not* durable memory, which is why nothing else catches them. A memory system that persisted session constraints would be an alternative fix.
- [[llm-agent]] — the deployment setting throughout.

## References

- [arXiv:2608.11242](https://arxiv.org/abs/2608.11242) · suite: `github.com/ZhiqiEliWang/compaction-integrity`
- Anthropic 2026, *Compaction: server-side context management for long conversations* (Claude API docs, beta header `compact-2026-01-12`).
- Pan et al. 2024, *LLMLingua-2* — the token-classification compactor baseline.
- Zhao et al. 2024, *WildChat*; Mueller 2026, *Hermes Agent*; Li et al. 2026, *OpenResearcher* — the three filler-context environments.
- Zhao et al. 2025 — proactive preference-following under long context; the nearest prior work and the source of the proximity effect this paper's positional result matches.
- Robinette et al. 2026 — verifiable long-context instruction following on *static* contexts.
- Hsieh et al. 2024; Liu et al. 2024 — the U-shaped "lost in the middle" retrieval finding that this paper's monotonic pattern diverges from.

## Related 2026 sources

- [[compaction-cliff-long-running-agent-memory-zerhoudi-2026]] — the sibling result (CIKM '26), and it **agrees on the fix**: both reject better compactor prompts and propose a cheap classifier *outside* the compactor that reinstates the constraint class verbatim, both landing in a 90–96% band. The 17% vs 53% gap is construct and pressure, not conflict — transient session constraints under 182× compression versus installed config text under 2×. **One genuine conflict**: that paper grounds part of the failure in positional bias, citing lost-in-the-middle, which this paper's monotonic-position finding rules out for this task.
