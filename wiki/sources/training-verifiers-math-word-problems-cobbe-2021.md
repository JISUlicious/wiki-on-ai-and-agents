---
title: "Training Verifiers to Solve Math Word Problems"
type: source
created: 2026-10-06
updated: 2026-10-06
sources:
  - training-verifiers-math-word-problems-cobbe-2021.md
arxiv_id: "2110.14168"
venue: arXiv preprint
authors:
  - Karl Cobbe
  - Vineet Kosaraju
  - Mohammad Bavarian
  - Mark Chen
  - Heewoo Jun
  - Lukasz Kaiser
  - Matthias Plappert
  - Jerry Tworek
  - Jacob Hilton
  - Reiichiro Nakano
  - Christopher Hesse
  - John Schulman
first_author: Karl Cobbe
introduces:
  - "[[gsm8k]]"
  - "[[verifier]]"
year: 2021
tags:
  - 2021
  - paper
status: complete
importance: high
---

# Training Verifiers to Solve Math Word Problems

**Source**: `sources/training-verifiers-math-word-problems-cobbe-2021.md` ([arXiv:2110.14168](https://arxiv.org/abs/2110.14168))
**Authors**: Karl Cobbe, Vineet Kosaraju, Mohammad Bavarian, Mark Chen, Heewoo Jun, Lukasz Kaiser, Matthias Plappert, Jerry Tworek, Jacob Hilton, Reiichiro Nakano, Christopher Hesse, John Schulman ([[openai]]). Author order as listed on arXiv.
**Date**: October 2021 (v2 dated 2021-11-18)

## Summary

The paper introduces **[[gsm8k|GSM8K]]**: 8.5K linguistically diverse grade-school math word problems with natural-language solutions. It then proposes **training verifiers**:
- a generator samples many candidate solutions;
- a separately trained model judges whether each is correct;
- at test time, the top-ranked candidate is chosen.

This is the founding paper of the modern LLM **[[verifier]]** line.

## Key claims

- Even the largest GPT-3 models do poorly on GSM8K. The failures are "catastrophic mistakes" in multi-step reasoning, because "autoregressive models have no mechanism to correct their own errors."
- **Verification is worth about a 30× increase in model size.** A 6B model with a verifier slightly outperforms a fine-tuned 175B model. Verifiers also scale better with more data than fine-tuning does.
- Why verifiers help: they gain from *optionality* (choosing among samples), and verifying is generally an easier task than generating.
- **Token-level** verifiers, which predict correctness after every token, beat solution-level ones and are less prone to overfitting.
- Dropout is a strong regularizer for both fine-tuning and verification.
- **Calculator annotations.** Training solutions carry `<<…>>` calculation annotations. At test time a calculator overrides sampling inside them, an early form of [[tool-use|tool augmentation]].
- **The verifier's limit:** as the number of samples grows, performance first rises, then falls. With too many candidates, the search finds "adversarial" solutions that fool the verifier.

## Lineage

- **Built on:** [[reward-modeling|reward models]] from the RLHF line (Christiano 2017; Stiennon 2020), and the concurrent work of Shen et al. (2021).
- **Fed into:**
  - **process supervision**, in [[lets-verify-step-by-step-lightman-2023|Let's Verify Step by Step]] (2023), led by the same group;
  - **best-of-N** sampling as a [[test-time-compute]] strategy;
  - [[self-consistency]] (2022), which compares against verifier reranking;
  - the RL-with-verifiable-rewards line ([[deepseek-r1]]);
  - agentic verifiers such as SWE-Gym's.
- **GSM8K itself** became one of the most-used reasoning benchmarks of 2022–2024: [[chain-of-thought-prompting|CoT]] and self-consistency both report on it.

## Concepts & entities

[[verifier]] · [[gsm8k]] · [[reward-modeling]] · [[outcome-based-reward]] · [[process-reward-model]] · [[test-time-compute]] · [[tool-use]] · [[openai]]
