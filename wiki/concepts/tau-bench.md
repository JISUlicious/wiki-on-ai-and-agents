---
title: τ-Bench
type: concept
created: 2026-09-07
updated: 2026-09-30
sources:
  - recuris-experiential-working-memory-evolution-yu-2026.md
  - skill-state-scalable-long-horizon-agent-skills-badhe-2026.md
status: draft
importance: medium
tags: []
---

# τ-Bench

An agent benchmark from Sierra that evaluates **tool-use agents in multi-turn conversations with a simulated user**, against a domain policy. The agent must gather information across turns, obey rules it was given rather than rules it inferred, and — critically — **write** to a database. Retail and Airline are the canonical domains; **τ²-Bench** is the successor, and the two are not interchangeable when comparing numbers.

What distinguishes it from single-shot tool benchmarks is that **success is gated on specific state changes**, not on plausible output. An agent that says the right thing and fails to commit the write scores zero. That property is why τ-Bench separates systems that other benchmarks rank as equivalent, and why routing precision costs so much more here than elsewhere — see the disagreement recorded on [[skill-routing]].

## Results in this wiki

| System | Result | Note |
|---|---|---|
| [[recuris-experiential-working-memory-evolution-yu-2026\|Recuris]] | **+17.76** on GPT-5.6 Sol; Claude Opus 5 to **87.94%** | τ²-**Retail** specifically; on τ²-Airline the same package gives Opus 5 only +1.0 against an 89.5 baseline |
| [[skill-state-scalable-long-horizon-agent-skills-badhe-2026\|SKILL.state]] | used as one of four evaluation settings | |

> [!warning] Check which variant and which domain
> Numbers quoted as "τ-bench" in summaries are frequently **τ²-Bench**, and frequently **Retail only**. Retail and Airline behave differently enough that a package can be worth +17.8 on one and +1.0 on the other, as Recuris is. Airline baselines are already high, so headroom differs too.

## Related

- [[agent-benchmark]] · [[agent-evaluation]] · [[tool-use]] · [[function-calling]]
- [[skill-routing]] — τ-Bench is the setting where bad routing is expensive, because success depends on a specific write.
- [[merchantbench-long-term-coherence-shi-2026]] · [[e-commerce-bench-autonomous-business-operation-fan-2026]] — the long-horizon commerce benchmarks, where τ-Bench's single-session scope ends.
- [[swe-bench]] · [[terminal-bench]] · [[longbench]] — other benchmark pages.

## References

- [[recuris-experiential-working-memory-evolution-yu-2026]] · [[skill-state-scalable-long-horizon-agent-skills-badhe-2026]]

## Related 2026 sources

- [[gauge-when-not-to-trust-llm-as-a-judge-bodhwani-2026]] — Amazon audit of simulator+judge release gates: ranks agents well, but satisfaction says nothing about task success
- [[xyeval-agents-say-yes-to-bad-advice-wu-2026]] — DeepMind: a plausible wrong user hint cuts agent scores up to 46.7%; warning prompts only partly help
- [[salesforce-koa-enterprise-agentic-tool-use-salesforce-2026]] — Salesforce Koa: GRPO on environments built from agent configs; small gains, and its SFT checkpoint beats the RL release on 2 of 3
