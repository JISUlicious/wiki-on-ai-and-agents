---
title: "Masked Visual Actions for Unified World Modeling"
type: source
created: 2026-08-05
updated: 2026-08-05
sources:
  - masked-visual-actions-alzayer-2026.md
arxiv_id: "2607.19343"
year: 2026
authors:
  - Hadi Alzayer
  - Wenlong Huang
  - Lvmin Zhang
  - Maneesh Agrawala
  - Li Fei-Fei
tags:
  - 2026
  - paper
status: complete
importance: low
---

# Masked Visual Actions for Unified World Modeling

**Hadi Alzayer, Wenlong Huang, Lvmin Zhang, Maneesh Agrawala, … Jiajun Wu, Li Fei-Fei** (Stanford University; University of Maryland College Park; Harvard) — [arXiv:2607.19343](https://arxiv.org/abs/2607.19343), cs.CV, 21 Jul 2026. Preprint; no venue listed in the arXiv comments.

## Summary

A robotics world-model paper (peripheral to this wiki's LLM focus, filed for the world-model connection). The idea: express robot **action as a pixel-space signal** — a *partially revealed spatiotemporal trajectory* of some entity in the video — so that conditioning is native to the visual space in which a pretrained video model learned its interaction priors. Revealing the *robot's* motion makes the model a **forward dynamics model** (predict the scene's response); revealing the desired *object* motion makes the same checkpoint an **inverse model** (synthesize the robot behavior that produces that outcome). Prior unified video-action models mask *modality channels* (action vector vs. video), so actions stay low-dimensional and embodiment-specific; masking here is *spatial*, so active and passive entities share one pixel canvas and the same switch also crosses the embodiment gap.

Implementation is a LoRA finetune (rank 256, batch size 4) of **Wan-Fun-Control 2.2 14B**, with the masked conditioning video encoded by the same autoencoder and concatenated — trained on only **15 hours** of masked examples from real and simulated data, one checkpoint for all experiments.

## Key points

- **Video fidelity vs. baselines** (DROID = seen embodiment, BEHAVIOR = unseen bimanual embodiment): Ours 0.095 LPIPS / 0.887 SSIM / **23.74 PSNR** on DROID vs. Ctrl-World 0.362 / 0.708 / 18.15; on BEHAVIOR 0.123 / 0.843 / **22.90** vs. Ctrl-World 0.196 / 0.837 / 18.39. Wan-move (track-conditioned, GT tracks) and Wan2.2 image-to-video are far behind (12.4–13.2 PSNR on DROID).
- **Dense beats sparse conditioning where it matters.** On in-distribution DROID, end-effector-pose (22.64 PSNR) and skeleton (22.74) conditioning nearly match masked actions (23.74). The gap opens on shift: unseen gripper in the real world 20.32 / 21.02 vs. **22.79**, and unseen BEHAVIOR embodiment 19.23 / 19.58 vs. **22.90**.
- **Policy evaluation.** Simulated success rate inside the video model tracks ground-truth RoboCasa success with **r = 0.982** across tasks (open-loop diffusion policy, 10 rollouts/scene), though the model shows a consistent **positive bias toward task progress**. Four real-world tasks (20 demonstrations each, per-task progress rubric) show closely matching progress distributions with the same optimistic bias.
- **Model-based planning.** Best-of-N (N = 10, 10 scenes/task) over Diffusion Policy samples, rolled out in the video model and ranked by a **Gemini 3.1 Pro critic** on task success, interaction fidelity, and physical realism. Success rate rises with the number of sampled actions — a test-time-scaling pattern with policy+video model as generator and VLM as verifier.
- **Zero-shot inverse modeling.** Trained only on forward examples, the model nonetheless handles the inverse query without inverse-specific finetuning. On RoboCasa CoffeeServeMug (20 trials; IDM and baselines trained on 100 demonstrations, video model never trained on the task), action extraction hits **90%** success vs. Diffusion Policy 50%, ACT 80%, SmolVLA 85%.
- Stated limitation: the model captures correlation in object interaction rather than causal structure.

## Concepts & entities

- [[world-model]] — the paper's claim is that forward and inverse dynamics are not separate architectures but different conditional queries against one learned interaction prior.
- [[from-foundation-to-application-vla-wu-2026]] — same vision-language-action lineage; this is the video-model-as-simulator branch rather than the policy branch.
- [[infinite-worlds-versatile-interactions-gao-2026]] — adjacent robot-interaction-data work; here the "environment" is a finetuned video model rather than a simulator.

## References

- [arXiv:2607.19343](https://arxiv.org/abs/2607.19343)
