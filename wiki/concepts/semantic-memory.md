---
title: Semantic Memory
type: concept
created: 2026-09-29
updated: 2026-09-29
sources:
  - mirix-wang-2025.md
  - mem-alpha-wang-2025.md
  - code-as-agent-harness-ning-2026.md
status: draft
importance: medium
---

# Semantic Memory

**Semantic memory** is memory of general facts and concepts, cut loose from when and where they were learned. The term comes from cognitive science: Tulving (1972) set it against **episodic memory**, which is autobiographical events tied to a time and place. In LLM agents, semantic memory is the store of durable facts about the user, the domain and the world, such as "Postgres uses MVCC". It sits beside [[episodic-memory-llm|episodic memory]] (what happened) and [[procedural-memory]] (how to do things). It is also distinct from the model's parametric knowledge, because it is an external, editable store that the agent writes to.

## In this wiki

- **A typed store in MIRIX.** [[mirix]] ([[mirix-wang-2025]]) makes Semantic Memory one of six typed stores. It holds abstract concepts and entities with fields `name / summary / details / source`, arranged as a tree (for example Social Network → Favorites → Sports), and has its own Memory Manager agent.
- **Atomic facts in Mem-α.** [[mem-alpha]] ([[mem-alpha-wang-2025]]) uses a three-part memory of core, semantic and episodic stores, where semantic memory holds *atomic factual statements* that support insert, update and delete. Its RL policy learns which component each new piece of information belongs in. Structured memory (0.592 on MemoryAgentBench) beats flat single-paragraph memories (0.071 and 0.198).
- **Repository memory for coding agents.** The [[code-as-agent-harness-ning-2026]] survey (§3.2) lists "semantic/repo memory" (AutoCodeRover, RepoCoder, CodeRAG) as one of six functional memory roles.
- **A shared taxonomy.** The [[memory-systems-anatomy-2026]] query places semantic memory in the classification by cognitive role that runs from [[cognitive-architectures-for-language-agents|CoALA]] through MIRIX and Mem-α. It cites [[hipporag-2]] and [[mem0]] as semantic-style stores. [[llm-agent]] describes a plain vector store as the simplest form of semantic memory.

## Related

- [[episodic-memory-llm]] · [[procedural-memory]] · [[long-term-memory]] · [[memory-management]] · [[memory-consolidation]]
- [[cognitive-architectures-for-language-agents]] · [[mirix]] · [[mem-alpha]] · [[mem0]] · [[hipporag-2]] · [[knowledge-graph]] · [[graph-memory]]

## References

- [[mirix-wang-2025]] · [[mem-alpha-wang-2025]] · [[code-as-agent-harness-ning-2026]] · [[memory-systems-anatomy-2026]]
