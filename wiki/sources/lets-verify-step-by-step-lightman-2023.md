---
title: "Let's Verify Step by Step"
type: source
created: 2026-10-06
updated: 2026-10-06
sources:
  - lets-verify-step-by-step-lightman-2023.md
arxiv_id: "2305.20050"
venue: "arXiv preprint (later ICLR 2024)"
authors:
  - Hunter Lightman
  - Vineet Kosaraju
  - Yura Burda
  - Harri Edwards
  - Bowen Baker
  - Teddy Lee
  - Jan Leike
  - John Schulman
  - Ilya Sutskever
  - Karl Cobbe
first_author: Hunter Lightman
introduces:
  - "[[process-reward-model]]"
year: 2023
tags:
  - 2023
  - paper
status: complete
importance: high
---

# Let's Verify Step by Step

**Source**: `sources/lets-verify-step-by-step-lightman-2023.md` ([arXiv:2305.20050](https://arxiv.org/abs/2305.20050))
**Authors**: Hunter Lightman, Vineet Kosaraju, Yura Burda, Harri Edwards, Bowen Baker, Teddy Lee, Jan Leike, John Schulman, Ilya Sutskever, Karl Cobbe ([[openai]])
**Date**: 2023-05-31 (arXiv v1). The ICLR 2024 venue is from general knowledge, not the PDF.

## Summary

A large comparison of two ways to train the reward model that verifies chain-of-thought solutions:
- **Outcome supervision (ORM)**: feedback on the final answer only.
- **Process supervision ([[process-reward-model|PRM]])**: feedback on every intermediate step.

On the challenging MATH dataset, **process supervision wins clearly**.

## Key results

- **Best-of-1,860 on a representative 500-problem MATH subset:**

  | Selector | % solved |
  |---|---|
  | PRM | **78.2%** |
  | ORM | 72.4% |
  | Majority voting | 69.6% |

- The PRM's advantage **grows with N**, the number of candidate solutions searched over.
- **PRM800K** is released: 800,000 step-level human labels (positive, negative or neutral) on 75K solutions to 12K problems.
- **Active learning** (labelling the convincing wrong-answer solutions) makes process-label collection about 2.6× more data-efficient.
- **Alignment argument.** Process supervision rewards a chain of thought that humans endorse. It is easier to interpret, and it avoids rewarding incorrect reasoning that happens to land on the right answer. The authors present this as a *negative alignment tax*: the safer method also performs better.
- **Relation to prior work.** Uesato et al. (2022) had found ORM and PRM roughly equal on grade-school math. This paper uses a stronger base model, much more human feedback and the harder MATH dataset.

## Lineage

- **Built on:** [[training-verifiers-math-word-problems-cobbe-2021|Cobbe et al. 2021]] (outcome verifiers for best-of-N; Cobbe is a senior author here), Uesato et al. 2022, and [[chain-of-thought-prompting]].
- **Fed into:**
  - PRM-guided search and step-level reward in 2024–2025 reasoning models ([[large-reasoning-model]]);
  - the wiki's [[outcome-based-reward]] vs process-reward debate;
  - process-level evaluation of agents, e.g. PROBE in [[hallucination-detection]].
- **Counterpoint:** [[deepseek-r1]] (2025) deliberately avoided neural process reward models in RL, citing reward hacking and annotation cost. PRMs work well as best-of-N *selectors*, but whether they can serve as *RL rewards* at scale is still contested.

## Concepts & entities

[[process-reward-model]] · [[outcome-based-reward]] · [[verifier]] · [[reward-modeling]] · [[chain-of-thought-prompting]] · [[test-time-compute]] · [[openai]] · [[ilya-sutskever]]
