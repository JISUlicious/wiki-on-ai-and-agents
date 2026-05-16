---
title: "A-MEM: Agentic Memory for LLM Agents"
type: source
created: 2026-05-16
updated: 2026-05-16
sources:
  - a-mem-xu-2025.md
arxiv_id: "2502.12110"
venue: NeurIPS 2025
authors:
  - Wujiang Xu
  - Zujie Liang
  - Kai Mei
  - Hang Gao
  - Juntao Tan
  - Yongfeng Zhang
first_author: Wujiang Xu
year: 2025
introduces:
  - "[[a-mem]]"
  - "[[zettelkasten-memory]]"
  - "[[memory-evolution]]"
tags:
  - 2025
status: complete
importance: high
---

# A-MEM: Agentic Memory for LLM Agents

- **Source**: `sources/a-mem-xu-2025.md`
- **arXiv**: [arXiv:2502.12110](https://arxiv.org/abs/2502.12110)
- **Venue**: NeurIPS 2025
- **Authors**: Wujiang Xu, Zujie Liang, Kai Mei, Hang Gao, Juntao Tan, Yongfeng Zhang (Rutgers University; Independent Researcher; AIOS Foundation)

## Summary

A-MEM applies the **Zettelkasten** note-taking philosophy to LLM-agent memory. Rather than storing interactions as flat, append-only entries, each new memory becomes a richly structured *note* with LLM-generated keywords, tags, contextual descriptions, and an embedding. The system then uses semantic-similarity retrieval over historical memories combined with an LLM-driven analysis step to autonomously decide which existing notes the new memory should be **linked** to — building an interconnected knowledge network rather than a flat log.

The crucial contribution is **memory evolution**: when a new memory is added, the system not only links it to retrieved neighbors but also lets the LLM *rewrite the context, keywords, and tags of those neighbors* in light of the new experience. This breaks the append-only assumption of nearly all prior agent-memory systems (MemGPT, MemoryBank, ReadAgent, Mem0-style graph stores). The memory network continuously refines itself, with notes able to belong simultaneously to multiple Zettelkasten-style "boxes" of related ideas, and higher-order patterns emerging over time without any predefined schema.

A-MEM is positioned as a 2026-era reference design for self-organizing agent memory: empirical evaluation on LoCoMo and DialSim across six foundation models (Llama 3.2 1B/3B, Qwen2.5 1.5B/3B, GPT-4o, GPT-4o-mini) shows consistent gains over MemGPT, MemoryBank, ReadAgent, and LoCoMo baselines, with the largest gains on multi-hop and temporal reasoning where evolving the memory representation matters most.

## Key Points

- **Structured note schema** per memory: $m_i = \{c_i, t_i, K_i, G_i, X_i, e_i, L_i\}$ — original content, timestamp, LLM-generated keywords, tags, contextual description, dense embedding, and link set. Atomic, self-contained, Zettelkasten-style.
- **Link generation**: new note's embedding retrieves top-k semantic neighbors; LLM then judges which connections to actually establish based on shared attributes and conceptual relationships beyond raw similarity.
- **Memory evolution** (the novel contribution): for each retrieved neighbor $m_j$, the LLM is prompted with the new memory, the rest of the neighborhood, and $m_j$ itself, and decides whether to rewrite $m_j$'s context, keywords, and tags. The evolved $m_j^*$ replaces the original in the store. This makes the memory network *non-append-only*.
- **"Boxes" of related memories**: linked notes form Zettelkasten-style boxes, but unlike rigid graph schemas, a single memory can belong to multiple boxes simultaneously. Retrieving one memory auto-surfaces linked memories in the same box.
- **No predefined schema**: contrasted explicitly with Mem0-style graph-database memory whose fixed schemas limit adaptability. A-MEM's structure emerges from content.
- **Distinguished from agentic RAG**: agentic RAG exercises agency at *retrieval* time over a static knowledge base; A-MEM exercises agency at *storage and evolution* time over a self-modifying memory.
- **Empirical results**: on LoCoMo (~9K-token dialogues, up to 35 sessions, 7,512 QA pairs across multi-hop / temporal / open-domain / single-hop / adversarial), A-MEM consistently beats LoCoMo, ReadAgent, MemoryBank, and MemGPT across six foundation models. Strongest gains on multi-hop and temporal reasoning; on GPT-4o-mini multi-hop, F1 jumps from 9.65 (no linking, no evolution) → 21.35 (linking only) → 27.02 (full A-MEM).
- **Ablation confirms memory evolution matters**: removing the memory-evolution module drops multi-hop F1 from 27.02 to 21.35 and temporal F1 from 45.85 to 31.24 on GPT-4o-mini.
- **Implementation**: text encoder is `all-MiniLM-L6-v2`; default top-k = 10; Qwen/Llama served via Ollama with LiteLLM for structured output; GPT models use the native structured output API.

## Entities Mentioned

- [[rutgers-university]] (lead author affiliation)
- [[memgpt]] (baseline)
- [[memorybank]] (baseline)
- [[mem0]] (baseline class — graph-database memory)
- [[llama-3]] (eval foundation model)
- [[qwen-2-5]] (eval foundation model)
- [[gpt-4o]] (eval foundation model)
- [[claude-3-5-haiku]] (Appendix eval)

## Concepts Discussed

- [[a-mem]]
- [[zettelkasten-memory]]
- [[memory-evolution]]
- [[memory-management]]
- [[llm-agent]]
- [[retrieval-augmented-generation]]
- [[agentic-rag]]
- [[long-term-memory]]
- [[embedding-retrieval]]

## Notable Quotes

> "When a new memory is added, we generate a comprehensive note containing multiple structured attributes, including contextual descriptions, keywords, and tags. The system then analyzes historical memories to identify relevant connections, establishing links where meaningful similarities exist. Additionally, this process enables memory evolution — as new memories are integrated, they can trigger updates to the contextual representations and attributes of existing historical memories, allowing the memory network to continuously refine its understanding."

> "While agentic RAG approaches demonstrate agency in the retrieval phase by autonomously deciding when and what to retrieve, our agentic memory system exhibits agency at a more fundamental level through the autonomous evolution of its memory structure."

## References

_Original source: `sources/a-mem-xu-2025.md`_
