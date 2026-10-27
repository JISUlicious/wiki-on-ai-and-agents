---
title: Humanoid Motion Tracking
type: concept
created: 2026-10-24
updated: 2026-10-24
sources:
  - sonic-supersizing-motion-tracking-luo-2025.md
  - kimodo-kinematic-motion-diffusion-nvidia-2026.md
status: draft
importance: medium
---

# Humanoid Motion Tracking

Training a physics-based control policy, usually with [[ppo|PPO]] in simulation, to make a humanoid robot **follow a reference kinematic motion**, then transferring it to the real robot.

## Generator → tracker

It's the second half of a common split:
1. **A kinematic generator proposes the motion.** It can be [[text-to-motion]] (e.g. [[kimodo]], which outputs Unitree G1 joint trajectories as MuJoCo CSV), retargeted human mocap, VR or video.
2. **A tracker executes it** under physics.

## Scaling the tracker

[[sonic|SONIC]] argues tracking is a **scalable foundation task**: dense per-frame supervision, no reward engineering. It scales model, data and compute, from 1.2M to 42M parameters, on 100M+ frames, with 21k GPU-hours.
- **Success:** 99.2% on a real Unitree G1.
- **Against other trackers:** far above BeyondMimic and Any2Track.
- **As a VLA backend:** a shared FSQ token interface lets a [[vision-language-action-model|VLA]] drive it ([[sonic-supersizing-motion-tracking-luo-2025]]).

## Related

[[text-to-motion]] · [[vision-language-action-model]] · [[ppo]] · [[aspire-agentic-skill-programming-robotics-lu-2026]]
