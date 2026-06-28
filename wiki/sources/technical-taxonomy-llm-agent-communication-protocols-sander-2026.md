---
title: "A Technical Taxonomy of LLM Agent Communication Protocols"
type: source
created: 2026-06-29
updated: 2026-06-29
sources:
  - technical-taxonomy-llm-agent-communication-protocols-sander-2026.md
arxiv_id: "2606.19135"
authors:
  - Linus Sander
  - Habtom Kahsay Gidey
  - Alexander Lenz
  - Alois Knoll
first_author: Linus Sander
year: 2026
venue: arXiv preprint
tags: [2026, survey]
status: complete
importance: medium
---

# A Technical Taxonomy of LLM Agent Communication Protocols

Surfaced via the NLP newsletter (Top AI Papers of the Week, 2026-06-21–06-28).

## Summary

As LLM-based [[multi-agent]] systems proliferate, communication protocols become essential infrastructure for distributed agent networks — yet the protocol landscape is fragmented, posing an interoperability problem. This survey (TU München) builds a technical taxonomy to classify and compare LLM-agent communication protocols, aiming to make that fragmented landscape legible. Following an established iterative taxonomy-development method (five iterations: three empirical-to-conceptual, two conceptual-to-empirical) over nine actively maintained open-source protocols with demonstrable adoption, it derives a **five-dimensional taxonomy**: counterparty, payload, interaction state, discovery mechanism, and schema flexibility. The authors argue the field is unlikely to converge on a single protocol and will instead evolve toward a federated, layered protocol stack.

## Key points

- **Five-dimensional taxonomy**: counterparty, payload, interaction state, discovery mechanism, schema flexibility — the meta-characteristic for classifying protocols.
- Grounded in **nine** actively maintained, adopted open-source protocols selected via explicit criteria, including Anthropic's [[mcp]], A2A, [[acp]], ANP, and Agora.
- **Recurring patterns**: all sampled agent-to-agent protocols combine hybrid payloads with session-state persistence; most support multiple predefined schemas (two negotiate schemas at runtime); decentralized discovery remains rare.
- **Short-term** convergence pressure toward protocols unifying agent-to-agent and agent-to-context (tool/data) communication; **long-term** no single protocol maximizes versatility, efficiency, and portability simultaneously.
- Flags open research gaps: privacy and policy enforcement across protocol boundaries.

## Concepts & entities

- [[multi-agent]] — communication protocols as the backbone of collective intelligence in MAS.
- [[mcp]] — Anthropic's Model Context Protocol, one of the nine classified (agent-to-context).
- [[acp]] — Agent Communication Protocol, one of the agent-to-agent protocols classified.

## References

- [arXiv:2606.19135](https://arxiv.org/abs/2606.19135) — Sander, Gidey, Lenz, Knoll. "A Technical Taxonomy of LLM Agent Communication Protocols" (2026). Technische Universität München.
