---
title: "Boris Cherny on RAG and agentic search in Claude Code"
type: source
created: 2026-11-01
updated: 2026-11-01
sources:
  - cherny-agentic-search-statements.md
  - claude-code-latent-space-2025.md
authors:
  - Boris Cherny
year: 2026
url: https://x.com/bcherny/status/2017824286489383315
status: complete
importance: high
tags:
  - 2026
---

# Boris Cherny on RAG and agentic search in Claude Code

Three first-hand statements by the [[claude-code]] lead.

| Date | Where | What he says |
|---|---|---|
| 2025-02-24 | [Hacker News](https://news.ycombinator.com/item?id=43164253) | Claude Code does not use RAG; in the team's testing agentic search out-performed it for typical uses |
| 7 May (2025 inferred) | [Latent Space](https://www.latent.space/p/claude-code), 00:45:30–00:46:20 | Very early versions indexed the code with off-the-shelf RAG, "I think" using Voyage. Other search tools were tried. Agentic search won. |
| 2026-02-01 | [X](https://x.com/bcherny/status/2017824286489383315) | Early versions used RAG and a local vector database; agentic search generally works better and is simpler |

## Reasons given

- **Performance:** it "outperformed everything". Asked for the benchmark, he says mostly internal vibes, with some internal benchmarks. No numbers are given.
- **Indexing costs:** code drifts out of sync with the index, and an index held by a provider is a security liability.
- **Trade-off:** he states it costs latency and tokens.

> [!warning] Limits
> This is a developer's account of one product. No evaluation was published. It says nothing about other tools or about the field.
