---
title: Skill Routing
type: concept
created: 2026-09-05
updated: 2026-09-05
sources:
  - what-keeps-agent-skills-from-being-reusable-zhang-2026.md
  - at-skills-attention-is-all-you-have-yin-2026.md
  - demystifying-agent-skills-jiang-2026.md
  - evaluating-skills-not-just-agents-kevin-2026.md
  - evomal-self-poisoning-self-evolving-coding-agents-wu-2026.md
  - recuris-experiential-working-memory-evolution-yu-2026.md
builds-on:
  - [[agent-skills]]
status: complete
importance: high
---

# Skill Routing

Getting the *right* [[agent-skills|skill]] in front of the model at the right moment. Distinct from whether the skill is any good: a perfectly written skill that never fires, or fires on the wrong task, contributes nothing.

By late 2026 this is the binding constraint on skill libraries, and four independent groups measure it four different ways.

## The four measurements

| Lens | Finding | Source |
|---|---|---|
| **Prevalence** | **67.0%** of 138,133 public skills carry at least one routing defect; missing trigger guidance is the single most common defect at 52.3%. A clean-vs-defective BM25 probe gives **88.5% vs 82.6% Hit@1** | [[what-keeps-agent-skills-from-being-reusable-zhang-2026]] |
| **Scarcity** | Skills compete for a small number of reliable trigger slots in the system prompt; the long tail never fires no matter how good it is | [[at-skills-attention-is-all-you-have-yin-2026]] |
| **Precision collapse** | Actual-use precision falls **29.6% → 3.3%** as the pool grows 5 → 100. Every skill added makes the rest harder to select | [[demystifying-agent-skills-jiang-2026]] |
| **A metric** | **Routing premium** = Lift(grouped) − Lift(isolated): run the paired test with the skill alone, then buried in a realistic library, and take the difference | [[evaluating-skills-not-just-agents-kevin-2026]] |

The routing premium is the useful contribution here, because it separates "this skill helps" from "this skill can be found" with a single paired-run design.

## Routing is a security property, not only a quality one

[[evomal-self-poisoning-self-evolving-coding-agents-wu-2026|EvoMal]] targets a skill's *description* rather than its body and lifts self-poisoning by **+26.9 percentage points, peaking at 86.7%**. The mechanism is that a malicious skill never has to be *invoked* — being **retrieved into context as an authoring template** is enough for the agent to copy its payload into a new skill.

That generalizes a condition [[mind-viruses-papadopoulos-2026|Mind Viruses]] found for agent-writable system-prompt files: what matters is not "is this in the system prompt" but "is this auto-retrieved into context." A similarity-indexed skill library reproduces the dangerous property without being a system prompt. So the retrieval index is part of the trust boundary.

## What actually helps

**Ground selection in explicit task state.** [[recuris-experiential-working-memory-evolution-yu-2026|Recuris]] runs the cleanest controlled comparison — the same ten skills with byte-identical bodies, all-resident versus state-selected — and state grounding wins by **18.0 points [+11.6, +24.6]**, while the all-resident variant scores *below carrying no skills at all* (65.6 vs 82.0) at 46% more cost per success. Making the routing rule an explicit, evolvable component also turns "missed, late, or irrelevant invocation" into a diagnosable fault class.

**Separate triggering from content and persistence.** [[at-skills-attention-is-all-you-have-yin-2026|@skills]]'s three tiers — Reference, Saved, Installed — exist because installation currently bundles all three, and only *triggering* needs to occupy prompt space. Decide per skill whether it must fire unasked; most need not.

**Keep the active set small.** The precision curve is the empirical form of the scarcity argument, and it does not care how good your skills are.

> [!warning] Two papers disagree on what bad routing costs
> [[demystifying-agent-skills-jiang-2026|Jiang et al.]] find precision collapsing from 29.6% to 3.3% while **downstream success stays essentially flat** (36.4% → 39.3%), concluding that exact ground-truth invocation is "neither sufficient nor necessary." [[recuris-experiential-working-memory-evolution-yu-2026|Recuris]] finds bad routing costs **18 points**.
>
> Both are probably right about their own setting. Recuris' benchmark gates success on specific database writes, where invoking the wrong skill really does fail the task; Jiang's tasks tolerate an approximate route. The practical read is that **routing precision matters exactly as much as your task's outcome depends on a specific action** — which is why the metric to watch is [[evaluating-skills-not-just-agents-kevin-2026|routing premium]] on *your* workload, not precision in the abstract.
>
> Note also that Recuris never measures routing precision directly, and its library is ten skills, never a hundred; the confusable-at-scale case is untested.

## Related

- [[agent-skills]] · [[skill-selection]] · [[skill-library]] · [[skill-md-format]] · [[skill-evaluation]]
- [[catastrophic-remembering]] — why an accumulating library makes routing worse over time.
- [[harness-if-instruction-following-across-instruction-surfaces-huang-2026]] — skill descriptions rank *last* in precedence when instruction surfaces conflict.
- [[prompt-injection]] · [[mind-viruses-papadopoulos-2026]] — the retrieval index as trust boundary.
- [[dense-retrieval]] · [[bm25]] — the machinery underneath.

## References

- [[what-keeps-agent-skills-from-being-reusable-zhang-2026]] · [[at-skills-attention-is-all-you-have-yin-2026]] · [[demystifying-agent-skills-jiang-2026]] · [[evaluating-skills-not-just-agents-kevin-2026]] · [[evomal-self-poisoning-self-evolving-coding-agents-wu-2026]] · [[recuris-experiential-working-memory-evolution-yu-2026]]

## Related 2026 sources

- [[building-an-ai-native-sdlc-plugin]] — the routing-scarcity argument applied: seven skills in the plugin, only two of them competing for selection
