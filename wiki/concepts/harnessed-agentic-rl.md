---
title: Harnessed Agentic RL
type: concept
created: 2026-09-29
updated: 2026-09-29
sources:
  - agent-lightning-v1-harnessed-agentic-rl-he-2026.md
  - clawgym-ii-black-box-rl-agent-harness-song-2026.md
  - lego-rl-harness-native-reinforcement-learning-coding-agents-du-2026.md
status: draft
importance: medium
---

# Harnessed Agentic RL

**Harnessed agentic RL** is RL post-training run *through the same agent harness used at deployment*, such as mini-SWE-agent, [[openhands|OpenHands]], [[opencode|OpenCode]], [[claude-code|Claude Code]] or [[openclaw|OpenClaw]]. The harness's model endpoint is pointed at a proxy that the trainer owns. The harness keeps control of context construction, tool execution, subagents and compaction. The trainer sees only the request-response pairs that cross the model boundary, and it updates only the weights. [[agent-lightning-v1-harnessed-agentic-rl-he-2026|Agent Lightning v1.0]] coined the name. Its central claim is that this is not traditional [[agentic-rl]] with an extra network hop, because **one rollout no longer maps to one training sample**.

## In this wiki

A cluster of 2026 frameworks fits this paradigm. They agree on the proxy and differ in how they rebuild training samples:

| Framework | Harness stance | Sample construction |
|---|---|---|
| [[agent-lightning-v1-harnessed-agentic-rl-he-2026\|Agent Lightning v1.0]] | open, instrumentable | best-effort linear merge; mean 2.41 samples per rollout |
| [[clawgym-ii-black-box-rl-agent-harness-song-2026\|ClawGym II]] | production binary treated as a black box | rollout-level prefix tree |
| [[lego-rl-harness-native-reinforcement-learning-coding-agents-du-2026\|LEGO-RL]] | unmodified harnesses with thin adapters | in-process proxy records token IDs and log-probs |
| [[molt-pytorch-native-agentic-rl-framework-hu-2026\|Molt]] | token-in/token-out loopback | never trains on a token it did not generate |

Agent Lightning names four problems that the paradigm creates: retokenization drift (see [[tool-call-parser]] and [[toktier-exact-stateful-tokenization-zhang-2026]]), advantage calculation when the number of samples varies, loss normalization, and backend scheduling. It and ClawGym II agree on a rollout-level [[grpo|GRPO]] baseline, and both defer credit assignment *within* a rollout. [[reward-hacking]] shows up in concrete form: on SWE-smith, agents fetched the gold patch through git history or the network.

It is the mirror image of [[agentic-harness-engineering]]. That approach freezes the model and evolves the harness, while this one freezes the harness and moves the weights. [[envharness-awakening-static-worlds-huang-2026|EnvHarness]] is the environment-side complement. ClawGym II's same-weights, two-harness results feed into [[harness-vs-model-attribution]].

## Related

- [[agentic-rl]] · [[reinforcement-learning]] · [[grpo]] · [[ppo]] · [[outcome-based-reward]] · [[reward-hacking]]
- [[agentic-harness-engineering]] · [[code-as-harness]] · [[harness-vs-model-attribution]] · [[swe-smith]] · [[swe-bench]]

## References

- [[agent-lightning-v1-harnessed-agentic-rl-he-2026]] · [[clawgym-ii-black-box-rl-agent-harness-song-2026]]
- [[lego-rl-harness-native-reinforcement-learning-coding-agents-du-2026]] · [[molt-pytorch-native-agentic-rl-framework-hu-2026]]
