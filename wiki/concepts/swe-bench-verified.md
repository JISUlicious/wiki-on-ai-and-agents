---
title: SWE-bench Verified
type: concept
created: 2026-09-30
updated: 2026-09-30
sources:
  - swe-bench-jimenez-2023.md
  - swe-bench-live-zhang-2025.md
  - deepseek-r1-2025.md
  - swe-gym-pan-2024.md
  - darwin-godel-machine-zhang-2025.md
  - agent-lightning-v1-harnessed-agentic-rl-he-2026.md
  - lego-rl-harness-native-reinforcement-learning-coding-agents-du-2026.md
  - frognano-training-a-4b-coding-agent-via-online-task-synthesis-kim-2026.md
  - envharness-awakening-static-worlds-huang-2026.md
  - nvidia-oo-agents-furgale-2026.md
  - llm-as-a-verifier-kwok-2026.md
  - acm-agentic-context-management-li-2026.md
  - stop-comparing-llm-agents-without-disclosing-the-harness-zhang-2026.md
  - demystifying-evals-for-ai-agents-anthropic-2026.md
  - evomal-self-poisoning-self-evolving-coding-agents-wu-2026.md
  - agentic-harness-engineering-lin-2026.md
  - swe-refactor-bench-hong-2026.md
status: draft
importance: medium
---

# SWE-bench Verified

**SWE-bench Verified** is a human-filtered subset of **500 instances** from the [[swe-bench]] test set, built in collaboration with OpenAI. The [[swe-bench-live-zhang-2025|SWE-bench-Live]] page dates it to 2024. Annotators checked each instance for a clear problem statement, correct test patches, and a task that is solvable from the information given. The format is unchanged from SWE-bench: the agent gets a real GitHub issue and a Python repository, and it is scored on whether its patch makes the FAIL_TO_PASS tests pass. It became the standard headline number for coding agents. The monthly-refreshed [[swe-bench-live]] is presented as its contamination-resistant successor.

## Recorded scores

Scores depend on the harness as much as on the model, so each row names both.

| System (harness) | Score | Source |
|---|---|---|
| DeepSeek-R1 | 49.2% | [[deepseek-r1-2025\|DeepSeek-R1]] |
| OpenHands + Claude 3.7 Sonnet | 43.20% (vs 19.25% on SWE-bench-Live) | [[swe-bench-live-zhang-2025\|SWE-bench-Live]] |
| Qwen-2.5-Coder-32B, SWE-Gym fine-tune + verifier best-of-N | 7.0 → 20.6 → 32.0% | [[swe-gym-pan-2024\|SWE-Gym]] |
| DGM agent on Claude 3.5 Sonnet | 20.0 → 50.0% | [[darwin-godel-machine-zhang-2025\|DGM]] |
| Qwen3.5-9B, mini-SWE-agent + GRPO | 41.8 → 56.4% | [[agent-lightning-v1-harnessed-agentic-rl-he-2026\|Agent Lightning v1]] |
| Qwen3.5-9B, ReAct → ACM post-trained | 0.489 → 0.530 | [[acm-agentic-context-management-li-2026\|ACM]] |
| LEGO-RL (OpenHands / Claude Code / OpenCode) | 70.4 / 68.2 / 66.6 | [[lego-rl-harness-native-reinforcement-learning-coding-agents-du-2026\|LEGO-RL]] |
| FrogNano-4B (Leaf harness) | 61.5% | [[frognano-training-a-4b-coding-agent-via-online-task-synthesis-kim-2026\|FrogNano]] |
| NOOA + GPT-5.5 xhigh / Opus 4.6 high | 82.2 / 79.8% (leaderboard SOTA then 79.2%) | [[nvidia-oo-agents-furgale-2026\|NOOA]] |
| LLM-as-a-Verifier over a 3-model pool | 78.2% (Claude Opus 4.5 baseline 76.8%) | [[llm-as-a-verifier-kwok-2026\|LLM-as-a-Verifier]] |
| Claude Sonnet 4.6 + EnvHarness skills | 67.2 → 72.4% | [[envharness-awakening-static-worlds-huang-2026\|EnvHarness]] |

Trend claims: [[demystifying-evals-for-ai-agents-anthropic-2026|Anthropic]] says Verified went from about 40% to over 80% in a year, and [[swe-refactor-bench-hong-2026|SWE-Refactor-Bench]] puts frontier agents at 70–80%. [[evomal-self-poisoning-self-evolving-coding-agents-wu-2026|EvoMAL]] uses 153 of the 500 tasks as a testbed for skill poisoning. [[agentic-harness-engineering-lin-2026|AHE]] uses Verified as its transfer benchmark.

## Same model, different numbers

- **Harness swings.** [[stop-comparing-llm-agents-without-disclosing-the-harness-zhang-2026|Zhang et al.]] report scaffold-only variation of 11 points for GPT-5 and 15 for Kimi K2 Thinking on Verified. At step 0, LEGO-RL's single Qwen3.5-35B-A3B scores 64.0 / 62.4 / 57.2 in its three harnesses. See [[harness-vs-model-attribution]].
- **Qwen3.5-9B** starts at 41.8% in Agent Lightning's mini-SWE-agent and 48.9% in ACM's ReAct setup. The harnesses and budgets differ.
- **FrogNano's base model** (Qwen3.5-4B in Leaf) is reported as 37.2%, 39.4% or 43.0% in different sections of the same paper. The same base scores 8.3% in the R2E-Gym harness.
- **Claude 3.7 Sonnet + OpenHands**: 43.20% in the SWE-bench-Live re-run, against more than 60% on the public leaderboard with heavier rollouts.
- **EnvHarness** gives several baselines for one policy: 49.88 with original-environment skills (Tables 3 and 5), and 52.13 for original environments in the scaling study (Fig. 5).
- **DGM**: the Claude 3.7 Sonnet transfer result (19.0 → 59.0%) is written as "SWE-bench" in the source page, so it is left out of the table.

## Contamination and saturation

- **Contamination.** The 43.20 vs 19.25 gap for the same agent on Verified and on fresh issues is the wiki's main evidence that Verified is inflated by leakage or overfitting to its repositories ([[benchmark-contamination]]).
- **Used as the validation set.** FrogNano and LEGO-RL both pick checkpoints on Verified and report results on it, so neither headline number is held out.
- **Saturation.** Frontier scores near 80% leave little headroom, and scaffold effects of 11–15 points are as large as the gaps between models. That is why newer sources turn to SWE-bench Pro, [[swe-bench-live]] and [[terminal-bench]].

## Related

[[swe-bench]] · [[swe-bench-live]] · [[swe-gym]] · [[swe-smith]] · [[benchmark]] · [[benchmark-contamination]] · [[harness-vs-model-attribution]] · [[agent-benchmark]]

## References

- SWE-bench Verified overview (500 human-filtered instances, created with OpenAI): https://www.swebench.com/verified.html
- Dataset card, "a subset of 500 samples from the SWE-bench test set": https://huggingface.co/datasets/princeton-nlp/SWE-bench_Verified
- The wiki sources listed in frontmatter.
