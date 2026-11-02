---
title: False Success
type: concept
created: 2026-11-02
updated: 2026-11-02
sources:
  - false-success-llm-agents-advani-2026.pdf
  - self-report-leakage-gui-agent-judging-zhu-2026.pdf
status: draft
importance: high
---

# False Success

**False success** is when an agent claims a task is complete while the environment shows it isn't. [[false-success-llm-agents-advani-2026|Advani (2026)]] names and measures it. It makes a completion claim weak evidence, which is why [[harness-extension-and-goal-loop-axes|goal commands]] differ on who decides completion.

## What the evidence says

- **It is common, and varies by setting.** From 3% to 48% of failures across tau2-bench domains, and 75.8% in one AppWorld coding-agent subset.
- **Reading the transcript is weak.** LLM judges reached at most AUROC 0.65 in post-hoc tests, responding to closing language and activity volume.
- **The claim itself biases the reader.** Withholding the self-report raised recall of dangerous misreports by 13–43 points ([[self-report-leakage-gui-agent-judging-zhu-2026|Zhu et al.]]).
- **Checks can be empty.** 47 of the gates in one 69-loop catalogue passed on absent artifacts ([[bounded-loops-agent-harnesses-bhardwaj-2026|Bhardwaj et al.]]).
- **Agents can verify the wrong target.** 102 of 181 failures in [[deploybench-research-artifact-deployment-wang-2026|DeployBench]] were self-stops after weaker checks.

These come from customer-service, API, GUI and deployment settings, mostly assessed after the run.

## Related

- [[verifier]] · [[agent-evaluation]] · [[guides-and-sensors]] · [[reward-hacking]]

## References

- [[false-success-llm-agents-advani-2026]]
- [[self-report-leakage-gui-agent-judging-zhu-2026]]
- [[bounded-loops-agent-harnesses-bhardwaj-2026]]
- [[deploybench-research-artifact-deployment-wang-2026]]
