---
title: "RoboTTT: Context Scaling for Robot Policies"
type: source
created: 2026-08-05
updated: 2026-08-05
sources:
  - robottt-context-scaling-robot-policies-jiang-2026.md
arxiv_id: "2607.15275"
year: 2026
authors:
  - Yunfan Jiang
  - Yevgen Chebotar
  - Ruijie Zheng
  - Li Fei-Fei
  - Linxi "Jim" Fan
tags:
  - 2026
  - paper
status: complete
importance: low
---

# RoboTTT: Context Scaling for Robot Policies

**Yunfan Jiang, Yevgen Chebotar, Ruijie Zheng, … Li Fei-Fei, Yuke Zhu, Linxi "Jim" Fan** (NVIDIA; Stanford University; The University of Texas at Austin) — [arXiv:2607.15275](https://arxiv.org/abs/2607.15275), cs.RO, 16 Jul 2026. Preprint; no venue listed in the arXiv comments.

## Summary

A robotics paper (peripheral to this wiki's LLM focus, filed for the long-context/test-time-training angle). Robot foundation models still condition on a single step or a couple of history frames, while context length has become a scaling axis for language models. RoboTTT pushes visuomotor context to **8K timesteps** — three orders of magnitude beyond SOTA policies — **without growing inference latency**, by making the recurrent state a set of **fast weights**: parameters of a small MLP updated by gradient descent at every timestep during *both* training and inference (W_t ← W_{t−1} − η∇_W ‖f_W(K_t) − V_t‖², then O_t = f_{W_t}(Q_t)). History is compressed into weight space rather than a KV cache, so cost stays constant in context length; the fast model is more expressive than an RNN's vector state, and the outer task loss meta-learns both the initialization W₀ and the update dynamics.

Instantiated on pretrained **GR00T N1.7** with a TTT layer added to each of its 16 DiT layers (two-layer MLP per fast model), pretrained for 30K steps on 16 NVIDIA GB200 GPUs over bimanual robot plus egocentric human data, with context length grown gradually to target. Training at long context is made feasible by **sequence action forcing + truncated BPTT**. Two capabilities ride on the long context: one-shot imitation from an in-context human video, and **DAgger Distillation**, which distills failure-to-correction mappings into the fast weights so the policy self-corrects online without human intervention.

## Key points

- **Main evaluation** (three real bimanual assembly tasks — Pup Go Car, Circuit, Gear Bot; 20 trials each, 10 for Gear Bot): average task completion score **79%**, vs. GR00T N1.7 single-step 42% (+87%) and best baseline GDN (Gated DeltaNet) 56% (+41%). Fully successful trials — RoboTTT 9/20, 13/20, **2/10**; GR00T N1.7 3/20, 3/20, 0/10; GR00T N1.7 Hist. 0/20, 8/20, 0/10; GDN 3/20, 8/20, 0/10. Gear Bot averages **five minutes and ten stages**, and RoboTTT is the only method that ever completes it.
- **Context length is a scaling axis.** Sweeping pretraining context 128 → 8K timesteps, completion score rises steadily to **71.5% at 8K vs. 43.9% at 1K (+63%)** and 45.6% for the best short-context baseline (+57%), with no sign of saturation. **GDN shows no such trend** — the authors attribute this to its linear associative update admitting no meta-learning, whereas RoboTTT's gradient-descent fast weights do.
- **Naive history hurts.** GR00T N1.7 Hist. (one extra frame) scores 39.5% on Pup Go Car vs. 57% for the no-history GR00T N1.7 — concatenated history introduces spurious correlations and leaves the robot temporally out of distribution.
- **One-shot imitation** on Circuit (80 configurations; train on 20, test on 60; identical prompt so the target is identifiable only from the video): RoboTTT 65% completion and **6/10** successful assemblies; GDN 33% and **0/10**. Recurrent memory can encode the context but fails to *use* it.
- **Perturbation robustness** (human removes an installed roof or tire mid-episode): RoboTTT recovers 15/20 and 18/20 (≈83% overall) vs. 10/20 and 11/20 for GR00T N1.7 (≈53%); GR00T N1.7 Hist. is worst at 3/20 and 5/20.
- **DAgger Distillation** yields roughly **36%** better performance than the same model without that training, and beats standard DAgger fine-tuning on the full trajectories.
- Qualitative account of *why* long context helps: it disambiguates visually aliased stages of multi-stage assembly, enables strategic retry after a failed sub-step instead of proceeding as if it succeeded, and mitigates partial observability when the target object is occluded. The relevant observation window cannot be specified a priori — the model learns what to retain.

## Concepts & entities

- [[long-context-llm]] — imports the LLM context-scaling thesis into visuomotor control, with the twist that the state is fast weights updated by gradient descent rather than a KV cache, so latency stays flat as context grows.
- [[world-model]] — a policy-side counterpart: rather than simulating futures, it compresses the past rollout into weights and conditions on it.
- [[from-foundation-to-application-vla-wu-2026]] — RoboTTT is built as a modification of a VLA robot foundation model (GR00T N1.7), adding TTT layers inside its DiT stack.

## References

- [arXiv:2607.15275](https://arxiv.org/abs/2607.15275)
