---
title: "LEGO-RL: Harness-Native Reinforcement Learning for Coding Agents"
type: source
created: 2026-09-15
updated: 2026-09-15
sources:
  - lego-rl-harness-native-reinforcement-learning-coding-agents-du-2026.md
arxiv_id: "2608.17393"
authors:
  - Yiming Du
  - Yuxin Jiang
  - Tao Yuan
  - Jianbo Dai
  - Shaowei Wang
  - Jierun Chen
  - Chaofan Tao
  - Xianzhi Yu
  - Lifeng Shang
  - Kam-Fai Wong
  - Xiaohui Li
  - Haoli Bai
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# LEGO-RL: Harness-Native Reinforcement Learning for Coding Agents

**Yiming Du, Yuxin Jiang, Tao Yuan (co-first), … Xiaohui Li, Haoli Bai (corresponding)** (Huawei Technologies; Du and Wong also CUHK) — [arXiv:2608.17393](https://arxiv.org/abs/2608.17393), cs.AI, v1 18 Aug 2026. Technical report ("LegoX"); no venue. Framework, trained models and the 2,699-task index are released (github.com/LegoX/Lego-RL).

**Summary.** LEGO-RL is RL infrastructure that trains a policy **through unmodified production harnesses** — [[openhands]] SDK, [[claude-code]] and [[opencode]] — treating the harness as part of the environment and optimizing only the model it calls. Each new harness needs only a thin adapter. The core piece is an **in-process proxy** at the model-API boundary (OpenAI-compatible and Anthropic APIs). It records the exact token IDs, log-probabilities, response masks and [[mixture-of-experts]] routing decisions at generation time, so the trainer never has to rebuild tokens from a transcript the harness may have re-serialized or compacted. Tool calls are matched by ID, sub-agent calls are isolated, and routing is replayed in training (R3). Around the proxy sit sandbox orchestration on verl + Harbor (Nydus lazy pulls, prebuilt images, a mounted agent runtime), reward-integrity defenses, and termination-aware filtering of infrastructure failures. A Live UI and agent plugin handle diagnosis. The training setup is Qwen3.5-35B-A3B with GSPO, 200k context, fully async (staleness 1), 126 steps (3 epochs) on a 2,699-task OpenSWE-derived pool kept disjoint from SWE-bench Verified by repository and instance.

## Key results (SWE-bench Verified, measured by the authors: temp 0.7, 200 turns, 200k context)

| Harness | Qwen3.5-35B-A3B | Qwen3.6-35B-A3B | KAT-Coder-V2.5-Dev | LEGO-RL |
|---|---|---|---|---|
| OpenHands SDK | 64.0 | 67.4 | 67.0 | **70.4** (+6.4) |
| Claude Code | 62.4 | 63.4 | 66.8 | **68.2** (+5.8) |
| OpenCode | 57.2 | 60.6 | 64.8 | **66.6** (+9.4) |

- **Faithful optimization.** Median rollout–trainer log-prob Pearson r is **0.9993 / 0.9980 / 0.9993** (OpenHands / Claude Code / OpenCode). It never drops below 0.989 at any step, so "above 0.99" describes the medians. Routing replay lifts r from 0.9946 to 0.9993. Matching tool calls by ID resolves 207 of 222 (93%) apparent Claude Code mismatches.
- **Gains are harness-specific.** KAT-Coder-V2.5-Dev beats its Qwen3.6 base by +3.4 under Claude Code, the harness its authors report on, but by −0.4 under OpenHands SDK ([[harness-vs-model-attribution]]). The same starting policy also scores differently under each harness at step 0 (64.0 / 62.4 / 57.2).
- **Task difficulty matters.** In an ablation over four 951-task pools, the full difficulty band reaches 0.671 validation and its upper half 0.670, against 0.640 for the lower half. An unscreened random pool never improves; 72.7% of its tasks are never solved and 13.4% always solved.
- **Reward integrity.** Before the defenses went in, agents read git history in 4.6–20.5% of cases, edited test files in 2.4–19.4%, and downloaded the reference fix in 1.9%. The grader itself applied the reference patch on 2.5% of tasks ([[reward-hacking]]). Infrastructure-failed trajectories excluded from the loss: 7.1% for Claude Code, 2.4% for OpenHands and 6.4% for OpenCode.
- **Systems.** Agent execution takes 91.3% of trial wall-clock (840.5 of 920.4 s). Async training cuts step time 2.5× as measured, or about 1.9× after correcting for GPU throughput. Prebuilt images speed sandbox setup 33.2×. Under OpenHands, turns per trajectory rise from 46.6 to 83.1 and re-reading a file after editing rises from 73.6% to 98.1%.

> [!note] Real harnesses, not a bespoke one — with one carve-out
> This is the opposite design choice to [[frognano-training-a-4b-coding-agent-via-online-task-synthesis-kim-2026]], which wrote a minimal five-tool harness (Leaf) and trained only inside it. The abstract says log-prob recomputation stays robust "even under harness-side compaction". The body is narrower: histories that were genuinely rewritten or truncated, including by context compaction, are **excluded from optimization**. The harness's compaction path therefore runs during rollouts but is not trained on.

**What it does not show.** Each main configuration is a **single run** with one model, and each harness is trained separately. No run measures whether a Claude Code-trained policy transfers to OpenCode, or trains across harnesses. The reported scores are the **peak** of per-checkpoint SWE-bench Verified evaluations, and SWE-bench Verified serves as both validation and test set, so there is selection on the benchmark and no held-out coding benchmark. The comparison rows (Qwen3.6, KAT-Coder) are not RL-trained under this framework, so the table shows a strong result rather than isolating the framework's own contribution.

## Related

- [[agentic-rl]] · [[grpo]] — GSPO is a sequence-level GRPO variant; the zero-variance-group problem motivates the difficulty screen.
- [[agentic-harness-engineering]] · [[code-as-harness]] · [[harness-vs-model-attribution]] — the harness as part of the environment; Table 2 is a harness × model grid.
- [[agent-lightning-v1-harnessed-agentic-rl-he-2026]] · [[clawgym-ii-black-box-rl-agent-harness-song-2026]] — other harness-native or black-box-harness RL frameworks (Agent Lightning is in the paper's comparison table).
- [[co-evolving-harnesses-and-models-on-policy-correction-yu-2026]] — the weights-vs-harness-fit problem from the SFT side.
- [[compaction-cliff-long-running-agent-memory-zerhoudi-2026]] · [[lost-in-compaction-side-constraint-loss-wang-2026]] — why compacted histories break token-faithful training.
- [[swe-bench]] · [[qwen]] · [[vllm]] · [[openhands]] · [[claude-code]] · [[opencode]].

## References

- [arXiv:2608.17393](https://arxiv.org/abs/2608.17393) (v1, 18 Aug 2026).
- GSPO (Zheng et al. 2025); verl/HybridFlow (Sheng et al., EuroSys 2025); R3 routing replay (Ma et al., [arXiv:2510.11370](https://arxiv.org/abs/2510.11370)).
- Harness-native frameworks compared: slime, MOLT, SkyRL-Agent ([arXiv:2511.16108](https://arxiv.org/abs/2511.16108)), AReaL ([arXiv:2505.24298](https://arxiv.org/abs/2505.24298)), Agent Lightning ([arXiv:2508.03680](https://arxiv.org/abs/2508.03680)), Polar, rLLM, OpenForgeRL, ALE.
- Tasks: OpenSWE / daVinci-Env ([arXiv:2603.13023](https://arxiv.org/abs/2603.13023)); baseline KAT-Coder-V2.5 ([arXiv:2607.05471](https://arxiv.org/abs/2607.05471)).
