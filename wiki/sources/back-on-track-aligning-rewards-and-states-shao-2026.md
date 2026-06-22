---
title: "Back on Track: Aligning Rewards and States for Reasoning in Diffusion Large Language Models"
type: source
created: 2026-06-22
updated: 2026-06-22
sources:
  - back-on-track-aligning-rewards-and-states-shao-2026.pdf
arxiv_id: "2606.08501"
authors:
  - Yawen Shao
  - et al.
first_author: Yawen Shao
year: 2026
tags: [2026]
status: complete
importance: medium
---

# Back on Track: Aligning Rewards and States for Reasoning in Diffusion Large Language Models

*Surfaced via the NLP newsletter (Top AI Papers of the Week, 2026-06-14–06-21).*

## Summary

This paper applies [[reinforcement-learning]] to **diffusion LLMs (dLLMs)** — non-autoregressive models like LLaDA and Dream that predict multiple tokens per denoising step. Naively porting policy-gradient RL to dLLMs suffers a *dual misalignment*: (1) process-reward misalignment, where a sparse terminal reward is assigned indiscriminately to arbitrary intermediate steps with no step-specific credit; and (2) lack of state authenticity, where updates run on "artificial states" built by randomly masking the final completion rather than true intermediate denoising contexts. Random step selection also wastes updates on already-settled decisions.

To fix this, the authors propose **Process Aligned Policy Optimization (PAPO)**, which aligns the RL update with the dLLM's true generative trajectory via two components: **Step-Aware Process Rewards (SPR)**, which transform sparse terminal rewards into dense step-wise credit, and **Entropy-Guided Historical Re-enactment (EHR)**, which replays authentic trajectories at high-uncertainty (high-entropy) steps so learning focuses where the model actually struggles. (Note: the newsletter blurb's "4.5%→42.2% on GSM8K/MATH500" conflates benchmarks — see Key points for the per-benchmark figures.)

## Key points

- Diagnoses two RL failure modes for dLLMs: sparse-reward / process-reward misalignment and unfaithful "artificial" masked states (trajectory drift).
- **SPR** converts sparse terminal rewards into dense, step-aware process rewards for proper credit assignment.
- **EHR** re-enacts authentic historical states at entropy-selected high-uncertainty steps, replacing inefficient random step selection.
- Gains vs. baselines: up to **+4.5% GSM8K**, **+4.8% MATH500**, **+42.2% Countdown**, **+16.1% Sudoku** (four benchmarks).
- Summarized as up to +4.7% on math reasoning and +29.2% on planning tasks over strong dLLM baselines.
- Built on diffusion-LM backbones (LLaDA, Dream); positions process-aligned RL as key to structured reasoning in non-autoregressive models.

## Concepts & entities

- [[reinforcement-learning]]
- [[latent-reasoning]]
- [[cola-continuous-latent-diffusion-language-model-guo-2026]]
- [[elf-embedded-language-flows-hu-2026]]

## References

- [arXiv:2606.08501](https://arxiv.org/abs/2606.08501)
