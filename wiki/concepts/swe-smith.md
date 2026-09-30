---
title: SWE-smith
type: concept
created: 2026-09-29
updated: 2026-09-29
sources:
  - agent-lightning-v1-harnessed-agentic-rl-he-2026.md
  - swe-bench-live-zhang-2025.md
  - envharness-awakening-static-worlds-huang-2026.md
  - toktier-exact-stateful-tokenization-zhang-2026.md
status: draft
importance: low
---

# SWE-smith

**SWE-smith** (Yang et al., 2025) is a pipeline that generates **training data** for software-engineering agents. Given any Python codebase, it builds an execution environment and then synthesizes hundreds to thousands of task instances that **break existing tests**. The agent's task is to repair them. The released dataset has 50K instances from 128 GitHub repositories, an order of magnitude more than earlier datasets such as [[swe-gym]]. Its authors include [[carlos-jimenez]] and [[ofir-press]] of [[swe-bench]]. It is a training corpus, not an evaluation.

## In this wiki

- **RL training tasks.** [[agent-lightning-v1-harnessed-agentic-rl-he-2026|Agent Lightning v1]] trains its coding agent on SWE-smith tasks inside a mini-SWE-agent harness with GRPO. Qwen3.5-9B goes from 41.8% to 56.4% on [[swe-bench-verified|SWE-bench Verified]].
- **Environment-generation baseline.** [[envharness-awakening-static-worlds-huang-2026|EnvHarness]] compares against SWE-smith as a domain-specific environment generator. The page cautions that SWE-smith was built to run at scale, not at 100–300 seeds.
- **Trajectory corpus.** [[toktier-exact-stateful-tokenization-zhang-2026|TokTier]] replays 74,064 repairs from public SWE-smith trajectories to test its tokenizer splicing.
- **Benchmark landscape.** [[swe-bench-live-zhang-2025|SWE-bench-Live]] lists it alongside Multi-SWE-bench and [[livecodebench]].

## Related

- [[swe-bench]] · [[swe-gym]] · [[swe-bench-live]] · [[swe-agent]] · [[harnessed-agentic-rl]]

## References

- Yang et al., "SWE-smith: Scaling Data for Software Engineering Agents" (2025), [arXiv:2504.21798](https://arxiv.org/abs/2504.21798)
- [[agent-lightning-v1-harnessed-agentic-rl-he-2026]] · [[envharness-awakening-static-worlds-huang-2026]]
