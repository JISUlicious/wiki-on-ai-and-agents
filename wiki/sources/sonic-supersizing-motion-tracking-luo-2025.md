---
title: "SONIC: Supersizing Motion Tracking for Natural Humanoid Whole-Body Control"
type: source
created: 2026-10-24
updated: 2026-10-24
sources:
  - sonic-supersizing-motion-tracking-luo-2025.md
tags:
  - 2025
  - paper
status: complete
importance: medium
authors:
  - Zhengyi Luo
  - Ye Yuan
  - Tingwu Wang
  - Chenran Li
  - Fernando Castañeda
  - Linxi Fan
  - Yuke Zhu
introduces:
  - [[sonic]]
  - [[humanoid-motion-tracking]]
year: 2025
venue: arXiv preprint (v4, 2026-08); Science Robotics per secondary coverage
arxiv_id: "2511.07820"
---

# SONIC: Supersizing Motion Tracking

[arXiv:2511.07820](https://arxiv.org/abs/2511.07820), first posted 2025-11; v4 2026-08-13. From [[nvidia]] GEAR, 30 authors advised by Linxi "Jim" Fan and Yuke Zhu. The paper doesn't state a venue; web coverage says Science Robotics 2026.

| | |
|---|---|
| **Project** | GEAR-SONIC |
| **Code** | NVlabs/GR00T-WholeBodyControl |

## Thesis

[[humanoid-motion-tracking|Motion tracking]] (following a reference motion) is a **scalable foundation task for humanoid control**. It gives dense per-frame supervision with no reward engineering. So scale it like a language model, on three axes:
- **Model:** 1.2M → 42M parameters.
- **Data:** 100M+ frames from 700 h of Bones mocap, the same data as [[kimodo|Kimodo]].
- **Compute:** up to 21k GPU-hours (128 GPUs × 7 days).

## How it works

- **RL:** [[ppo|PPO]] in Isaac Lab, with an asymmetric actor-critic (no teacher–student distillation) and broad domain randomization.
- **Universal controller:**
  - Three encoders: robot motion, human SMPL motion, and a hybrid of VR keypoints plus robot legs.
  - They map to a shared **FSQ token**, which a decoder turns into 29 joint PD targets for a **Unitree G1**.
  - FSQ beats VQ-VAE by 8.7 mm MPJPE.
- **Kinematic planner:** SONIC's own masked-token in-betweening model, *not* Kimodo. It plans 0.8–2.4 s segments in about 12 ms on a Jetson Orin.
- **Deployment:** Jetson Orin + TensorRT, 1–2 ms per policy step, 50 Hz control.

## Results

- **Scaling:** the 42M model reaches 99.6% success and 23.8 mm MPJPE, against 98.0% and 27.7 mm at 1.2M.
- **Against other trackers** (success on test-content / test-repetition / PHUMA):
  - SONIC 98.5 / 99.2 / 97.2%
  - BeyondMimic 82.0 / 85.4 / 73.8%
  - Any2Track 61.6 / 69.4 / 78.5%
- **Real G1** (124 motions): 99.2% success, against 100% in simulation.
- **As a VLA backend:** driven by GR00T N1.5, it reaches 75% average success. FSQ tokens beat explicit SMPL poses, 68% vs 27% ([[vision-language-action-model]]).

## Relation to Kimodo

The SONIC paper never mentions Kimodo. The link is in Kimodo's README: a **proof-of-concept demo** where Kimodo generates G1 kinematics and GEAR-SONIC tracks them in simulation. The pattern is the generator → tracker split: a kinematic motion model proposes, a physics policy executes.

## Limitations

- No formal treatment of safety or energy use.
- Can lose balance on extreme motions or noisy planner output.

## Related

[[sonic]] · [[humanoid-motion-tracking]] · [[kimodo-kinematic-motion-diffusion-nvidia-2026]] · [[ppo]] · [[vision-language-action-model]] · [[text-to-motion]]
