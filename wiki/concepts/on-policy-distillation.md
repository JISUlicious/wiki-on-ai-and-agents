---
title: On-Policy Distillation
type: concept
created: 2026-08-17
updated: 2026-09-15
sources:
  - weak-to-strong-on-policy-distillation-yu-2026.md
  - on-policy-self-distillation-without-supervision-li-2026.md
  - agentopsd-recursive-self-distillation-agentic-rl-wang-2026.md
status: complete
importance: high
---

# On-Policy Distillation

[[knowledge-distillation|Distillation]] in which the training sequences are sampled from the **student**, not the teacher. The teacher is asked to score or correct the student's own rollouts, and the student is trained to match that signal token by token.

The distinction matters because off-policy distillation trains on a distribution the student will never visit at inference time. Sequences the student actually produces are where its errors live; scoring those is what makes the gradient informative. This is the same exposure-bias argument that motivates on-policy RL, applied to a supervised objective — which is why on-policy distillation sits between [[fine-tuning|SFT]] and [[grpo|GRPO]] in both cost and behaviour.

## Three variants in this wiki

**Weak-to-strong** — the teacher is *worse* than the student. [[weak-to-strong-on-policy-distillation-yu-2026|Yu et al.]] build a proxy teacher by logit arithmetic, `softmax(z_base + α(z⁺ − z⁻))`, contrasting a weak positive model against a weak negative one and adding the difference to the strong student's own base logits. A Qwen3-8B student reaches 51.8 on math against 46.5 for plain on-policy distillation, **surpassing the 4B expert supplying the signal (48.8)**. Out of domain the gap is larger: [[gpqa|GPQA]]-Diamond 38.9 → 56.5, while plain OPD *degrades* IFBench below the base model.

**Self-distillation without supervision** — no external teacher at all. [[on-policy-self-distillation-without-supervision-li-2026|U-OPSD]] derives the correction signal from internal consistency across the model's own rollouts. Non-thinking mode goes 43.57 → 54.31 at 8B, **beating gold-label-supervised OPSD by 2.3 points** even though its pseudo-labels are 13.3% wrong — the signal apparently does not need to be correct so much as *on-distribution*.

**Turn-level credit assignment** — [[agentopsd-recursive-self-distillation-agentic-rl-wang-2026|AgentOPSD]] applies the same idea inside agentic RL, using the self-distillation gap as a bounded, strictly positive multiplier on GRPO's advantage (so it can never reverse the update direction). Its distinguishing result is horizon robustness: **−0.54 success points per extra turn against GRPO's −2.91**.

## The pattern worth carrying

All three get their gains from *where* the signal is measured rather than from a better teacher. Weak-to-strong beats its own teacher; self-distillation beats gold labels; turn-level beats trajectory-level. The recurring claim is that on-policy placement is worth more than supervision quality — which, if it holds, changes what a "teacher" is for.

> [!warning] Reported gains are thinner than the abstracts suggest
> Each paper overstates in its own abstract. AgentOPSD's "consistently improves" is six of eight aggregate comparisons (it ties SDAR on 3B ALFWorld and *loses* 7B WebShop accuracy). U-OPSD's thinking-mode gain at 8B is **+0.02 — a tie** — and its own ablations show the reported defaults are suboptimal, kept only to cost-match the baseline. Yu et al.'s "multiple weaker teachers" is only literally true in the small-gain settings; the headline setting uses a 4B *domain expert* that beats the student 48.8 vs 17.0 in-domain. None of the three reports seed variance.

## Related

- [[knowledge-distillation]] — the parent concept; off-policy is the default there.
- [[grpo]] · [[agentic-rl]] · [[reinforcement-learning]] — the neighbouring paradigm, and AgentOPSD's host.
- [[outcome-based-reward]] · [[verifier]] — what supplies reward when there is no teacher.
- [[self-consistency]] — U-OPSD's internal-consistency signal is the training-time analogue.
- [[embarrassingly-simple-self-distillation-zhang-2026]] — earlier self-distillation result in this wiki.
- [[steering-vectors]] · [[activation-addition]] — adjacent to (but not the same as) Yu et al.'s logit arithmetic.

## References

- [[weak-to-strong-on-policy-distillation-yu-2026]] · [[on-policy-self-distillation-without-supervision-li-2026]] · [[agentopsd-recursive-self-distillation-agentic-rl-wang-2026]]

## Related 2026 sources

- [[rethinking-on-policy-distillation-ii-one-training-example-fu-2026]] — one training problem recovers 87% of full-data OPD's gain at 300 steps; the paper's diagnosis is that OPD is limited by how fast the student absorbs signal, not by data
- [[co-evolving-harnesses-and-models-on-policy-correction-yu-2026]] — on-policy correction beats imitation: fine-tuning a weaker model on a stronger model's full trajectories *dropped* it on all seven tasks (−4 to −30 points) under the harness evolved for it; rewriting only the failing turn helps
