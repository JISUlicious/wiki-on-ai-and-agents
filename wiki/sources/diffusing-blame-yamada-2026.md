---
title: "Diffusing Blame: Task-Dependent Credit Assignment in Biologically Plausible Dual-Stream Networks"
type: source
created: 2026-07-07
updated: 2026-07-07
sources:
  - diffusing-blame-yamada-2026.txt
arxiv_id: "2606.31700"
venue: ALIFE 2026
year: 2026
authors:
  - Yutaro Yamada
  - Luca Grillotti
  - Rujikorn Charakorn
  - Sebastian Risi
  - David Ha
  - Robert Tjarko Lange
introduces:
  - [[dales-principle]]
  - [[backprop-free-learning]]
tags:
  - 2026
  - paper
status: complete
importance: medium
---

# Diffusing Blame: Task-Dependent Credit Assignment in Biologically Plausible Dual-Stream Networks

**[[sakana-ai|Sakana AI]]**, [arXiv:2606.31700](https://arxiv.org/abs/2606.31700) (v1: 2026-06-30), accepted at **ALIFE 2026**.
**Authors**: Yutaro Yamada, Luca Grillotti, Rujikorn Charakorn, Sebastian Risi, David Ha, Robert Tjarko Lange.

## Summary

A [[backprop-free-learning|backpropagation-free]] learning study: can a network learn competitively while strictly obeying **[[dales-principle|Dale's principle]]** (each neuron's synapses are uniformly excitatory *or* inhibitory)? The authors revive **Error Diffusion (ED)** (Kaneko 2000) — which routes a global output-space error signal directly to all hidden units, without transporting transposed forward weights and without random feedback matrices — and extend it beyond binary classification via **modulo error routing**.

Headline results: **96.7% MNIST** and a **61.7% CIFAR-10 baseline** (the first application of ED to convolutional networks; prior work reached only ~MLP-level), plus a reinforcement-learning extension **ED-PPO** that is competitive with [[direct-feedback-alignment|DFA]] on continuous control.

The paper's more interesting contribution is methodological, not numerical: **which mechanisms matter reverses between tasks**, exposing credit-assignment bottlenecks invisible to single-benchmark evaluation.

## Method

**Dual-stream architecture** — each layer is split into an excitatory (`p`) and inhibitory (`n`) stream. All *learnable* parameters stay non-negative; the negation signs on cross-stream connections are **structural (hardcoded)**, so inhibition comes from fixed pathway identity rather than from signed weights. Cost: four weight sub-matrices per layer — **32M params vs 8M** for a comparable single-stream DFA network.

**Error Diffusion update** — replaces backpropagated layerwise errors with an output-space error routed directly to hidden units. The extension to multi-output: each hidden unit is assigned a **fixed output channel** (deterministic modulo routing), giving coarse output-associated credit assignment with no weight transport and no random feedback.

**Three classification-specific innovations** (not used in the RL setting):
1. **layer-specific sigmoid widths** — the sigmoid derivative directly gates the ED error signal, so attenuation is severe; wider sigmoids keep derivatives alive
2. **batch-centered class error (MCE)** — subtract the mini-batch mean per class, countering the 9:1 non-target/target imbalance in 10-way classification
3. **asymmetric initialization** (3:1 E/I)

## Results

| Method | MNIST | CIFAR-10 | Dale-compliant? |
|---|---|---|---|
| **Proposed ED** | **96.7%** | **61.7%** | ✅ |
| [[direct-feedback-alignment\|DFA]] | 97.6% | 69.1% | ❌ (requires negative weights) |

The ED↔DFA gap **widens with task difficulty** (0.9pp → 7.4pp) — a quantification of the *cost of enforcing Dale's principle*. CIFAR-10 remains far from gradient-trained competitiveness; the authors say so plainly.

### The ablation reversal (the conceptual headline)

The importance hierarchy of the three innovations **inverts between tasks**:

- **MNIST** — removing layer-specific sigmoid widths is *catastrophic* (collapses to near-chance). Batch-centering costs little; asymmetric init has no measurable effect. **Gradient flow is the sole bottleneck.**
- **CIFAR-10** — **reversed**: removing batch-centered error is now most destructive (four of five seeds collapse). Higher inter-class similarity makes the 9:1 error imbalance overwhelming, so centering becomes essential to prevent uniform output suppression.

> Evaluating biologically plausible methods on a single benchmark may obscure critical design trade-offs.

### Reinforcement learning (ED-PPO)

ED integrated with [[ppo|PPO]], on Google Brax continuous control + **Craftax** (open-ended exploration):

- **HalfCheetah** — ED-PPO *outperforms* BP-PPO; matches DFA-PPO
- **Ant** — on par with both PPO variants
- **Humanoid / Craftax** — trails BP-PPO
- **DFA-PPO is the weakest on Craftax** — random feedback pathways that suffice for supervised classification **fail on complex open-ended RL**, mirroring the cross-task reversal pattern

ED-PPO shows higher variance than BP-PPO, suggesting coarse modular routing is less reliable for fine-grained temporal credit assignment. Notably the RL setting uses ReLU + RMS norm and needs *none* of the classification innovations — ReLU's unbounded positive derivative appears to sidestep the attenuation problem.

## Emergent / post-hoc findings

- **Gradient attenuation** from output to first hidden layer, visible from epoch 3 — a structural property of sigmoid-gated ED, motivating layer-specific widths.
- **E/I balance self-organizes**: the 3:1 asymmetric init converges toward ~1:1, with a *depth-dependent gradient* (layer 1 → 1.03, layer 2 → 0.90, layer 3 → 0.81). Increasing inhibition with depth is loosely consistent with cortical observations (Markram et al. 2004) — homeostatic balance emerging from learning alone, with no balance-enforcing mechanism.
- **Implicit sparsity**: the non-negative weight floor prunes **37.3%** of weights (weights at the floor cannot recover). Inhibitory cross-stream FC connections are pruned most aggressively (up to **68.8%**), convolutional layers barely (<1–18%) — a structured capacity bottleneck that preferentially suppresses inhibition.

## Why it matters / forward-looking claims

- **Neuromorphic hardware** — non-negative synaptic magnitudes with fixed-sign routing may suit analog, photonic, and synapse-device substrates where physical elements naturally encode non-negative quantities.
- **Free model compression** — the weight floor acts as a natural pruning mechanism.
- **Continual learning conjecture** — a dedicated inhibitory stream structurally dampens large gradient excursions and the sign constraint prevents weight-sign flips, both implicated in catastrophic forgetting.

Fits [[sakana-ai|Sakana]]'s house style: nature-inspired, ALife-adjacent, emergence-over-engineering (cf. [[darwin-godel-machine]], [[sakana-fugu-sakana-ai-2026|Sakana Fugu]]).

## Concepts & entities

- [[dales-principle]] — the biological constraint being enforced.
- [[backprop-free-learning]] — the family (ED, [[direct-feedback-alignment|DFA]], the weight-transport problem).
- [[sakana-ai]] — publisher; [[ppo]] — the RL algorithm ED is integrated with.

## References

- [arXiv:2606.31700](https://arxiv.org/abs/2606.31700) — Diffusing Blame (ALIFE 2026)
- _Original source: `sources/diffusing-blame-yamada-2026.txt` (extracted from arXiv HTML)_
