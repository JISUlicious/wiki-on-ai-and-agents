---
title: "Generative Agents: Interactive Simulacra of Human Behavior"
type: source
created: 2026-05-16
updated: 2026-05-16
sources:
  - generative-agents-park-2023.md
arxiv_id: "2304.03442"
venue: UIST 2023
authors:
  - "[[joon-sung-park]]"
  - Joseph C. O'Brien
  - Carrie J. Cai
  - Meredith Ringel Morris
  - "[[percy-liang]]"
  - Michael S. Bernstein
first_author: "[[joon-sung-park]]"
year: 2023
introduces:
  - "[[memory-stream]]"
  - "[[reflection-mechanism]]"
  - "[[generative-agents]]"
tags:
  - 2023
status: complete
importance: high
---

# Generative Agents: Interactive Simulacra of Human Behavior

- **Source:** `sources/generative-agents-park-2023.md`
- **arXiv:** [arXiv:2304.03442](https://arxiv.org/abs/2304.03442)
- **Venue:** UIST 2023
- **Authors:** Joon Sung Park, Joseph C. O'Brien, Carrie J. Cai, Meredith Ringel Morris, Percy Liang, Michael S. Bernstein
- **Affiliations:** Stanford University; Google Research; Google DeepMind

## Summary

Park et al. introduce **generative agents**: LLM-backed software agents that simulate believable human behavior over long time horizons by maintaining a persistent natural-language memory and reasoning about it. The paper's core contribution is an **agent architecture** that extends a large language model (ChatGPT, in their instantiation) with three components — a memory stream, a reflection mechanism, and a hierarchical planner — designed to solve the long-context, long-horizon coherence problem that arises when an agent must act consistently with weeks of accumulated experience.

The **memory stream** is an append-only, timestamped log of natural-language observations recording everything the agent perceives or does. Because the full stream exceeds any feasible prompt budget, the architecture uses a **retrieval function** that scores each memory by a weighted sum of three normalized signals: `score = α_recency · recency + α_importance · importance + α_relevance · relevance`. Recency is an exponential decay over time since last access; importance is an LLM-scored integer (1–10) assigned when each memory is first stored ("brushing teeth" ≈ 1, "asking your crush out" ≈ 8); relevance is the cosine similarity between the memory's embedding and an embedding of the current query. All three α values are set to 1 in their implementation, and the top-ranked memories that fit in the context window are spliced into the prompt.

The **reflection mechanism** synthesizes higher-level memories from lower-level ones. Periodically (when summed importance of recent memories crosses a threshold), the agent generates salient questions about its recent experience, retrieves memories relevant to each, and prompts the LLM to produce abstract inferences ("Klaus is passionate about research"). Reflections are written back into the memory stream and can themselves be reflected on, yielding **reflection trees** with observations as leaves and progressively more abstract conclusions as inner nodes. Together with a top-down **planning hierarchy** (daily plan → hour blocks → 5–15-minute actions, with re-planning on perceived disruption), this gives the agent both long-term coherence and moment-to-moment responsiveness.

The architecture is instantiated in **Smallville**, a Sims-like sandbox with 25 agents. The authors report emergent social behavior (information diffusion, relationship formation, party coordination from a single seed prompt) and run a controlled "interview" evaluation plus an end-to-end evaluation. **Ablations show all three components — observation memory, reflection, and planning — contribute critically**; removing any one degrades believability. This paper is the canonical reference for the recency × importance × relevance retrieval triad and for reflection as an LLM-driven memory consolidation primitive, and it has become a touchstone for downstream work on long-term LLM agent memory.

## Key Points

- **Memory stream**: append-only natural-language log of observations, timestamped; the substrate for everything else.
- **Retrieval scoring**: `score = α_recency · recency + α_importance · importance + α_relevance · relevance`, with min-max normalization to [0,1]. All αs = 1 in the paper.
  - **Recency** = exponential decay since last access.
  - **Importance** = LLM-rated 1–10 score assigned at write time (mundane vs. core memories).
  - **Relevance** = cosine similarity between memory embedding and query embedding.
- **Reflection**: triggered when summed importance of recent observations exceeds a threshold; the agent generates salient questions, retrieves memories per question, and asks the LLM to produce abstract inferences that are stored back into the stream.
- **Reflection trees**: reflections can recursively reflect on other reflections, forming a tree of progressively higher-level abstractions over raw observations.
- **Planning hierarchy**: top-down daily plan → hourly blocks → 5–15-minute actions; plans live in the memory stream and are revised when observations conflict with them.
- **Smallville**: 25-agent sandbox demonstration; emergent behaviors include Valentine's Day party invitations spreading from a single seed prompt over two simulated days.
- **Ablation finding**: removing memory, reflection, or planning each significantly hurts interview-evaluation believability — the three are complementary, not redundant.
- **Common failure modes**: failure to retrieve the relevant memory, fabricated embellishments, and overly formal LM speech leaking into agent dialogue.

## Entities Mentioned

- [[joon-sung-park]] — first author; Stanford PhD student leading the work.
- [[percy-liang]] — co-author; Stanford NLP/foundation-models advisor.
- [[stanford-university]] — host institution for four of the six authors.
- [[google-research]] — affiliation of Carrie J. Cai (Meredith Ringel Morris is at Google DeepMind).

## Concepts Discussed

- [[generative-agents]] — the paper's central construct: LLM-driven agents with persistent memory that simulate believable human behavior.
- [[memory-stream]] — append-only NL observation log that serves as the agent's long-term memory substrate.
- [[reflection-mechanism]] — periodic LLM synthesis of higher-level inferences over recent memories; the paper's signature memory-consolidation primitive.
- [[memory-management]] — broader family of techniques for handling LLM agent memory across context-window limits; this paper is a foundational reference.
- [[llm-agent]] — the general paradigm of LLM-backed autonomous agents that this paper helped concretize for long-horizon settings.

## Notable Quotes

> "Our architecture comprises three main components. The first is the memory stream, a long-term memory module that records, in natural language, a comprehensive list of the agent's experiences. A memory retrieval model combines relevance, recency, and importance to surface the records needed to inform the agent's moment-to-moment behavior."

> "We demonstrate through ablation that the components of our agent architecture—observation, planning, and reflection—each contribute critically to the believability of agent behavior."

## References

_Original source: `sources/generative-agents-park-2023.md`_
