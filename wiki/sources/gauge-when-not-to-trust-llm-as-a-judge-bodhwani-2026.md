---
title: "GAUGE: When Not to Trust LLM-as-a-Judge in User-Simulated Evaluation of Task-Oriented Agents"
type: source
created: 2026-09-30
updated: 2026-09-30
sources:
  - gauge-when-not-to-trust-llm-as-a-judge-bodhwani-2026.md
arxiv_id: "2609.12191"
authors:
  - Umesh Bodhwani
  - Thanh Tran
  - Kai Wei
year: 2026
status: complete
importance: high
tags:
  - 2026
  - paper
---

# GAUGE: When Not to Trust LLM-as-a-Judge in User-Simulated Evaluation of Task-Oriented Agents

**Umesh Bodhwani, Thanh Tran, Kai Wei** ([[amazon|Amazon]]) — [arXiv:2609.12191](https://arxiv.org/abs/2609.12191), cs.CL, 10 Sep 2026. Preprint; no venue given.

## Summary

The standard offline release gate for customer-service agents works like this. Persona-driven LLM user simulators talk to each candidate, an [[llm-as-a-judge|LLM judge]] scores the transcripts, and the higher-scoring variant ships. **GAUGE** (Grounded Audit of User-simulator-and-judge Gate Evaluation) audits that gate against a **non-LLM verifiable reward**: the [[tau-bench|τ²-bench]] DB-state and action oracle, plus SimulatorArena's human-graded correctness. The study covers 25 agent configurations from 14 models and six providers, four judges and about 3,700 transcripts.

It separates two kinds of validity that release practice conflates. **Ranking validity** is strong: the gate orders agents like the reward (ρ = 0.94). **Construct validity** fails: the satisfaction the gate is certified against carries almost no information about task success. A gate can be *human-validated yet mis-anchored*.

## Key points

- **Satisfaction–success gap.** A blind three-person panel rated 150 transcripts (Krippendorff's α = 0.79). **57.5% of conversations rated satisfied (≥5/7) had failed the task**, which is statistically the same as the sample's 57.3% base failure rate. Correlation with success is ρ = −0.147 and transcript-level AUC is 0.44. All five subjective dimensions (respect, clarity, helpfulness, would-return) show |ρ| ≤ 0.17. The LLM judges reproduce human satisfaction well (Opus-4.8 ρ = 0.846, GPT-5.5 ρ = 0.827), which is exactly why human alignment does not certify them.
- **Evidence access, not capability, decides.** A *policy-aware* gate that reads tool calls and the goal halves failure risk (20.0% of accepted conversations fail, against a 40.2% base rate; AUC 0.73). A *process-blind* proxy behaves like satisfaction (32.7%; AUC 0.49). Truncating a transcript mid-task gave reward 0.0 but the highest human rating of any degraded variant (4.6/7).
- **Resolution limit.** On near-equal pairs (reward gap < 0.1, which is 30% of all pairs and 55% of top-half pairs) the gate **promotes the lower-reward agent 31% of the time**, against < 1% (0.9%) on wide pairs. Every judge does the same (30–39%). Averaging four judges reproduces the same 31% (27/87), because they flip the same pairs. Only calibrated abstention helps, reducing the error to 14.8%.
- **Self-preference.** An Opus-4.8 judge inflates Claude agents by +0.75/7 relative to GPT-5.5 (a diff-in-diff), without breaking the ranking.
- **Personas.** A persona population cuts mean reward by 42% (0.47 → 0.27) compared with τ²-bench's single cooperative user.
- **Free tripwire.** A judge-free completion bit matches the judge on broken-vs-working agents (ρ 0.87 vs 0.80). But only 3.5% of 1,485 grid failures are truncations, so on the full grid it collapses to ρ 0.51 against the gate's 0.94.
- **Negative result.** Out-of-sample recalibration does not transfer: persona reweighting goes 0.71 → 0.80 in-sample and 0.68 leave-one-out. Hence the recommendation is **calibrate-then-trust**: run the verifiable audit on a trigger, then trust the cheap gate only inside the region the audit validated.

## What it does not show

- The human panel is small: 3 raters and 150 transcripts.
- τ²-bench instructions are synthetic, and absolute pass rates are not leaderboard-comparable because of a tolerant parser on Bedrock.
- The degradation set uses only budget-style perturbations, so it tests truncation, not every failure mode. Live customer traffic is untested.

> [!warning] Qualifies [[llm-as-a-judge]]
> That page presents judge alignment as *"the practice that separates working judges from decorative ones"*, measured as *"alignment score = % agreement with human experts."* GAUGE shows that aligning to human *satisfaction* can certify a judge that is anchored to the wrong construct. Human labels must themselves be grounded in a verifiable outcome.

## Related

- [[llm-as-a-judge]] · [[agent-as-a-judge]] · [[reliability-without-validity-llm-as-a-judge-norman-2026]] — the same "reliable but not valid" theme, applied to agents at the release-decision level.
- [[tau-bench]] · [[agent-evaluation]] · [[agent-benchmark]] — the substrate and the evaluation practice being audited.
- [[reward-hacking]] — the Goodhart argument against optimizing a satisfaction gate.
- [[sycophancy]] — satisfied-but-failed conversations are what an agreeable agent produces.
- [[xyeval-agents-say-yes-to-bad-advice-wu-2026]] — the same τ²-bench substrate, showing agents complying with users instead of solving the task.

## References

- [arXiv:2609.12191](https://arxiv.org/abs/2609.12191)
