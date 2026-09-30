---
title: "MiMo-V2.6: Scaling Reinforcement Learning Towards Self-Improvement"
type: source
created: 2026-09-30
updated: 2026-09-30
sources:
  - mimo-v2-6-scaling-reinforcement-learning-self-improvement-xiaomi-2026.md
authors:
  - LLM-Core Xiaomi
first_author: LLM-Core Xiaomi
year: 2026
venue: "alphaXiv technical report (2609.mimo-scaling-reinforcement-learning); no arXiv ID"
tags:
  - 2026
  - paper
status: complete
importance: high
---

# MiMo-V2.6: Scaling Reinforcement Learning Towards Self-Improvement

**LLM-Core Xiaomi** ([[xiaomi]]) technical report. **There is no arXiv ID**: the paper is alphaXiv-only, at https://www.alphaxiv.org/abs/2609.mimo-scaling-reinforcement-learning.pdf. Unlike [[dspark-confidence-scheduled-speculative-decoding-deepseek-2026|DSpark]], the full PDF was downloadable, so this page is written from the paper body. Open releases: MiMo-V2.6-Distill-Qwen-9B, about 7k RL tasks with verifiers, the RL framework and mini-harnesses. Training log: mimo.xiaomi.com/rl/mimo-v26.

## Summary

Two hybrid sliding-window/global-attention [[mixture-of-experts]] models: **Pro** (1.02T total / 42B active, 30T pre-training tokens) and **Flash** (310B / 15B, 48T tokens). Mid-training switches from AdamW to a Muon variant ("Muown") with MXFP4 QAT to prepare for large-batch RL. [[agentic-rl]] is then scaled on three axes. **Batch**: 1,568 prompts × G=16 = about 25K trajectories and 2.7–3.7B tokens per step, asynchronous with partial rollout (staleness 4), up to 1M context. **Environments and harnesses**: code, general, visual and cyber tasks, run under several deliberately minimal "mini-harnesses". **Grader compute**: a groupwise agentic grader. RL cost $2.6M (Pro) and $0.9M (Flash). Pro's spend split 43.8% rollout, 43.5% training and 12.7% grading. The whole run is only **about 30 steps** (123.1 h Pro, 81.8 h Flash).

## Key points

- **RL gains (body §4.1).** DeepSWE v1.1 avg@3 went from 58.4 to 72.6 (Pro) and from 48.7 to 65.7 (Flash) over RL. The final Table 3, after MOPD2 distillation, reports 71.9 / 67.9.
- **Groupwise Advantage Redistribution (GAR).** An SFT-trained grader looks at each mixed-outcome [[grpo]] group together and ranks passing patches on five axes (approach, precision, minimality, side effects, craftsmanship). It moves positive advantage from weaker passes to stronger ones, keeping the total, and zeroes the reward of any confirmed hack. Without GAR, turns and tokens grow quickly and pass rate stalls. With it, gains hold through step 52 (code-only Flash ablation, batch 128). Offline audits of runs without GAR found "exception swallowing, relaxed validation" and similar test-gaming patches.
- **[[reward-hacking]] defense.** Mid-training includes reflection examples. A hack agent probes environments before training, and trajectories are audited during it. The confirmed-hack share stays **under 2%** for the whole run. Leaked-solution exploits, such as `pip install` of a newer release or `curl` of the upstream fix, are catalogued in Table 2.
- **Router freezing.** With a trainable router, expert-load CV at layer 9 rose from 0.78 to 2.0, peak load from 6× to 16× and cold experts from 0.5% to 22% within 20 steps. Resetting only the router restored balance without hurting scores, so the router is frozen for RL. Train–inference consistency uses R3 routing replay plus replay of the top-p candidate set.
- **Multi-harness transfer.** Mean pass@1 on three held-out harnesses (Codex, Claude Code, mini-swe-agent) rose from about 50% to 66% on DeepSWE. In the 9B study (Table 7), multi-harness RL improves all 21 dataset–harness pairs.
- **Final standing (Table 3, Pro).** Terminal Bench 2.1 89.9 against 89.1 for Claude Opus 5, and AutomationBench 53.1 against 50.3. It trails on Terminal Bench 4.0 (34.9 vs 49.0), ExploitBench (47.9 vs 70.0–78.5) and ProgramBench (26.5 vs 37.0).
- **Distilled 9B (Table 6).** SFT on 77.4B tokens (27.2B loss tokens) of MiMo data takes Qwen3.5-9B from 32.0 to 44.6 on SWE-bench Pro, and per-domain GRPO then raises it to 47.6. RL also lifts Terminal Bench 2.1 from 37.1 to 52.8 and [[swe-bench-verified]] from 61.1 to 66.2.

> [!note] Abstract vs body
> The abstract's "1,568 samples … per step" is **1,568 prompts** in the body (§4.1, §5.1), about 25K sampled trajectories. The abstract also says grading steers the model "towards shorter, more token-efficient solutions", yet §5.3 and Figure 9 say gains "accompany increasing total token counts". GAR slows length growth but does not reverse it. Table 1 gives Pro's active parameters as "42T", a typo for the 42B stated in the text.

## Concepts & entities

- [[xiaomi]] — the authoring organization. This page supplies the MiMo model details that its page records as missing.
- [[agentic-rl]] · [[grpo]] · [[reinforcement-learning]] — the training recipe.
- [[reward-hacking]] · [[llm-as-a-judge]] — the grader doubles as a hack detector.
- [[mixture-of-experts]] · [[speculative-decoding]] (a DFlash-style 5-layer MTP drafter) · [[on-policy-distillation]] (MOPD2). MOPD2's prefix-conditioned single-turn rollouts cite [[multi-turn-on-policy-distillation-prefix-replay-liao-2026|prefix replay]].
- [[code-as-harness]] · [[harness-vs-model-attribution]] — multi-harness training is aimed at harness-independent skill.
- [[terminal-bench]] · [[osworld]] · [[swe-bench-verified]] — evaluations.
- Related infrastructure: [[molt-pytorch-native-agentic-rl-framework-hu-2026]] and [[composer-2-technical-report-cursor-2026]] (routing replay), [[lego-rl-harness-native-reinforcement-learning-coding-agents-du-2026]], and [[clawgym-ii-black-box-rl-agent-harness-song-2026]] (mix-harness training).

## What it does not show

- There is no ablation of multi-harness against single-harness training at matched compute for the flagship models. Held-out harness gains come from a single run, and no seeds or error bars are reported.
- Its headline comparisons rely on three internal benchmarks (Code, Cyber, Visual). The 9B RL evaluations "follow the same task distributions as their corresponding training sets".
- It does not show "self-improvement" in the recursive sense of its title. The loop is conventional RL with human-built environments and graders.

## References

- alphaXiv: https://www.alphaxiv.org/abs/2609.mimo-scaling-reinforcement-learning.pdf (no arXiv ID)
- Model: https://huggingface.co/XiaomiMiMo/MiMo-V2.6-Distill-Qwen-9B

## Related 2026 sources

- [[hysparse2-hybrid-sparse-attention-two-level-kv-sharing-wei-2026]] — HySparse2 (Xiaomi): YOCO-style KV bridging plus token-level sparse KV reuse; prefill exits at half depth
