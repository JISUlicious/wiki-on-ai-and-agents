---
title: Embodied Agent
type: concept
created: 2026-05-21
updated: 2026-09-30
sources:
  - magma-yang-2025.md
status: draft
importance: medium
tags:
  - agent
---

An **embodied agent** is the physical-world counterpart to a [[computer-use-agent]]: a policy that perceives via cameras / sensors and acts on a robot body or simulated environment with continuous motor commands. Vision-language-action models like [[magma]] aim to unify GUI agents and embodied agents under a single multimodal foundation policy.

- Perception: RGB(-D) cameras, proprioception, sometimes tactile / audio
- Action space: joint torques, end-effector poses, gripper open/close
- Modern instances use [[vision-language-action-model]] backbones
- [[magma]]'s [[trace-of-mark]] provides motion supervision aligned with this setting

Related: [[vision-language-action-model]], [[magma]], [[computer-use-agent]], [[trace-of-mark]]

## References
- [[magma-yang-2025]]

## Related 2026 sources

- [[rl-for-real-time-vla-policies-dong-2026]] — Real-Time EXPO-FT: a fast RL edit policy corrects stale asynchronous VLA action chunks; 29/30 on real robots
- [[gavel-graph-world-models-llm-task-planning-wang-2026]] — GAVEL: a hand-specified scene-graph world model verifies and repairs LLM plans; Qwen3-8B 41.2%→91.8% on BEHAVIOR-1K
