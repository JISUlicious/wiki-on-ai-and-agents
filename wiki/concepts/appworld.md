---
title: AppWorld
type: concept
created: 2026-09-29
updated: 2026-09-29
sources:
  - sage-skill-library-wang-2025.md
  - agentic-context-engineering-zhang-2025.md
  - agent-skills-survey-xu-2026.md
status: draft
importance: medium
---

# AppWorld

**AppWorld** (Trivedi et al., 2024) is a benchmark for interactive coding agents that carry out everyday digital tasks across many apps. Its engine simulates 9 everyday apps with 457 APIs, populated with the activity of about 100 fictitious users. The benchmark has 750 tasks that require the agent to write code with real control flow against those APIs, over several steps, rather than make a short fixed sequence of tool calls. It sits in the wiki's [[agent-benchmark]] family next to [[gaia]] and OSWorld.

## In this wiki

- **Skill libraries trained with RL.** [[sage-skill-library-wang-2025|SAGE]] trains Qwen2.5-32B with [[grpo|GRPO]] plus a skill library. It uses AppWorld's grouping of tasks into scenarios of three similar tasks, which suits its Sequential Rollout. It reports **+8.9 Scenario Goal Completion** (60.7 vs 51.8) with fewer steps and tokens. The [[agent-skills-survey-xu-2026|skills survey]] and [[skill-optimization]] repeat this result.
- **Context engineering.** [[agentic-context-engineering-zhang-2025|ACE]] reports its headline result on the AppWorld leaderboard: DeepSeek-V3.1 plus ACE matches the GPT-4.1-powered top entry, IBM-CUGA. ACE also uses AppWorld to demonstrate *context collapse*.

## Related

- [[agent-benchmark]] · [[tool-use]] · [[skill-library]] · [[agentic-context-engineering]] · [[llm-agent]]

## References

- Trivedi et al., "AppWorld: A Controllable World of Apps and People for Benchmarking Interactive Coding Agents" (2024), [arXiv:2407.18901](https://arxiv.org/abs/2407.18901)
- [[sage-skill-library-wang-2025]] · [[agentic-context-engineering-zhang-2025]]
