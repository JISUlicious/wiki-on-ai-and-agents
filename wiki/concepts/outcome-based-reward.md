---
title: Outcome-Based Reward
type: concept
created: 2026-07-28
updated: 2026-07-28
sources:
  - swe-gym-pan-2024.md
  - memory-r1-yan-2025.md
status: draft
importance: medium
---

# Outcome-Based Reward

An **outcome-based reward** (or outcome-supervised reward model, ORM) scores a trajectory by **whether it ultimately succeeded**, not by the quality of its intermediate steps. The training signal is a single terminal label — did the tests pass, was the task solved — propagated back over the whole rollout.

It is the simpler half of a standing dichotomy in [[verifier|verification]] and reward modeling:

| | **Outcome supervision** | **Process supervision** |
|---|---|---|
| Label | one terminal success/failure | per-step correctness |
| Annotation cost | cheap — often free from execution | expensive, usually human or LLM-judged |
| Credit assignment | coarse; a lucky path scores like a sound one | fine-grained; localizes the error |
| Needs | an executable or checkable outcome | a step-level notion of "correct" |

## Where it works

Outcome rewards shine wherever the world supplies the label for free — exactly the "verify by running" case in [[verifier]]. [[swe-gym-pan-2024|SWE-Gym]] is the canonical instance: an outcome-supervised verifier trained on trajectory pass/fail labels from unit tests, then used for **test-time best-of-N selection** — sample many rollouts, keep the verifier's pick. That pushed SWE-Bench Verified to **32.0%** and Lite to 26.0%, a new open-weight state of the art at release, with log-linear gains in N.

The appeal is that the reward is **grounded rather than judged**: no LLM opinion sits between the model and the signal, so there is far less to game than with a rubric judge.

## The limitation

Coarseness. A trajectory that stumbles into the right answer scores identically to one that reasons soundly, so the model can learn to value luck. This is the motivation for **process reward models** (PRMs) and for [[probe-process-benchmark-hallucination-detection-2026|PROBE]]-style step-level evaluation — and it is the same argument [[agent-as-a-judge]] makes about intermediate feedback, noting that when full-task solve rates sit at 0–1.8%, a binary terminal signal carries almost no information.

The general caution in [[verifier]] applies with force here: prefer outcome rewards as an **inference-time selector** over an RL objective, since optimization pressure finds whatever the terminal check fails to capture.

> [!note] Draft
> Created during a lint pass — `[[outcome-based-reward]]` was referenced from [[verifier]] and two source pages with no page behind it. Deserves expansion with the ORM-vs-PRM literature (Lightman et al. "Let's Verify Step by Step", math-reasoning PRMs) once those are ingested.

## Related

- [[verifier]] — learned verifiers and best-of-N sit inside its mechanism taxonomy.
- [[swe-gym-pan-2024]] · [[memory-r1-yan-2025]] — outcome-supervised training in practice.
- [[agentic-rl]] · [[policy-optimization-methods]] · [[rlhf]] — where the reward signal is consumed.
- [[test-time-compute]] — best-of-N is an inference-time-scaling technique.

## References

- [[swe-gym-pan-2024]] — outcome-supervised verifier; test-time best-of-N.
- [[memory-r1-yan-2025]] — outcome-based reward applied to memory operations.
