---
title: "AutoRefine: Auto-Calibrating Code Judges from Workflow Outcomes"
type: source
created: 2026-11-17
updated: 2026-11-17
sources:
  - autorefine-shahidi-2026.md
arxiv_id: "2611.09291"
venue: arXiv preprint (cs.AI)
authors:
  - Shervin Shahidi
  - James Poon
  - Hang Xie
  - Kenneth O. Xu
  - Robin Håkanson
  - Vikram Nair
  - Jens Pillgram-Larsen
  - Nachiappan Nagappan
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# AutoRefine: Auto-Calibrating Code Judges from Workflow Outcomes

**Source:** Shervin Shahidi, James Poon (equal contribution), Hang Xie, Kenneth O. Xu, Robin Håkanson, Vikram Nair, Jens Pillgram-Larsen, Nachiappan Nagappan ([[meta|Meta Platforms]]) — [arXiv:2611.09291](https://arxiv.org/abs/2611.09291), cs.AI, v1 6 Nov 2026.

## Summary

Many agent-written diffs (refactors, performance work, cleanups) have no test oracle, so an [[llm-as-a-judge|LLM judge]] decides which ones reach human reviewers. A judge prompt must be fitted to its task and refitted whenever the agent changes, and hand labels for that are costly and go stale. Development workflows produce labels for free: tests pass or fail, and diffs land or are abandoned. These labels are noisy, though, because the outcome may depend on things the judge never sees, such as hidden tests, scope or process.

AutoRefine treats the judge prompt as parameters. Each iteration scores the judge against labels with Scott's π (chance-corrected agreement). An LLM (Claude Opus 4.7) analyses each disagreement and proposes **balanced** edits, pairing every new reason to fail with a reason to pass. An uncurated validation split picks the final prompt, with at most five iterations. Before the loop starts, an **informed teacher** sees exactly the judge's input plus the outcome. It writes a justification for each label it keeps, rarely flips a label, and **abstains** when the outcome cannot be explained from what the judge sees. A high abstention rate tells the team the judge needs better inputs, not a better prompt.

## Key points

- **Trustworthy labels.** On 417 monorepo tasks with fail-to-pass tests, a rubric judge rises from π 0.36 to 0.60 on the 85-task test split. Threshold tuning alone reaches only 0.33, or 0.42 with the best threshold picked on the test set itself. The pass/fail judge's failure recall nearly doubles, from 0.38 to 0.73.
- **Invisible failures.** Without the reference patch, the teacher abstains on 12 of 13 failures and on 0 of 72 passes. Every abstained failure lives only in hidden tests. Two human teams reviewing blind also missed nearly all of them; 54% of their failure ratings called the patch correct.
- **Label-noise test.** With 25% of labels flipped at random, the teacher sets aside 13 of 24 flipped landed diffs against 16 of 73 genuinely abandoned ones (Fisher p = 0.004), cutting noise from 24.9% to 20.4%. It catches correct diffs labeled FAIL but not failing diffs labeled PASS, by design, because landed-diff recall is the deployment metric.
- **Real merge/abandon labels** (512 diffs from a performance-optimization agent). The teacher abstains on 60% of diffs abandoned after review, 32% of drafts and 5.5% of landed diffs. Abstentions match cases where the reason for abandonment is not visible in the code.
- **Recalibration.** The new judge (v2) raises π from 0.16 to 0.51 on 17 independent held-out current diffs, and landed recall from 12/20 to 18/20 on 43 current diffs. On 16 legacy diffs the old judge stays ahead (0.50 vs. −0.02). The lesson: recalibrate whenever the agent evolves.

## Caveats

- Samples are small, each loop ran once, everything comes from one company, and judge and teacher share a model family, which risks self-preference.

## Concepts & entities

- [[llm-as-a-judge]] · [[agent-as-a-judge]] · [[prompt-optimization]] · [[dspy]] — judge calibration as prompt optimization.
- [[knowledge-distillation]] — the teacher is loosely analogous to distillation with privileged information.
- [[code-generation]] · [[verifier]] · [[outcome-based-reward]] — no-oracle code review and outcome labels.
- Related: [[judging-llm-as-a-judge-zheng-2023]] · [[gauge-when-not-to-trust-llm-as-a-judge-bodhwani-2026]] · [[reliability-without-validity-llm-as-a-judge-norman-2026]].
- [[meta]] · [[claude-opus-4-8]] (the judge and teacher model).

## References

- [arXiv:2611.09291](https://arxiv.org/abs/2611.09291)
