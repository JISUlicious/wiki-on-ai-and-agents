---
title: SWE-bench
type: concept
created: 2026-05-21
updated: 2026-11-02
sources:
  - swe-bench-jimenez-2023.md
status: draft
importance: high
tags:
  - 2023
---

# SWE-bench

**SWE-bench** is the foundational coding-agent benchmark introduced by Jimenez et al. (2023). It contains 2,294 real GitHub issues drawn from 12 popular Python repositories, paired with the corresponding fix commits and passing unit tests, so an agent's patch can be graded by executing the test suite.

## How it works

- **Task:** given the issue text and a repository snapshot, produce a patch.
- **Pass condition:** at least one fail-to-pass test flips, and previously passing tests keep passing.
- **Construction:** about 90,000 candidate pull requests are scraped, then filtered by attributes and by execution.
- **Scale:** codebases average 3,010 non-test files and 438K lines. The reference patch averages 1.7 files and 32.8 lines, so finding the right place is much of the task.

## Variants

| Variant | Size | Purpose |
|---|---|---|
| SWE-bench | 2,294 | The full set |
| SWE-bench Lite | 300 | Cheaper, self-contained subset |
| [[swe-bench-verified]] | 500 | Human-filtered subset that most papers report |
| [[swe-bench-live]] | 1,319 at release | Monthly refresh, to resist contamination |
| [[swe-gym]] | Separate | A training environment, not a test set |

## Reading the scores

- **The rise.** At release the best BM25 baseline solved under 4%. [[swe-bench-verified|Verified]] scores are now reported around 70–80%, on a different subset and with agents rather than single-shot patching, so the two aren't directly comparable.
- **Harness effects.** Scores on Verified can move 11–15 points with the scaffold alone, which is more than many claimed model gains. See [[stop-comparing-llm-agents-without-disclosing-the-harness-zhang-2026]] and [[harness-vs-model-attribution]].
- **Contamination.** The same OpenHands and Claude 3.7 Sonnet setup scored 43.20% on Verified and 19.25% on [[swe-bench-live]]. The tasks differ too, so this suggests contamination without isolating it.

Related: [[swe-bench-verified]], [[swe-bench-live]], [[swe-gym]], [[llm-agent]], [[agentic-rl]], [[benchmark]]

## References
- [[swe-bench-jimenez-2023]]

## Related 2026 sources

- [[swe-refactor-bench-hong-2026]] — the long-horizon counterpart: whole-repository stack migrations, where **28 of 520 runs (5.4%) pass all three stages** and 13 of 20 tasks receive no accepted solution — against frontier resolution rates in the 70s on this benchmark
- [[frognano-training-a-4b-coding-agent-via-online-task-synthesis-kim-2026]] — 61.5% on Verified from a 4B model trained only with RL on synthetic tasks — but Verified is also its validation set
- [[lego-rl-harness-native-reinforcement-learning-coding-agents-du-2026]] — harness-native RL results under three real harnesses
