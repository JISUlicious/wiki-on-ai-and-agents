---
title: Backpropagation-Free Learning
type: concept
created: 2026-07-07
updated: 2026-07-07
sources:
  - diffusing-blame-yamada-2026.md
builds-on:
  - [[backpropagation]]
status: complete
importance: medium
---

# Backpropagation-Free Learning

A family of learning rules that train deep networks **without [[backpropagation]]**, motivated primarily by **biological plausibility** and secondarily by hardware.

## The weight transport problem

Backprop computes each layer's error by multiplying the downstream error by the **transpose of the forward weights** (`Wᵀ`). That requires each synapse to somehow *know* the strength of a different, forward synapse — the **weight transport problem**. Real neurons have no mechanism for this, which is the central reason backprop is considered biologically implausible. Backprop also requires a strictly sequential backward pass, which is awkward for parallel and analog hardware.

Backprop-free rules all answer the same question: **how does a hidden unit learn what it did wrong, without `Wᵀ`?**

## The main approaches

| Approach | How error reaches hidden units | Weight transport? |
|---|---|---|
| **Backpropagation** | exact gradient via `Wᵀ` | ✅ required |
| **Feedback Alignment (FA)** | a *fixed random* backward matrix; forward weights learn to align with it | ❌ |
| **[[direct-feedback-alignment\|Direct Feedback Alignment (DFA)]]** | output error broadcast to every layer through **random** matrices | ❌ |
| **Error Diffusion (ED)** | output-space error routed directly to hidden units by **fixed deterministic assignment** | ❌ (and no random matrices either) |

**Error Diffusion** (Kaneko 2000) is the strictest of these: it uses neither transposed forward weights *nor* random feedback matrices. [[diffusing-blame-yamada-2026|Yamada et al. 2026]] extend it past binary classification with **modulo error routing** — each hidden unit is assigned a fixed output channel and learns from that error component, giving coarse but deterministic output-associated credit assignment.

## What the evidence says

From [[diffusing-blame-yamada-2026]] (which also enforces [[dales-principle|Dale's principle]], a separate constraint):

- These rules **work, but pay a tax that grows with task difficulty**. ED: 96.7% MNIST → 61.7% CIFAR-10; DFA: 97.6% → 69.1%. Both remain far from gradient-trained competitiveness on CIFAR-10.
- **Which rule wins is domain-dependent.** DFA is the stronger method on supervised classification but the *weakest* on Craftax, an open-ended RL exploration task — random feedback pathways that suffice for classification **fail on complex RL**. Meanwhile ED-PPO beats BP-PPO on HalfCheetah.
- Coarse routing introduces **higher variance** than backprop, suggesting weaker reliability for fine-grained temporal credit assignment.

> [!note] Methodological lesson
> The same paper's ablation reversal between MNIST and CIFAR-10 — where the single most critical mechanism on one task is negligible on the other — argues that **single-benchmark evaluation of biologically plausible methods is misleading**. Cross-domain evaluation is what exposes credit-assignment bottlenecks.

## Related

- [[backpropagation]] — the baseline being replaced, and the source of the weight-transport objection.
- [[dales-principle]] — an orthogonal biological constraint often imposed alongside.
- [[direct-feedback-alignment]] — the strongest random-feedback baseline.

## References

- [[diffusing-blame-yamada-2026]] — Error Diffusion across classification and RL, under Dale's principle.
