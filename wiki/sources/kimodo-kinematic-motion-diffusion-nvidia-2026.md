---
title: "Kimodo: Scaling Controllable Human Motion Generation"
type: source
created: 2026-10-24
updated: 2026-10-24
sources:
  - kimodo-kinematic-motion-diffusion-nvidia-2026.md
  - repos/kimodo-readme-e1c219c.md
  - repos/kimodo-changelog-e1c219c.md
tags:
  - 2026
  - paper
status: complete
importance: medium
authors:
  - Davis Rempe
  - Mathis Petrovich
  - Ye Yuan
  - Haotian Zhang
  - Xue Bin Peng
  - Sanja Fidler
introduces:
  - [[kimodo]]
year: 2026
venue: NVIDIA technical report
arxiv_id: "2603.15546"
---

# Kimodo: Scaling Controllable Human Motion Generation

[arXiv:2603.15546](https://arxiv.org/abs/2603.15546), an [[nvidia]] technical report dated 2026-03-16. Rempe and Petrovich are co-first authors among 24; Sanja Fidler is last.

| | |
|---|---|
| **Repo** | [nv-tlabs/kimodo](https://github.com/nv-tlabs/kimodo), commit `e1c219c` (2026-10-15) |
| **Licences** | code Apache-2.0; checkpoints NVIDIA Open Model License, except SMPL-X under the NVIDIA R&D Model License |

## What it is

[[kimodo|Kimodo]] ("**ki**nematic **mo**tion **d**iffusi**o**n") is a [[text-to-motion]] model for humans and humanoid robots. It is trained on **700 hours** of studio optical mocap, about **20× HumanML3D**.
- **Controls:** text plus **kinematic constraints**: full-body keyframes, hand/foot position and rotation, 2D root waypoints or paths, and foot-contact patterns.
- **What it doesn't need:** no ControlNet, no test-time guidance or optimization, no RL.
- **Positioning:** an *offline authoring* model, at 2–5 s per prompt on an RTX 3090 and at most 10 s of motion per prompt.

## How it works

**Data:**
- **Bones Rigplay** (700 h, 170 subjects) is retargeted to SOMA (77 joints), Unitree G1 and SMPL-X. Each clip has a summary caption plus fine-grained **timeline sub-labels**.
- **Augmentation:** prompts are paraphrased by Qwen3-32B, and random clips are stitched using model-generated transitions.
- **Public subset:** **BONES-SEED** has 288 h. Its timeline labels were released.

**Representation:** a smoothed root position, heading (cos/sin), joint positions relative to that root, global velocities, **global 6D rotations** (which make world-space rotation constraints possible), and 4 foot contacts.

**Denoiser:** a two-stage interleaved transformer.
- The **root stage** predicts the global root. It is converted to a local root, and the **body stage** predicts the rest.
- Each stage has 16 layers, 8 heads and width 1024, for **282M** parameters in total.

**Text encoder:** **LLM2Vec on Llama-3-8B-Instruct**, a 4096-d embedding that "outperform[ed] … CLIP … and T5". It is the reason Kimodo needs about **17 GB of VRAM**; `TEXT_ENCODER_DEVICE=cpu` brings that under 3 GB.

**Constraints:** **imputation**: the known values overwrite the noisy input and a mask is concatenated, following CondMDI.

**Training:**
- DDPM with T = 1000, smooth-L1 loss plus forward-kinematics consistency.
- 1M steps on 16 A100s at batch 2048.
- Curriculum: 500k steps text-only, then 500k with constraints.

**Sampling:** DDIM with 100 steps and **separated CFG**: text and constraint guidance get separate weights, both 2 by default. Gradient guidance helped little and was unstable. Optional foot-lock + IK post-processing is available, and the docs say it doesn't work well on G1.

## Results (ablations only; no external baselines)

| Setting (20 fps) | R@3 ↑ | FID ×100 ↓ | Skate (cm/s) ↓ | Full-body error (cm) |
|---|---|---|---|---|
| Ground truth | 75.6 | — | 2.21 | — |
| **Full model** | **71.9** | **1.85** | **3.87** | **2.67** |
| One-stage denoiser | 71.5 | — | 7.59 | 8.37 |

- **Scaling the model:** 56M → 148M → 282M raises R@3 from 64.0 to 69.2 to 71.9.
- **Scaling the data:** going from 10% to 100% of it cuts the full-body error from 4.60 cm to 2.77 cm.
- **Constraint accuracy at 30 fps:** full-body 3.21 cm, end-effector 3.63 cm / 6.88°.
- **Evaluator:** Kimodo's own [[tmr|TMR]], retrieving over the whole test set.

## Benchmark and its corrections

The **Kimodo Motion Generation Benchmark** has 22,474 test cases on BONES-SEED, in content and repetition splits, with text-only and constraint cases, evaluated by `TMR-SOMA-RP-v1`. Its changelog records three corrections worth knowing:
1. **Averaging bug (fixed 2026-05-03).** Sparse constraint metrics were divided by every motion in the split, so root-path accuracy showed about 0.57 when it was really about 0.93.
2. **TF32 (fixed 2026-10-15).** NGC containers enable TF32, which inflated foot skate by 0.3–0.4 cm/s. The scripts now force full fp32.
3. **Data leak (fixed 2026-10-15).** **Some BONES-SEED test sequences had leaked into Kimodo-SOMA-RP-v1.1's training set.**
   - The v1.2 models retrain on corrected splits; v1.2 is the new default.
   - The scripts now warn when evaluating v1.1.
   - Overview-prompt R@3 on the content split drops from 83.32 (v1.1) to 79.83 (v1.2), against 88.66 for ground truth.

## Robotics hooks

- **G1 output** as MuJoCo qpos CSV, ready for ProtoMotions tracking.
- **SMPL-X output** in AMASS format, for GMR retargeting to other robots.
- **SONIC demo:** a proof-of-concept in which [[sonic|GEAR-SONIC]] tracks Kimodo-generated G1 motion in simulation ([[sonic-supersizing-motion-tracking-luo-2025]]).

## Limitations

- At most 10 s per prompt, and fewer than 20 constrained frames per type.
- Long prompts blur, and conflicting constraints cause artifacts.
- Weak outside the training domain (the docs' example is baseball).
- Offline only. The real-time follow-up is [[ardy]].

## Related

[[kimodo]] · [[text-to-motion]] · [[ardy-autoregressive-motion-diffusion-zhao-2026]] · [[sonic-supersizing-motion-tracking-luo-2025]] · [[tmr-text-to-motion-retrieval-petrovich-2023]] · [[human-motion-diffusion-model-tevet-2022]] · [[unimate-unified-skeleton-animation-mou-2026]]
