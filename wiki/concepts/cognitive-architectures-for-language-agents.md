---
title: Cognitive Architectures for Language Agents (CoALA)
type: concept
created: 2026-09-29
updated: 2026-09-29
sources:
  - mirix-wang-2025.md
  - procedural-graphs-self-evolving-execution-structures-lu-2026.md
status: draft
importance: medium
---

# Cognitive Architectures for Language Agents (CoALA)

**CoALA** is a conceptual framework for describing [[llm-agent|language agents]], proposed by Sumers, [[shunyu-yao|Yao]], [[karthik-narasimhan|Narasimhan]] and Griffiths (arXiv 2023; TMLR). It draws on cognitive science and symbolic AI and describes an agent through three parts: **modular memory components**, a **structured action space** covering actions on internal memory and on external environments, and a **generalized decision-making process** that chooses among those actions. The authors use it to organise existing agents looking back and to suggest research directions looking forward. CoALA is a taxonomy, not a system. The wiki cites it as the conceptual source that later memory systems put into practice.

## In this wiki

- **The typology that MIRIX builds.** [[mirix]] ([[mirix-wang-2025]]) is described as operationalizing the cognitive-architecture memory typology CoALA proposed, using six typed stores (Core, Episodic, Semantic, Procedural, Resource, Knowledge Vault), each with its own manager agent.
- **Typed stores over flat stores.** The [[memory-systems-anatomy-2026]] query traces the "typed compositional stores" trend in [[mirix]], [[mem-alpha]] and [[chronos]] to this lineage. It calls those systems CoALA "finally getting empirical instantiations", and it uses the lineage for its classification of memory by cognitive role: [[episodic-memory-llm|episodic]], [[semantic-memory|semantic]], [[procedural-memory|procedural]] and working memory.
- **Claiming a slot.** [[procedural-graphs-self-evolving-execution-structures-lu-2026|Procedural Graphs]] positions itself in CoALA's procedural-memory slot, storing procedures as editable graph transitions rather than code skills (compare [[voyager]]'s [[skill-library]]).
- [[memory-management]] lists MIRIX as the CoALA-typology entry in its taxonomy of agent memory systems.

## Related

- [[llm-agent]] · [[memory-management]] · [[long-term-memory]] · [[multi-agent-memory]]
- [[semantic-memory]] · [[episodic-memory-llm]] · [[procedural-memory]]
- [[mirix]] · [[mem-alpha]] · [[react]] · [[global-workspace-theory]]
- [[shunyu-yao]] · [[karthik-narasimhan]] · [[princeton-university]]

## References

- Sumers, Yao, Narasimhan, Griffiths, "Cognitive Architectures for Language Agents" — [arXiv:2309.02427](https://arxiv.org/abs/2309.02427)
- [[mirix-wang-2025]] · [[procedural-graphs-self-evolving-execution-structures-lu-2026]] · [[memory-systems-anatomy-2026]]
