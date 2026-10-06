---
title: Lilian Weng
type: entity
created: 2026-10-06
updated: 2026-10-07
sources:
  - llm-powered-autonomous-agents-weng-2023.md
  - prompt-engineering-weng-2023.md
  - harness-engineering-self-improvement-weng-2026.md
  - scaling-laws-carefully-weng-2026.md
  - lilianweng-archives.md
affiliations:
  - org: "[[openai]]"
    role: research and applied AI / safety (per her 2023 posts, "my team at OpenAI")
status: complete
importance: medium
tags:
  - person
---

# Lilian Weng

ML researcher and author of **Lil'Log** (lilianweng.github.io): "documenting my learning notes in this blog since 2017". It holds 50+ long, carefully referenced survey posts. **This wiki treats Lil'Log as a trusted source.**

**Affiliation:** her October 2023 post refers to "my team at OpenAI" working on safe default behaviour via RLHF. Later affiliations are not covered by any ingested source. Her 2026 posts give no affiliation, and one credits John Schulman with feedback.

## Posts ingested in this wiki

| Date | Post | Role in the wiki |
|---|---|---|
| 2023-03-15 | [[prompt-engineering-weng-2023\|Prompt Engineering]] | Practitioner survey; defines it as "in-context prompting" whose goal is "alignment and model steerability" |
| **2023-06-23** | [[llm-powered-autonomous-agents-weng-2023\|LLM Powered Autonomous Agents]] | **The canonical 2023 agent model:** LLM as "brain" + planning + memory + tool use; three challenges |
| 2026-06-24 | [[scaling-laws-carefully-weng-2026\|Scaling Laws, Carefully]] | Reconciles Kaplan and Chinchilla; data-constrained laws; how fragile the fits are |
| **2026-07-04** | [[harness-engineering-self-improvement-weng-2026\|Harness Engineering for Self-Improvement]] | **The 2026 harness model:** a harness definition; design patterns; harness optimization as the path to recursive self-improvement; "loop engineering" |

**From 2023 to 2026, her framing moved** from "agent = LLM + memory + tools + planning + action" to harnesses that also include workflow and loop design, evaluation, permission controls and persistent state. That is the same move this wiki records, from [[llm-agent]] to [[agentic-harness-engineering]].

## Other notable posts (not yet ingested)

From the [Lil'Log archive](https://lilianweng.github.io/archives/):
- *Why We Think* (2025-05-01): test-time compute and chain of thought.
- *Reward Hacking in Reinforcement Learning* (2024-11-28).
- *Extrinsic Hallucinations in LLMs* (2024-07-07).
- *Thinking about High-Quality Human Data* (2024-02-05).
- *Adversarial Attacks on LLMs* (2023-10-25).
- *The Transformer Family Version 2.0* (2023-01-27).
- *Large Transformer Model Inference Optimization* (2023-01-10).
- *What are Diffusion Models?* (2021-07-11).
- *How to Build an Open-Domain Question Answering System?* (2020-10-29).

These are candidates for future ingest; *Why We Think* and *Reward Hacking* bear most directly on [[test-time-compute]] and [[verifier]].

## References

- [[llm-powered-autonomous-agents-weng-2023]]
- [[prompt-engineering-weng-2023]]
- [[harness-engineering-self-improvement-weng-2026]]
- [[scaling-laws-carefully-weng-2026]]
