---
title: "Looped Transformers for Length Generalization"
type: source
created: 2026-05-20
updated: 2026-05-20
sources:
  - looped-transformers-length-generalization-fan-2024.md
arxiv_id: "2409.15647"
venue: ICLR 2025
authors:
  - Ying Fan
  - Yilun Du
  - Kannan Ramchandran
  - "[[kangwook-lee]]"
first_author: Ying Fan
year: 2024
introduces:
  - "[[looped-transformer]]"
  - "[[length-generalization]]"
tags:
  - 2024
status: complete
importance: high
---

# Looped Transformers for Length Generalization

**Source**: `sources/looped-transformers-length-generalization-fan-2024.md`
**arXiv**: [2409.15647](https://arxiv.org/abs/2409.15647)
**Venue**: ICLR 2025
**Authors**: Ying Fan (University of Wisconsin–Madison), Yilun Du (MIT), Kannan Ramchandran ([[uc-berkeley|UC Berkeley]]), [[kangwook-lee]] (Wisconsin)
**Code**: https://github.com/UW-Madison-Lee-Lab/looped-tf

## Summary

This paper is the canonical work linking **looped Transformers** to **length generalization** on algorithmic tasks. Standard decoder-only Transformers have a *fixed* depth, which means they cannot allocate extra computation to longer or harder inputs — a known root cause of their failure to generalize on tasks like binary addition, parity, and copy beyond the training length distribution. The authors propose a recurrent decoder-only architecture that reuses the same Transformer block for a **variable, input-adaptive number of steps**, with input injection at every iteration and **NoPE** (no positional encoding). Crucially, training supervises only the final-step output (full-output prediction, not next-token), but the number of steps `T(n)` is provided per-example, so the same block is forced to receive supervision under many different step counts and lengths. At inference, the model either uses an oracle step count or a **maximum-confidence halting rule** that picks the step minimizing cross-entropy on the decoded output.

To formalize the scope, the paper defines the **n-RASP-L** problem class: tasks whose solution can be expressed as `P_pre ∘ (P')^T(n) ∘ P_post` where each component is a [[rasp-l|RASP-L]] program (i.e., each loop step is realizable by a fixed-depth Transformer). They give explicit n-RASP-L constructions for Copy, Parity, and Addition — none of which has a known single-pass RASP-L solution under standard next-token prediction, which explains why vanilla Transformers fail to length-generalize on them. Empirically, looped Transformers trained up to length 20 generalize near-perfectly to length 40+ on Parity, and substantially outperform NTP, NTP-with-pause-tokens, and NTP-with-weight-tied-layers baselines on Copy, Addition, Binary Sum, Binary Multiplication, and Unique Set. This paper is the headline citation when people argue that **looping = adaptive depth = length generalization** for algorithmic reasoning, and it positions itself as a decoder-only, FOP-based refinement of the [[universal-transformer|Universal Transformer]] / PonderNet lineage of [[adaptive-computation-time]].

## Key Points

- **n-RASP-L class** (Def. 3.1): algorithmic tasks decomposable as `P_pre ∘ (P')^T(n) ∘ P_post` with each component a RASP-L program. Strictly more expressive than RASP-L under NTP because the loop count `T(n)` is allowed to depend on input length.
- **Variable, input-adaptive step count.** A single Transformer block is applied `T(n)` times, where `T(n)` scales with the problem instance (e.g., `T = n` for parity/copy, `T = n+1` for n-digit addition, `T = n·m` for multiplication).
- **Full-output prediction (FOP)** instead of NTP: query is the only input, remaining positions are EOS-padded, and the model emits all answer tokens after the looped processing finishes. Distinguishes this work from prior looped/recurrent Transformer setups.
- **Input injection**: the original input embedding is added back to the output of every looped step (à la deep equilibrium models, Bai et al. 2019). Ablations show this materially helps Addition, Binary Sum, and Unique Set.
- **NoPE** (no positional encoding) is used so RASP-L assumptions hold; the paper notes positional embedding schemes are orthogonal and could compose.
- **Training algorithm**: end-to-end supervision on `(x, y, T(n))` triples — no intermediate-step / scratchpad / CoT labels needed. Because the dataset spans many `T(n)` values, the same shared block receives gradient signal at many depths, encouraging step-invariant computation.
- **Adaptive halting at inference**: (1) Oracle step count from the data-generation rule, or (2) **Maximum-confidence rule** — pick the step `t` that minimizes cross-entropy of the decoded output, computed either per-batch (`B = N_test`) or per-sample (`B = 1`).
- **Length-generalization results** (Fig. 4): trained on lengths `[1, 20]`, the looped model approaches 100% accuracy on lengths far beyond training, while NTP, NTP-Pause, and NTP-Loop (weight-tied fixed-depth) baselines collapse. Parity generalizes from length 20 → length 40+; Addition/Copy/Sum hold near-perfectly at training length + 10 where NTP fails.
- **Convergence behavior**: on Addition, Copy, Multiplication, and Unique Set, the loop's output converges and stays correct for many additional steps after the answer is reached — even though the model was never explicitly trained to converge. This is what makes the maximum-confidence halting rule practical.
- **Comparison table (Table 1) vs. [[universal-transformer|UT]] and PonderNet**: this work is decoder-only, FOP-based, no PE, uses input injection, and uses confidence-based or predefined halting — versus UT's encoder–decoder, NTP, with-PE, ACT halting design.
- **Stated limitations**: single-loop only (no nested or sequential loops yet); training cost grows with `T(n)`; requires the ground-truth step count `T(n)` in training data — a weaker assumption than full CoT supervision but stronger than vanilla end-to-end training.

## Entities Mentioned

- [[kangwook-lee]] — senior author, runs the UW-Madison Lee Lab that released the code
- [[uc-berkeley|UC Berkeley]] — Ramchandran's affiliation (the paper's third site, alongside UW–Madison and MIT)

## Concepts Discussed

- [[looped-transformer]] — the architecture this paper popularizes for length generalization
- [[length-generalization]] — the core capability being studied
- [[rasp-l]] — the learnable RASP subset that defines per-step expressivity; n-RASP-L is its loop-augmented extension
- [[adaptive-computation-time]] — the broader paradigm of input-conditional compute allocation; ACT is the halting mechanism used by Universal Transformers
- [[universal-transformer]] — direct architectural predecessor; this paper's Table 1 is essentially "what we changed vs. UT"
- [[chain-of-thought-prompting]] — discussed as the leading alternative (uses tokens-as-scratchpad to get adaptive sequential compute); the paper argues looping is more general because it does not require collecting CoT data

## Notable Quotes

> "We demonstrate that looped Transformers with an adaptive number of steps significantly improve length generalization. We focus on tasks with a known iterative solution, involving multiple iterations of a RASP-L operation—a length-generalizable operation that can be expressed by a finite-sized Transformer." — Abstract

> "Looped Transformer can naturally break the limitation of the fixed depth in the standard Transformer architecture: One can adjust the number of looped steps based on the computational complexity of the underlying algorithmic solution." — §1

> "Although we do not have intermediate supervision for this input, there could exist some other input where we match the output after T − 1 steps... As a result, we expect the model to learn length-generalizable intermediate steps that could potentially generalize to more steps at test time." — §4.2.2

> "For Parity, it can generalize to more than 40 digits near perfectly when trained with up to 20 digits. Moreover, for tasks like addition and copy, where the next token prediction failed when tested on maximum training length +10, our looped model can still perform almost perfectly." — §6.2

## References

_Original source: `sources/looped-transformers-length-generalization-fan-2024.md`_
