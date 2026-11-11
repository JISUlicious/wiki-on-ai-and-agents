---
title: "NorthCUA Technical Report: Turbocharging Computer-Use Agents through Experience at Scale"
type: source
created: 2026-11-10
updated: 2026-11-10
sources:
  - northcua-technical-report-zhang-2026.md
arxiv_id: "2611.02102"
venue: arXiv preprint (cs.LG); technical report
authors:
  - Chaoyun Zhang
  - Bingxuan Li
  - Pinxue Guo
  - NorthCUA Team (Shanghai Qiji Zhifeng Co., Ltd.)
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# NorthCUA Technical Report

**Source:** NorthCUA Team, Shanghai Qiji Zhifeng Co., Ltd. (arXiv lists Chaoyun Zhang first) — [arXiv:2611.02102](https://arxiv.org/abs/2611.02102), cs.LG, v3 4 Nov 2026. Weights on Hugging Face (beipo-ai/NorthCUA-Pro); harness code released.

## Summary

An open-weight [[computer-use-agent]] family (Nano 9B dense, Mini 35B, Pro 397B MoE), all built on Qwen3.5 ([[qwen]]). The central claim is that capability comes from *co-designing how experience is produced and how it is used*. Three parts form one loop. The **NorthCUA Harness** runs desktop and web tasks end to end, records trajectories, and checks outcomes with task verifiers plus an evaluator that inspects the live workspace. The **Experience Flywheel** produces grounding data and executable tasks, and curates verified trajectories. **Staged Learning** turns that experience into a policy, which returns to the same harness for more collection.

## Key points

- **Flywheel routes.** *Scaling Intents* (requests derived from available software and content), *Scaling Environments* (build workspaces, reconstruct reference sites), *Scaling Verifiers* (vary request, initial state and checker together). Grounding pairs combine visual/structural annotation with app documentation.
- **Hard tasks via [[recursive-self-improvement|RSI]].** Skill-guided exploration keeps tested procedures for later attempts ([[skill-library]]). Milestone-guided search snapshots software state and agent context at verified milestones so later attempts resume from progress.
- **Staged Learning.** (1) Broad GUI mid-training on grounding plus successful trajectories (over 4M interaction steps across dozens of apps). (2) Verified long-horizon [[supervised-fine-tuning|SFT]] on complete workflows that pass executable checks. (3) Joint desktop–web [[agentic-rl]] with [[grpo|GRPO]] on verifier outcomes, keeping valid failures as well as successes.
- **Results (reported settings).** Pro: ScreenSpot-Pro 93.3%, OSWorld-G 91.0% (grounding); [[osworld|OSWorld]]-Verified 88.6% (vs Qwen-CUA-Max 87.6%); OSWorld 2.0 30.6% binary / 66.8% partial; WebArena-Verified 73.9%, above the 71.6% for Kimi-K3 and for [[gpt-6-astra|GPT-6 Astra]] in their own run. Mini: 85.3% OSWorld-Verified, 70.4% WebArena-Verified. Nano: 78.9% / 67.2%. Pro beats trillion-parameter Qwen3.8-Max and Kimi-K3 on grounding, OSWorld-Verified and WebArena-Verified.
- **Weaknesses noted.** Correct refusal when the target is absent stays hard (~75% even for Pro). Comparisons mix source-specific protocols (Appendix I).
- **NorthCUA-Ultrafast.** Pre-RL Nano with a [[jev|Jev]]-inspired *choice head*. Finite decisions (click, hover, checkbox on a known control) are scored in one forward pass and turned straight into tool calls. Open-ended text falls back to generation. It is 2.3× faster on matched successful WebArena-Verified tasks, at some quality cost. This is a concrete use of the [[system-one-model]] / [[logit-scoring]] idea inside an agent loop.

## Concepts & entities

- [[computer-use-agent]] · [[gui-agent]] · [[agent-computer-interface]] · [[osworld]] · [[agent-benchmark]]
- Training: [[agentic-rl]] · [[grpo]] · [[supervised-fine-tuning]] · [[rl-environment-synthesis]] · [[outcome-based-reward]] · [[verifier]]
- Experience reuse: [[skill-library]] · [[self-improving-agent]] · [[recursive-self-improvement]] · [[experiential-learning]]
- Related models/ideas: [[qwen]] · [[ui-tars-2]] · [[jev]] · [[system-one-model]] · [[logit-scoring]] · [[mixture-of-experts]]

## References

- [arXiv:2611.02102](https://arxiv.org/abs/2611.02102)
