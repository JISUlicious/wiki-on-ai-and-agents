---
title: "Learning Heterogeneous Tactile Representation for General Dexterous Manipulation (LingBot-Touch)"
type: source
created: 2026-11-17
updated: 2026-11-17
sources:
  - lingbot-touch-heterogeneous-tactile-representation-han-2026.md
arxiv_id: "2611.12806"
venue: arXiv preprint (cs.RO)
authors:
  - Bing Han
  - Yichuan Peng
  - Haoyu Dong
  - Kecheng Zheng
  - Yujun Shen
  - Nan Xue
  - Wei Ning
  - et al. (19 authors)
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# Learning Heterogeneous Tactile Representation for General Dexterous Manipulation (LingBot-Touch)

**Source:** Bing Han, Yichuan Peng, Haoyu Dong, … Yujun Shen, Nan Xue, Wei Ning (project lead) (Robbyant) — [arXiv:2611.12806](https://arxiv.org/abs/2611.12806), cs.RO, v1 9 Nov 2026. Preprint. The model is released as **LingBot-Touch**, with Tiny and Small checkpoints.

## Summary

Robots still manipulate almost entirely by sight. Touch has not scaled because every tactile sensor type (piezoresistive, magnetic, visuotactile) has its own channels, layout, resolution and sampling rate. As a result, data from one sensor cannot be pooled with another's. LingBot-Touch is a tactile foundation model that removes the sensor-specific part. Every sensor's taxels are mapped into one **canonical fingertip space**, so a token is addressed by its position on a finger, not by its index in a sensor array. Modality-specific tokenizers feed a shared Haptic Transformer with 5D RoPE.

Training uses **Tactile-JEPA**, which predicts the representations of masked taxels in latent space. This suits tactile signals, which are sparse, mostly silent and unlabeled. A mask warm-up and layout augmentation keep training stable and help transfer across sensors. The corpus is 4,983 hours, 464K episodes and 1.33B frames. About two thirds comes from human tactile gloves and one third from teleoperated Sharpa Wave robot hands, across 24 task categories.

## Key points

- **Plug-in tokens.** On 26 real-world dual-arm tasks, a frozen 5.46M-parameter Tiny encoder raises mean success rates: Diffusion Policy 21.6% → 35.6%, π0.5 48.1% → 69.9%, LingBot-VLA 48.3% → 73.1%, T-Rex 45.4% → 68.6% (+13.9 to +24.8 pp). The encoder adds under 0.2% to policy parameters. Feeding the policies raw tactile signals yields at most about half that gain.
- **Probing contact physics.** On Sharpa Wave the frozen features recover force to 0.81 N, contact centre to 0.12 mm and coverage to 0.66 pp, against 2.31 N, 2.19 mm and 2.90 pp for FTP-1. Touch-to-video top-5 retrieval rises from 16.5% to 72.6%.
- **Unseen sensor.** On a magnetic hand whose sensor model was not in pretraining, it beats FTP-1 with a sensor-specific adapter on all five probes, without retraining.
- **Simulated RL.** Across 12 dexterous tasks, success is 69% with LingBot-Touch features, 46% with a task-specific CNN and 21% without touch, with lower variance across seeds.
- **Latent prediction beats reconstruction.** At 1K hours, contact-centre error is 0.90 mm, against 3.03 mm (VAE) and 3.07 mm (MAE), and retrieval is 32.1% against 4.4% and 5.6%. At 4K hours JEPA improves to 0.32 mm and 49.3%, while the VAE gets *worse* with more data on four of five tasks.
- **Scaling and transfer.** The Small model (21.76M) improves monotonically from 1K to 5K hours on every probe, and data from one sensor improves the representation of another.

## Concepts & entities

- [[jepa]] — Tactile-JEPA applies latent-space masked prediction to touch.
- [[vision-language-action-model]] · [[embodied-agent]] — tactile tokens plugged into π0.5 and LingBot-VLA policies; see [[from-foundation-to-application-vla-wu-2026]] (LingBot-VLA 2.0, same group).
- [[reinforcement-learning]] · [[transfer-learning]] · [[positional-encoding]] (5D RoPE).

## References

- [arXiv:2611.12806](https://arxiv.org/abs/2611.12806)
