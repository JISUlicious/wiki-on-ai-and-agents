---
title: "You Don't Need Long Context"
type: source
created: 2026-11-10
updated: 2026-11-10
sources:
  - you-dont-need-long-context-belcak-2026.md
tags:
  - 2026
  - paper
status: complete
importance: high
authors:
  - Peter Belcak
  - Saurav Muralidharan
  - Jindong Jiang
  - Yonggan Fu
  - Wonmin Byeon
  - Hanrong Ye
  - Ryan Angilly
  - Ali Taghibakhshi
  - Shih-Yang Liu
  - Siru Ouyang
  - Greg Heinrich
  - Pavlo Molchanov
year: 2026
venue: arXiv preprint
arxiv_id: "2611.05306"
---

# You Don't Need Long Context

**Source:** Peter Belcak, Pavlo Molchanov et al. ([[nvidia|NVIDIA]]) — [arXiv:2611.05306](https://arxiv.org/abs/2611.05306), cs.AI, v1 3 Nov 2026. Preprint. Featured in the NLP Newsletter as "Context Compaction".

## Summary

A position-plus-evidence paper arguing that agents can keep their live context short with little or no loss in accuracy. The cost motivation: decode cost grows with context depth, so serving at a 256K window instead of 16K costs **3.6× (GLM 5.2) to 7× (Qwen 3 Coder Next)** more per generated token on H100s, and on a [[dgx-spark|DGX Spark]] later tokens take about 2× longer. The authors split "long-context" workloads into long-horizon tasks (history builds up) and long-input tasks (big inputs), note that the second can always be turned into the first by putting the input in a file and reading it selectively, and test three questions with minimal ReAct/Bash harnesses (mini-swe-agent and a generalised "mini-agent") on Claude Opus 4.5 and GPT 5.6 Sol.

The answer to all three is yes. The headline experiment is **single-action compaction (SAC)**: each fresh conversation gets the task plus a compact natural-language state, takes exactly one action, updates the state, and is thrown away. On [[swe-bench-verified|SWE-bench Verified]] this keeps Opus 4.5 at baseline (70.2 vs 70.0) with a median peak context of about 8K tokens.

## Key points

- **Three regimes.** Full-History Accumulation (FHA, the baseline); End-of-Conversation Compaction (EoCC: compact when a threshold such as 24K/18K is crossed, restart with task + summary); SAC (compact after every action).
- **RQ1, long-horizon.** EoCC at 48K down to 16K limits usually matches or slightly beats FHA on SWE-bench Verified, SWE-bench Multilingual, [[terminal-bench|Terminal-Bench 2.1]], FRAMES, BFCLv3 and τ²-bench. Best EoCC: Opus 4.5 74.8 vs 70.0 on SWE-bench Verified; GPT 5.6 Sol 88.8 vs 84.4. FRAMES is the weak spot (−1.3 to −3.7 pp for Opus).
- **RQ2, long-input.** Selective reading via Bash tools, with compaction as fallback, matches or beats full-input baselines on [[longbench|LongBench v2]], LOONG, AA-LCR and four needle-style sets, sometimes by about 10 pp.
- **RQ3, single action.** SAC: Opus 4.5 70.2 (SWE-V) and 51.8 (TB 2.1, +4.6 over baseline); GPT 5.6 Sol 82.4 and 84.3 (−2.0 and −3.3). Maximum peak context falls from 130–200K to 15–37K.
- **Semantic State Space Models (SSSMs).** Proposed models post-trained to read a natural-language state, take one action, and write an updated state that another instance can continue from. Described as a [[state-space-model]] whose state is readable text rather than hidden activations. The authors argue such models would be cheaper to serve and easier to train. This is a proposal only; no SSSM is trained.
- **Stated limits.** Compaction is lossy; you can construct tasks (such as recalling the i-th of many coin tosses) that must fail without a log of the history. The authors' fix is to keep the full log outside the context and read it on demand, which turns the problem back into a long-input task.

## Caveats

Only two closed models drive the accuracy experiments, and the harnesses are deliberately minimal. Results show that long context is *not necessary* in the settings tested. They do not show it never helps; the authors say as much. The cost figures cover serving cost (open-weight models on rented H100s), not retail API prices, which are flat per token.

## Concepts & entities

- [[context-compaction]] · [[context-engineering]] · [[context-rot]] · [[long-context-llm]] — the core thesis.
- [[compaction-cliff-long-running-agent-memory-zerhoudi-2026]] · [[lost-in-compaction-side-constraint-loss-wang-2026]] — counterpoints on what compaction loses.
- [[coding-agents-effective-long-context-processors-cao-2026]] · [[context-as-an-environment-scroll-lin-2026]] — the "store input, read selectively" pattern.
- [[react]] · [[swe-agent]] · [[swe-bench-verified]] · [[terminal-bench]] · [[tau-bench]] · [[longbench]] — harness and benchmarks.
- [[state-space-model]] · [[kv-cache]] — SSSM analogy and the cost mechanism.
- [[nvidia]] · [[dgx-spark]] · [[glm-5-2]].

## References

- [arXiv:2611.05306](https://arxiv.org/abs/2611.05306)
