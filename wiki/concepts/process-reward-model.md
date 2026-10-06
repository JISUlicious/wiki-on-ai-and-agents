---
title: Process Reward Model (PRM)
type: concept
created: 2026-10-06
updated: 2026-10-06
sources:
  - lets-verify-step-by-step-lightman-2023.md
  - training-verifiers-math-word-problems-cobbe-2021.md
status: draft
importance: high
---

# Process Reward Model (PRM)

A reward model or [[verifier]] that scores **each intermediate step** of a chain-of-thought, rather than only the final answer. Its counterpart, the outcome reward model (ORM), scores only the result; see [[outcome-based-reward]].

## Lineage

| When | Work | Contribution |
|---|---|---|
| Oct 2021 | [[training-verifiers-math-word-problems-cobbe-2021\|Cobbe et al.]] | **Outcome** verifiers for best-of-N on [[gsm8k\|GSM8K]]; a *token-level* verifier, a step toward finer granularity |
| 2022 | Uesato et al. (DeepMind) | Names outcome vs process supervision; finds them roughly equal on grade-school math |
| **May 2023** | **[[lets-verify-step-by-step-lightman-2023\|Let's Verify Step by Step]]** | Process supervision clearly beats outcome supervision on MATH: best-of-1,860 solves **78.2%** (PRM) vs 72.4% (ORM) vs 69.6% (majority vote). Releases **PRM800K** |
| 2024–2025 | Reasoning models | PRMs used for search and reranking; disputed as RL rewards |
| 2025 | [[deepseek-r1\|DeepSeek-R1]] | Deliberately avoids neural PRMs in RL, citing reward hacking and annotation cost; uses rule-based outcome rewards |

## Arguments

**For** (from Lightman et al.):
- more precise feedback, because it locates the first error;
- more interpretable;
- it rewards reasoning humans endorse, not lucky answers. The authors call this a "negative alignment tax".

**Against or limits:**
- step labels are expensive to collect;
- under RL optimization the model can learn to exploit the PRM;
- it fits domains with clear steps (math) better than open-ended agent trajectories.

## In agents

The process-vs-outcome question comes back for agents as *trajectory* evaluation: grading each step of an agent run versus only the end state. See [[verifier]], [[agent-as-a-judge]] and [[hallucination-detection]].

## Related

[[verifier]] · [[outcome-based-reward]] · [[reward-modeling]] · [[test-time-compute]] · [[chain-of-thought-prompting]] · [[large-reasoning-model]]

## References

- [[lets-verify-step-by-step-lightman-2023]]
- [[training-verifiers-math-word-problems-cobbe-2021]]
