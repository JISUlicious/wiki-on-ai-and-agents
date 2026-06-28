---
title: World Model
type: concept
created: 2026-06-19
updated: 2026-06-19
sources:
  - cosmos-3-omnimodal-world-models-for-physical-ai-nvidia-2026.md
  - latent-spatial-memory-for-video-world-models-wang-2026.md
  - learn-from-your-own-latents-korchinski-2026.md
  - agentic-world-modeling-foundations-capabilities-laws-chu-2026.md
status: draft
importance: medium
tags:
  - 2026
---

# World Model

A **world model** is a learned internal model of an environment's dynamics — it predicts how the world (or a latent abstraction of it) evolves under actions, so an agent can simulate, plan, and act rather than merely react. The defining shift from token prediction is **predicting representations / future states** instead of surface symbols; this connects world models to [[latent-reasoning|latent reasoning]] and to the JEPA/data2vec line of self-supervised learning.

## Strands in this wiki

- **Physical AI / robotics** — [[cosmos-3-omnimodal-world-models-for-physical-ai-nvidia-2026|Cosmos 3]] (NVIDIA): an *omnimodal* world model unifying perception, reasoning, simulation, and action generation for robot policies.
- **Video world models** — [[latent-spatial-memory-for-video-world-models-wang-2026|Mirage]]: a video world model with a **latent spatial memory** for 3D-consistent generation at far lower GPU cost than RGB-memory baselines.
- **Theory of latent prediction** — [[learn-from-your-own-latents-korchinski-2026]]: a sample-complexity argument for *why* predicting your own latents (JEPA-style) is exponentially more data-efficient than token prediction.
- **Agentic world modeling** — [[agentic-world-modeling-foundations-capabilities-laws-chu-2026]]: foundations/capabilities/laws for world models built for agents.

## Why it matters

World models are the bridge between language agents and **embodied / physical AI**: an agent that can predict the consequences of actions in a learned model can plan over long horizons and transfer across tasks. The 2026 trend is toward **latent** world models (predict compact state, not pixels/tokens), which are cheaper and expose compositional structure directly.

## Related

- [[latent-reasoning]] — reasoning in continuous latent space; world models predict latent *states*.
- [[memory-management]] — Mirage's "latent spatial memory" is a world-model-specific memory substrate.
- [[scaling-laws]] — latent-prediction objectives change the data-efficiency frontier.

## References

- [[cosmos-3-omnimodal-world-models-for-physical-ai-nvidia-2026]] · [[latent-spatial-memory-for-video-world-models-wang-2026]] · [[learn-from-your-own-latents-korchinski-2026]] · [[agentic-world-modeling-foundations-capabilities-laws-chu-2026]]

## Related 2026 sources

Surfaced via newsletter ingests; see [[index]].

- [[can-llm-agents-infer-world-models-menaged-2026]] — world-model inference as agentic DFA learning
- [[looped-world-models-facemind-2026]] — LoopWM: first looped-transformer world model
