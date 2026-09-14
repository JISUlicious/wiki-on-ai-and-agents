---
title: Knowledge Distillation
type: concept
created: 2026-06-19
updated: 2026-09-15
sources:
  - deepseek-r1-2025.md
status: draft
importance: medium
tags:
  - 2026
---

# Knowledge Distillation

Knowledge distillation transfers capability from a large teacher model into a smaller, cheaper student, typically by training the student to match the teacher's outputs or reasoning traces. It lets the strengths of an expensive model be packaged into a more deployable one. Reasoning-focused work often distills traces produced under [[reinforcement-learning]] into compact students.

## References

- [[deepseek-r1-2025]]

## Related 2026 sources

Surfaced via newsletter ingests; see [[index]].

- [[on-the-geometry-of-on-policy-distillation-shen-2026]] — geometry of on-policy distillation: a distinct weight update
- [[multi-turn-on-policy-distillation-prefix-replay-liao-2026]] — ReOPD: replay pre-collected prefixes so the student never touches the environment — 4.2–9.1× faster rollouts
- [[on-policy-delta-distillation-heo-2026]] — OPD²: distil the *delta* (teacher minus the teacher's own base) rather than teacher minus student; +19.8 pp at 1.7B
- [[on-policy-distillation]] — train on the *student's* rollouts — the variant family behind three 2026 results
- [[weak-to-strong-on-policy-distillation-yu-2026]] — a student surpassing the weaker models supplying its signal (51.8 vs the 4B expert's 48.8)
- [[on-policy-self-distillation-without-supervision-li-2026]] — beats gold-label supervision with pseudo-labels that are 13.3% wrong
- [[illicit-distillation]] — the same technique done covertly and without authorization, through fraudulent accounts and proxy networks. Anthropic's 2026 report attributes seven campaigns to China-based labs, the largest at over 151 million exchanges, and claims a distilled student inherits capability but not the teacher's refusals
- [[co-evolving-harnesses-and-models-on-policy-correction-yu-2026]] — a measured failure of trajectory distillation: the student copies a planning strategy it cannot execute
