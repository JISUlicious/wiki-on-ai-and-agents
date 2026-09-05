---
title: Skill Selection
type: concept
created: 2026-09-05
updated: 2026-09-05
sources:
  - demystifying-agent-skills-jiang-2026.md
  - evaluating-skills-not-just-agents-kevin-2026.md
  - recuris-experiential-working-memory-evolution-yu-2026.md
builds-on:
  - [[skill-routing]]
status: complete
importance: medium
---

# Skill Selection

Three different things get called "the agent used a skill," and they diverge sharply enough that conflating them makes results incomparable:

1. **Retrieved** — the skill entered the context window.
2. **Selected** — the model chose it over the alternatives present.
3. **Actually used** — the skill's procedure shaped the trajectory.

[[demystifying-agent-skills-jiang-2026|Jiang et al.]]'s three experimental arms separate these and **disagree by roughly 70 points at a pool size of 100**. A library can look healthy on retrieval and be nearly inert on use.

The gap matters because interventions land on different stages. A better index improves *retrieval*. A better description improves *selection*. Neither guarantees the third — [[demystifying-agent-skills-jiang-2026|the same paper]] finds skills work mainly by **procedural anchoring (65.7% of helpful cases)** rather than knowledge injection (4.5%), so a skill that is selected but whose procedure the model does not follow has been retrieved, selected, and wasted.

## Measuring the right stage

[[evaluating-skills-not-just-agents-kevin-2026|ACES]]'s paired design measures the third stage directly: run the identical task with and without the skill loaded, under the same model, sandbox, workspace, and scorer, and take the difference in what the agent completed. That sidesteps the retrieval/selection question entirely by measuring the outcome, which is why its **mean composite Skill Lift of 0.2134** concentrates in *process* metrics — skill execution +0.3263, behaviour check +0.2983 — rather than raw accuracy (+0.1431).

[[recuris-experiential-working-memory-evolution-yu-2026|Recuris]] attacks stage 2 by conditioning selection on explicit task state rather than the full growing history, worth **18.0 points** against an all-resident baseline that itself scores below carrying no skills.

## Related

- [[skill-routing]] — the parent problem, with the prevalence and precision numbers.
- [[agent-skills]] · [[skill-library]] · [[skill-evaluation]] · [[library-learning]]
- [[in-context-learning]] — the baseline a skill library must beat, and often does not.

## References

- [[demystifying-agent-skills-jiang-2026]] · [[evaluating-skills-not-just-agents-kevin-2026]] · [[recuris-experiential-working-memory-evolution-yu-2026]]
