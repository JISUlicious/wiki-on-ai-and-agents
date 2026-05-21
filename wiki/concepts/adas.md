---
title: ADAS (Automated Design of Agentic Systems)
type: concept
created: 2026-05-21
updated: 2026-05-21
sources: []
status: draft
importance: medium
tags: []
---

# ADAS — Automated Design of Agentic Systems

ADAS (Hu, Lu, Clune 2024) frames agent construction itself as an open-ended search problem: a "meta agent" proposes new agent designs as code, evaluates them on tasks, and iterates. It treats the space of agent architectures (prompts, control flow, tool use, multi-agent topologies) as a programmable search space rather than a fixed template, and is a direct precursor to fully self-modifying systems like [[darwin-godel-machine]].

- Search is performed over executable code that defines an agent.
- A meta-agent reads prior designs + benchmark scores and proposes the next candidate.
- Demonstrates open-ended improvement on reasoning, coding, and tool-use benchmarks.
- Bridges the gap between hand-designed agent scaffolds and the [[self-improving-agent]] program.

## Related

- [[darwin-godel-machine]] — successor; self-modifying code agent
- [[self-improving-agent]]
- [[jeff-clune]] — co-author; open-endedness agenda
- [[sakana-ai]] — adjacent open-ended ML research

## References

- [[darwin-godel-machine]]
- [[self-improving-agent]]
