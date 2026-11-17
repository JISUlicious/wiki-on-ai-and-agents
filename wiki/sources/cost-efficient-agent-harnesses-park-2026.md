---
title: "System Optimizations for Cost-Efficient Agent Harnesses"
type: source
created: 2026-11-17
updated: 2026-11-17
sources:
  - cost-efficient-agent-harnesses-park-2026.md
arxiv_id: "2611.12122"
venue: arXiv preprint (cs.AI)
authors:
  - Jongseok Park
  - Jiaxiang Yu
  - Eric Zhou
  - Tony Hong
  - Sunga Kim
  - Yusuf Afifi
  - Ian Wong
  - Ion Stoica
  - Alvin Cheung
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# System Optimizations for Cost-Efficient Agent Harnesses

**Source:** Jongseok Park, Jiaxiang Yu, Eric Zhou, Tony Hong, Sunga Kim, Yusuf Afifi, Ian Wong, [[ion-stoica|Ion Stoica]], Alvin Cheung ([[uc-berkeley|UC Berkeley]]; Northwestern University; [[uiuc|UIUC]]) — [arXiv:2611.12122](https://arxiv.org/abs/2611.12122), cs.AI, v1 9 Nov 2026.

## Summary

The harness barely changes accuracy for current frontier models, but it changes cost a lot. On 100 tasks (Terminal-Bench 2, [[swe-bench|SWE-bench]], SWE-Bench Pro), a bare harness with no system prompt and only a bash tool scores about as well as [[claude-code|Claude Code]] and [[codex-cli|Codex]], while those two often cost about 2× more because of their larger system prompts and tool schemas.

The paper profiles where agent cost goes, using its own benchmark traces and two public trace sets of real Claude Code and Codex use, and finds five opportunities. It then builds **Agency**, a harness interposition platform modelled on Valgrind. Agency wraps an unmodified harness with ptrace and cgroups, captures every LLM call, process spawn and file access, translates them into a common format, and lets user hooks rewrite context or tool calls. Optimizations written once then run on any harness and model.

## Key points

- **Cache reads dominate cost.** Even with [[prefix-caching]], re-reading context is the largest cost item, rising from 13% of call cost under 32K tokens to 73% above 256K. A grep at 100K context costs about as much as 10K new input plus 1K output tokens.
- **Tool calls multiply cost.** At similar context sizes, traces with more tool calls cost 1.6–26× more, because every turn re-reads the whole context.
- **Tool outputs are sparse.** Agents later use only 27% of tool-output tokens, and 36% of outputs are never used.
- **Search eats turns.** Filesystem search takes about 43% of turns and 28% of cost on benchmarks, and 57–89% of turns on the OfficeQA Pro retrieval tasks. 42% of directory listings repeat one an earlier session already made.
- **Independent subtasks bloat context.** Long sessions interleave 1.3–2.7 independent subtasks per dependency depth.
- **Five techniques.** Lossless tool-output compaction (14% smaller, zero cost); [[context-compaction|context garbage collection]] with gradient-boosted trees that fires only when predicted savings exceed cache-rewrite and extra-turn cost (47% per compaction); a subtask stack machine (91% per compaction, rare); SQLite metadata-assisted search (up to 80% fewer search calls); a tool-call batching prompt (up to 30% fewer turns).
- **End to end.** Across Claude Code, Codex and a basic harness with GPT-6 Luna, GPT-6.1 Sol, Sonnet 5.5 and Opus 5.5, cost falls to 0.77 of baseline on average (best 0.65; 0.51 on UUID-named OfficeQA trees) with an accuracy ratio of 0.97. Savings are smallest on Claude Code (0.81) and Opus 5.5 (0.78).
- Agency itself adds no measurable cost: −1.50% per task (95% CI −4.69% to +2.04%).

## Caveats

Benchmark traces are 2–3× shorter than real-world ones, which the authors argue understates the savings. GPT-6 Luna loses some accuracy (0.93) when it misuses the SQLite tables.

## Concepts & entities

- [[agentic-harness-engineering]] · [[context-compaction]] · [[prefix-caching]] · [[kv-cache]] · [[context-engineering]] — cost-side harness design.
- [[subagents]] · [[tool-use]] · [[agentic-search]] — subtask isolation and search turns.
- [[swe-bench]] · [[swe-bench-verified]] · [[terminal-bench]].
- [[claude-code]] · [[codex-cli]] · [[vllm]] · [[ion-stoica]] · [[uc-berkeley]] · [[uiuc]].
- [[same-task-different-work-weinberger-2026]] · [[cache-engineering-for-agents-fu-2026]] — related 2026 work on harness cost and caching.

## References

- [arXiv:2611.12122](https://arxiv.org/abs/2611.12122)
