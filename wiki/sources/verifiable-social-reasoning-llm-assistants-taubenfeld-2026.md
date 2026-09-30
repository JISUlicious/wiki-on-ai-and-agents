---
title: "Verifiable Social Reasoning for LLM Assistants"
type: source
created: 2026-09-30
updated: 2026-09-30
sources:
  - verifiable-social-reasoning-llm-assistants-taubenfeld-2026.md
arxiv_id: "2609.17496"
authors:
  - Amir Taubenfeld
  - Zorik Gekhman
  - Avigail Grinstein-Dabush
  - Itay Laish
  - Ariel Goldstein
  - Marian Croak
  - Avinatan Hassidim
  - Yossi Matias
  - Amir Feder
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# Verifiable Social Reasoning for LLM Assistants

**Amir Taubenfeld\*, Zorik Gekhman\* et al.** ([[google-research|Google Research]], Hebrew University, University of Cambridge; \*equal contribution, order random) — [arXiv:2609.17496](https://arxiv.org/abs/2609.17496), cs.AI, 15 Sep 2026. Preprint; dataset and code released.

## Summary

People ask assistants about their social lives, for example "is my coworker supportive or undermining me?" Two things make this hard to evaluate. First, the assistant only hears the **user's subjective, partial retelling**. Second, the other person's real motive has no ground truth. **Fuse** (Framework for User-mediated Social Evaluation) solves the second problem by construction. A [[generative-agents|generative multi-agent simulation]] built on Concordia plays out several scenes in which a *target* persona acts on an assigned hidden motive. The simulated user then describes those events to the assistant under test, which must name the motive. Because simulation and debrief are separate, the dataset is static and reusable. It contains 30 scenario templates across five mental-state categories (desire, intention, belief, emotion, knowledge), 1,200 simulations and **21,600 first user messages**. Gemini 3.1 Flash-Lite drives the personas and acts as the judge.

## Key points

- **Human validation (24k annotations).** Raters shown the raw events recover the intended motive in 97% of 300 simulations. A human majority vote on the first user message alone gets 88% right, for an MSR of 89.8. MSR scores abstention at d = 0.75.
- **No model reaches human level.** Across 12 models the best MSR is 83.7 (Gemini 3.7 Flash). Many models are wrong more than 20% of the time. Four models from the Gemma and Claude families abstain in over a third of cases despite a forced-choice prompt, so they are correct at most 50% of the time. [[claude|Claude]] Opus 5 is near-perfect as an *observer* of the raw events but abstains heavily as an *assistant*. That is a policy choice, not missing capability.
- **User mediation costs every model.** The observer-vs-assistant gap is present for all of them, and even a 120B Mistral model is wrong 20.9% of the time on the raw events.
- **Biased framing.** A single sentence nudging the user toward the wrong motive lowers MSR by an average of 7.7 ± 3.1 points (6.9–12.5 for the 8 non-abstaining models), against 3.6 for humans. Case studies show models amplifying the user's suspicion, for example calling a new hire's deference "a classic ingratiation tactic."
- **Detail.** More narrative detail raises average model MSR from 74.2 to 80.1 (humans: 88.3 to 91.1). The gap narrows by 22%, which suggests models commit to an ungrounded default reading, sometimes an alarming one, before the evidence warrants it.
- **More turns do not reliably help.** Scores rise until turn 4 and then plateau or fall, because clarifying questions also give the user room to push their framing. No model reaches the human turn-2 baseline.

## What it does not show

- The scenarios are synthetic and deliberately clear-cut, with two motives each. Real situations are messier.
- One model family generates, simulates and judges. The analysis is mostly first-message.
- There is a minor internal inconsistency: the text says "seven closed-weight models" but lists six, so the total is 12, as the abstract says.

## Related

- [[sycophancy]] — framing bias here is sycophancy about *third parties*, not facts.
- [[xyeval-agents-say-yes-to-bad-advice-wu-2026]] — the agentic counterpart: a user's wrong framing pulls the model off the real problem.
- [[generative-agents]] · [[multi-agent]] — simulation as a way to build ground truth.
- [[agent-evaluation]] · [[llm-as-a-judge]] — a three-way correct / incorrect / abstain judge, with abstention-aware scoring.

## References

- [arXiv:2609.17496](https://arxiv.org/abs/2609.17496)
