---
title: Continual Learning
type: concept
created: 2026-08-17
updated: 2026-09-15
sources:
  - continualskillbench-guan-2026.md
  - why-does-claude-md-keep-growing-chakrabarti-2026.md
status: draft
importance: medium
---

# Continual Learning

Learning a sequence of tasks over time without losing what was learned earlier. The organizing problem of the field is **catastrophic forgetting**: gradient updates for a new task overwrite the weights encoding an old one, so performance on earlier tasks collapses.

Classical mitigations fall into three families — *regularization* (penalize movement in weights that mattered, e.g. EWC), *replay* (interleave stored or generated examples from earlier tasks), and *architecture* (allocate new capacity per task, freezing the old).

## Why this matters for LLM agents

The agentic literature has largely stepped around gradient-based continual learning by making adaptation **non-parametric**: append a skill, write a memory, extend an instruction file. Nothing is overwritten, so — the argument goes — nothing is forgotten. [[self-improving-agent]] and [[skill-library]] both rest on this move.

The 2026 evidence is that the escape is not free. Appending trades catastrophic forgetting for [[catastrophic-remembering|catastrophic *remembering*]]: the store grows without bound and degrades in usefulness, because deletion requires provenance the system never recorded. The two failures are duals, not an escape and a problem.

[[continualskillbench-guan-2026|ContinualSkillBench]] measures the in-context version directly — 100 interconnected subtasks per domain, ordered by difficulty, with deliberate reuse opportunities — and finds that **maintaining an explicit skill library performs comparably to just keeping the transcript** (0.602 vs 0.605). Most of the sequential gain comes from adapting to recent context and feedback, not from reusable abstraction. Weaker models accumulate *larger and more fragmented* libraries: GPT-4o reached 384 skills at quality 5.68 against GPT-5.3-Codex's 205 at 7.94, which is what failed abstraction looks like from outside.

## The open question

Neither the parametric nor the non-parametric route currently consolidates. Gradient methods overwrite; appending methods hoard. The mechanisms that work in both new-evidence settings — [[memory-consolidation|consolidation]], retrieval conditioned on the current state, and explicit rationale — are precisely the ones that decide *what to drop*, which is the operation both families lack.

## Related

- [[catastrophic-remembering]] — the dual failure mode and the 2026 evidence.
- [[memory-consolidation]] · [[offline-consolidation]] · [[memory-evolution]]
- [[in-context-learning]] — the non-parametric baseline that skill libraries fail to beat.
- [[fine-tuning]] · [[lora]] — the parametric routes.
- [[experiential-learning]] · [[skill-acquisition]] · [[self-improving-agent]]

## References

- [[continualskillbench-guan-2026]] · [[why-does-claude-md-keep-growing-chakrabarti-2026]]

## Related 2026 sources

- [[continual-learning-mechanisms-compose-long-horizon-memorization-zhang-2026]] — mechanisms compose: 28× better retention over 100 sequential tasks, with replay and merged LoRA more than additive — at the cost of general capability
