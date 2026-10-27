---
title: SONIC (GEAR-SONIC)
type: entity
created: 2026-10-24
updated: 2026-10-24
sources:
  - sonic-supersizing-motion-tracking-luo-2025.md
tags:
  - model
status: draft
importance: medium
family: gr00t
parameters: 42M (largest policy)
---

# SONIC (GEAR-SONIC)

[[nvidia]] GEAR's humanoid **[[humanoid-motion-tracking|motion-tracking]] foundation policy** for the Unitree G1. It scales the policy to 42M parameters, with 100M+ frames of training motion and 21k GPU-hours of PPO. It reaches 99.2% success on a real robot. It can be driven by VR, video, text or a [[vision-language-action-model|VLA]] (GR00T N1.5) through a shared FSQ token. Paper: [[sonic-supersizing-motion-tracking-luo-2025]].

## Lineage

It trains on the same Bones mocap as [[kimodo]]. In Kimodo's demo, SONIC is the **tracker** that executes Kimodo-generated G1 kinematics. SONIC's own planner, though, is a separate masked-token in-betweening model.

## References

- [[sonic-supersizing-motion-tracking-luo-2025]]
