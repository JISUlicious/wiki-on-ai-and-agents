---
title: "Human-like Autonomy Emerges from Self-Play and a Pinch of Human Data"
type: source
created: 2026-06-29
updated: 2026-06-29
sources:
  - human-like-autonomy-self-play-pinch-human-data-cornelisse-2026.md
arxiv_id: "2606.19370"
authors:
  - Daphne Cornelisse
  - Julian Hunt
  - Zixu Zhang
  - Jaime Fernández Fisac
  - Waël Doulazmi
  - Kevin Joseph
  - Eugene Vinitsky
first_author: Daphne Cornelisse
year: 2026
venue: arXiv preprint
tags: [2026]
status: complete
importance: medium
---

# Human-like Autonomy Emerges from Self-Play and a Pinch of Human Data

Surfaced via the NLP newsletter (Top AI Papers of the Week, 2026-06-21–06-28).

## Summary

Self-play [[reinforcement-learning]] can train driving policies without human data by substituting cheap large-scale simulation for expensive human demonstrations — but pure self-play discovers driving conventions that are effective yet *alien*, incompatible with human road users. Prior fixes lean on brittle, labor-intensive reward engineering and domain randomization. This work (NYU Tandon, NYU Courant, Princeton, Mines Paris/Valeo) instead keeps human demonstrations and uses them as a lightweight **regularization objective** layered on a minimal safe goal-reaching reward. The slogan: like spice in a stew, a little human data goes a long way. With only ~30 minutes of human demonstrations, the resulting policies coordinate with held-out human trajectories.

## Key points

- Human data used as a **regularizer** on top of a minimal safe goal-reaching reward, not as reward engineering or domain randomization.
- Uses only **~30 minutes** of human demonstrations — **2500× fewer** than comparable imitation-learning approaches.
- Anchored to the **Waymo Open Motion Dataset**; ~30 min human data paired with roughly **60 years** of synthetic self-play experience.
- Resulting policies **coordinate with held-out human trajectories** (human-replay proxies), addressing the alien-conventions problem of pure self-play.
- Trains in **~15 hours on a single consumer-grade GPU**; videos and source code released.

## Concepts & entities

- [[reinforcement-learning]] — self-play RL as the training paradigm, regularized toward human conventions.

## References

- [arXiv:2606.19370](https://arxiv.org/abs/2606.19370) — Cornelisse, Hunt, Zhang, Fisac, Doulazmi, Joseph, Vinitsky. "Human-like Autonomy Emerges from Self-Play and a Pinch of Human Data" (2026).
