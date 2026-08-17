---
title: ALFWorld
type: concept
created: 2026-08-17
updated: 2026-08-17
sources:
  - memharness-memory-is-reconstructed-not-replayed-wu-2026.md
  - agentopsd-recursive-self-distillation-agentic-rl-wang-2026.md
  - harness-r1-learning-to-edit-runtime-harnesses-shao-2026.md
  - reason-wide-not-deep-singh-2026.md
status: draft
importance: high
tags:
  - benchmark
---

**ALFWorld** is a text-based household-task environment that has become the default proving ground for [[embodied-agent|embodied]] and long-horizon [[llm-agent|LLM agents]]. An agent receives a natural-language goal ("put a clean mug in the coffee machine") and interacts with a simulated home through text commands, succeeding only if the final world state matches the goal.

Its structural properties are what make it ubiquitous:

- **Multi-step and sparsely rewarded.** Success is judged at the end, which makes it a natural testbed for credit-assignment methods ([[agentopsd-recursive-self-distillation-agentic-rl-wang-2026|AgentOPSD]]) and [[outcome-based-reward|outcome-based rewards]].
- **Task families with shared structure** (Pick, Clean, Heat, Cool, Examine, Pick-Two), so cross-task transfer and [[skill-library|skill reuse]] can be measured rather than assumed.
- **Cheap to run**, which is why it appears as the first column of nearly every agent table in this wiki.

## Why it recurs in this wiki

ALFWorld is referenced by more than twenty pages here. It is the shared axis along which otherwise unrelated claims can be compared:

| Result | Source |
|---|---|
| 85.2% success from reconstructing rather than replaying memory; `w/o memory` scores **100.0** on Pick alone | [[memharness-memory-is-reconstructed-not-replayed-wu-2026]] |
| 89.1% on Qwen2.5-7B; degradation only **−0.54** points per extra turn vs GRPO's −2.91 | [[agentopsd-recursive-self-distillation-agentic-rl-wang-2026]] |
| One of three environments where a learned harness editor lifted a frozen agent 44.3 → 53.6 | [[harness-r1-learning-to-edit-runtime-harnesses-shao-2026]] |
| A distilled 40–130 line skill beats reasoning mode outright (0.787 vs 0.713) | [[reason-wide-not-deep-singh-2026]] |

> [!warning] Saturation and the memory trap
> Two independent results here should temper ALFWorld numbers. [[memharness-memory-is-reconstructed-not-replayed-wu-2026|MemHarness]] found that *disabling test-time memory entirely* still scored 83.0 against 76.4 for RL-only — so most of the reported "memory" gain was a training-time effect — and that verbatim memory replay ([[mem0]]-style) landed **22–24 points below plain GRPO**. Per-task-family scores also diverge sharply from the average; Pick is close to saturated while composite tasks are not.

## Related

- [[webshop]] — the web-navigation counterpart, almost always reported alongside it.
- [[embodied-agent]] · [[llm-agent]] · [[agent-benchmark]] · [[agentic-rl]]
- [[reflexion]] · [[react]] — both used ALFWorld as a headline evaluation.
- [[memory-management]] · [[experiential-learning]] — the capability ALFWorld is most often used to probe.

## References

- [[memharness-memory-is-reconstructed-not-replayed-wu-2026]] · [[agentopsd-recursive-self-distillation-agentic-rl-wang-2026]] · [[harness-r1-learning-to-edit-runtime-harnesses-shao-2026]] · [[reason-wide-not-deep-singh-2026]]
