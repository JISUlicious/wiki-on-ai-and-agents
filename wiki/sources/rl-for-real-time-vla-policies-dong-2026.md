---
title: "Reinforcement Learning for Real-Time Vision-Language-Action Policies"
type: source
created: 2026-09-30
updated: 2026-09-30
sources:
  - rl-for-real-time-vla-policies-dong-2026.md
arxiv_id: "2609.18207"
authors:
  - Perry Dong
  - Kuo-Han Hung
  - Dorsa Sadigh
  - Chelsea Finn
year: 2026
tags:
  - 2026
  - paper
status: complete
importance: medium
---

# Reinforcement Learning for Real-Time Vision-Language-Action Policies

**Perry Dong\*, Kuo-Han Hung\*, Dorsa Sadigh, [[chelsea-finn|Chelsea Finn]]** ([[stanford-university|Stanford]]). [arXiv:2609.18207](https://arxiv.org/abs/2609.18207), cs.RO, 16 Sep 2026. Preprint with no venue given. Project page: pd-perry.github.io/real-time-expo-ft.

## Summary

Large [[vision-language-action-model|VLA]] policies are slow. By the time an action chunk is ready, the observation it was computed from is stale, and a delayed process is no longer Markovian in the current state, which biases RL credit assignment. Real-time chunking (RTC) hides the latency, but it is an imitation-learning method with no way to move beyond the data. **Real-Time EXPO-FT** splits control into two timescales on top of the EXPO-FT off-policy RL framework. (1) **Slow and asynchronous:** the VLA (π0.5 with LoRA and training-time RTC conditioning on the committed action prefix) samples N candidate chunks from the observation d steps before they are needed. (2) **Fast and synchronous:** at execution time a small *edit policy* adds a bounded correction to each candidate using the **latest** observation, and a chunk-level Q-critic picks the argmax over the base and edited candidates. Because the edit sees the current state, the combined policy stays Markovian under delay. A "noise-space" critic filters VLA seeds so that each Bellman backup needs one VLA decode instead of N.

## Key points

- **Kinetix simulation (Table II; 10 tasks, 4-step delay, 4 seeds × 100 episodes).** The average is **96.2%**, against 81.7% for EXPO-FT w/ RTC, 76.1% for DSRL w/ RTC, 80.0% for RTC, and **81.4% for the no-delay RLPD baseline**. The margins over the delayed baselines are +34.5 (DSRL), +20.1 (DSRL w/ RTC), +21.3 (EXPO-FT) and +14.5 (EXPO-FT w/ RTC) points.
- **Real robot (Table I; 30 Hz end-effector control, 30 trials per task, ≤10 min online data, no human interventions, human-verified success).** The average is **29/30**, against 25/30 for EXPO-FT w/ RTC, 20/30 for DSRL w/ RTC, 18/30 for SFT w/ RTC and 4.5/30 for RLPD. Per task: Dynamic Picking 30/30, Ball Balancing 28/30 (the best prior method reaches 23/30), Object Passing 30/30, Soccer Kicking 28/30. Latency is about 67 ms, with 100 ms added on three tasks to give d = 5 (about 167 ms).
- **Delay and speed sweeps (Fig. 6).** Performance stays stable as the delay d grows on H17 Unicycle, where plain RTC degrades. It stays near 100% as object-passing speed rises from 0.5× to 1.25×, where EXPO-FT without real-time editing degrades.
- **Compute asymmetry favours the baselines.** RLPD and DSRL run asynchronously at high update-to-data ratios, while EXPO-FT variants update once per episode.

> [!note] Abstract vs body
> The abstract claims "the best performance among delayed and non-delayed methods in 10 out of 10 environments". Table II bolds every method within **0.95× of the best RL result** and excludes BC and RTC. On raw numbers the method is outright best among RL methods in 7 of 10. It trails EXPO-FT on Cartpole Thrust (98 vs 99) and Hard Lunar Lander (94 vs 95), and DSRL w/ RTC on Chain Lander (92 vs 96). The abstract's "42% to 97%" real-world gain is measured from plain SFT (12.5/30). Against the stronger SFT w/ RTC (18/30, 60%) and EXPO-FT w/ RTC (25/30, 83%) the gap is smaller.

## Concepts & entities

- [[vision-language-action-model]] — the policy class; the base is π0.5 (Gemma-2B LoRA backbone).
- [[reinforcement-learning]] — off-policy actor-critic with a residual edit policy and a chunk-level TD backup.
- [[embodied-agent]] · [[chelsea-finn]] · [[stanford-university]].
- [[from-foundation-to-application-vla-wu-2026]] — the VLA survey context. [[robottt-context-scaling-robot-policies-jiang-2026]] is another 2026 robot-policy adaptation paper.

## What it does not show

- There are no language-conditioned or multi-task evaluations. Every real task has its own demonstrations, its own rule-based success classifier and human resets.
- For Ball Balancing and Soccer Kicking the **critic receives privileged state** from the success detector (ball position and velocity, keeper position), while the VLA does not. That weakens the "from pixels" reading on the two hardest tasks.
- Real-world results are single training runs per method, with no seeds or confidence intervals.

## References

- [arXiv:2609.18207](https://arxiv.org/abs/2609.18207)
