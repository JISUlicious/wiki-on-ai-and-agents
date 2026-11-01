---
title: "AutoCompact: Learning When to Compact Context in Long-Horizon Coding Agents"
type: source
created: 2026-10-29
updated: 2026-10-29
sources:
  - autocompact-learning-when-to-compact-zhang-2026.md
arxiv_id: "2610.02163"
authors:
  - Xuan Zhang
  - Longtao Zheng
  - Cunxiao Du
  - Bo An
  - Xin Dong
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# AutoCompact: Learning When to Compact Context in Long-Horizon Coding Agents

**Xuan Zhang, Longtao Zheng, Cunxiao Du, Bo An, Xin Dong** (Singapore Management University, Nanyang Technological University, Harvard University) — [arXiv:2610.02163](https://arxiv.org/abs/2610.02163), v1, cs.CL, 1 Oct 2026. Preprint.

## Summary

Coding harnesses such as [[claude-code|Claude Code]] and Codex compact context when it nears the window limit. That ties compaction to length, not to task progress: stale exploration piles up until the threshold, and compaction may then cut through an unresolved stage. AutoCompact trains the agent to decide **when** to compact, **what** working state to keep, and **how** to continue from it, all as part of its policy.

Training data comes from **judge-guided on-policy collection**. The base agent runs on tasks, and a judge (GPT-5.5-Codex) reviews each compaction decision, each summary and the actions right after compaction. Flawed outputs are replaced with corrected ones *before* execution, so the trajectory continues from the corrected state. The 1,052 corrected trajectories (from SWE-rebench) are used for SFT, followed by outcome-reward RL that jointly optimizes coding and compaction. No judge is used at inference.

## Key points

- **Base model:** Qwen3-Coder-30B-A3B-Instruct, one terminal-based scaffold for all variants.
- **Main result:** 39.6% on [[swe-bench-verified|SWE-bench Verified]] and 24.5% on SWE-PolyBench Verified, **+9.2 and +5.0 points absolute** over the base model (full runs, no cost cap).
- **Length-triggered compaction hurts here.** Fixed compaction is 1.6 and 0.9 points *below* the base model. CompactionRL recovers 3.9 and 1.2 points over fixed compaction but stays close to the base.
- **Proactive compaction helps even when nothing overflows.** At a 256K window that no trajectory fills, SelfCompact, SWE-Compressor and AutoCompact all beat full history. AutoCompact-SFT beats SWE-Compressor by 1.2 and 1.6 points with comparable data, which the authors attribute to the online correction.
- **Gains hold at every inference budget tested** ($0.10–$4.00 per task; cached tokens priced at 20%), both at 256K and at a 16K window with a forced-compaction fallback.
- **Following the summary matters.** The same checkpoint scores lower when made to ignore its own summaries, so the gain is not from training alone.

The newsletter's +9.2 / +5.0 figures match the paper (absolute points).

## What it does not show

- One base model and one scaffold. RL used 32K-token sequences, shorter than the 256K evaluation window (the authors flag this).
- The judge is a much stronger proprietary model, so part of the gain may be distillation of GPT-5.5-Codex's judgment rather than compaction skill per se. There is no ablation with a weaker judge.
- No comparison against the production compaction of Codex or Claude Code themselves; the authors leave combining with those to future work.

## Concepts & entities

- [[context-engineering]] · [[memory-management]] · [[context-rot]] — when to compact is a learned decision, not a threshold.
- [[compaction-cliff-long-running-agent-memory-zerhoudi-2026]] · [[lost-in-compaction-side-constraint-loss-wang-2026]] — evidence that threshold compaction loses information.
- [[learning-agent-compatible-context-management-yi-2026]] · [[acm-agentic-context-management-li-2026]] — other learned context-management policies.
- [[context-language-models-shao-2026]] — the unrestricted version: the model may edit any part of its context.
- [[agentic-rl]] · [[supervised-fine-tuning]] · [[llm-as-a-judge]] · [[qwen]] · [[swe-bench-verified]].
- [[context-compaction]]: concept page on when, what and who compacts agent context, and what compaction loses.

## References

- [arXiv:2610.02163](https://arxiv.org/abs/2610.02163)
