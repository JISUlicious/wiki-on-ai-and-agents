---
title: Claude 3.5 Sonnet
type: entity
created: 2026-09-29
updated: 2026-09-29
sources:
  - darwin-godel-machine-zhang-2025.md
  - sage-skill-library-wang-2025.md
  - openhands-wang-2024.md
  - swe-gym-pan-2024.md
family: claude
status: draft
importance: medium
tags:
  - model
---

# Claude 3.5 Sonnet

Mid-tier model of [[anthropic|Anthropic]]'s [[claude|Claude]] 3.5 generation, announced in June 2024 and updated in October 2024. Sources here cite the updated version as "Claude 3.5 Sonnet V2". In this wiki's 2024–25 sources it is the **default strong closed model for coding agents**, used both as a frozen agent backbone and as the expert that generates trajectories for training open models.

## In this wiki

- **Self-improving agent backbone.** In [[darwin-godel-machine-zhang-2025|Darwin Gödel Machine]], a minimal Claude-3.5-Sonnet agent with Bash and edit tools goes from **20.0% to 50.0% on SWE-bench Verified** and 14.2% to 30.7% on Polyglot over 80 iterations of self-rewriting. The discovered scaffolds transfer to Claude 3.7 Sonnet. See [[darwin-godel-machine]].
- **Expert-trajectory source.** [[sage-skill-library-wang-2025|SAGE]] warm-starts [[qwen-2-5|Qwen2.5]]-32B by SFT on Claude 3.5 Sonnet V2 trajectories, because open models could not follow the skill-library prompt. [[swe-gym-pan-2024|SWE-Gym]] rejection-samples GPT-4o and Claude 3.5 Sonnet rollouts to fine-tune Qwen-2.5-Coder-32B.
- **Harness evaluation.** [[openhands-wang-2024|OpenHands]] CodeActAgent v1.8 with `claude-3-5-sonnet` reaches **26.7% on SWE-Bench Lite**. [[on-the-fragility-of-self-improving-agents-ye-2026|On the Fragility of Self-Improving Agents]] re-examines AWM results run on Claude-3.5-Sonnet.

## Lineage

In the [[claude]] family it followed Claude 3 Sonnet and was followed by Claude 3.7 Sonnet (Feb 2025), which several of the sources above use as the next backbone. See the model table in [[claude]]; later generations run to [[claude-opus-4-8]].

## Related

- [[claude]] · [[anthropic]] · [[swe-bench]] · [[swe-agent]] · [[openhands]] · [[self-improving-agent]] · [[knowledge-distillation]]

## References

- [Anthropic, "Claude 3.5 Sonnet" announcement](https://www.anthropic.com/news/claude-3-5-sonnet) — June 2024 release
- [[darwin-godel-machine-zhang-2025]] · [[sage-skill-library-wang-2025]] · [[openhands-wang-2024]] · [[swe-gym-pan-2024]]
