---
title: "Context Engineering for AI Agents: Lessons from Building Manus"
type: source
created: 2026-11-01
updated: 2026-11-01
sources:
  - context-engineering-lessons-manus-ji-2025.md
authors:
  - Yichao Ji
year: 2025
url: https://manus.im/blog/Context-Engineering-for-AI-Agents-Lessons-from-Building-Manus
status: complete
importance: medium
tags:
  - 2025
  - blog
---

# Context Engineering for AI Agents: Lessons from Building Manus

Yichao "Peak" Ji, Manus blog, **18 July 2025**. A builder's account of [[manus|Manus]].

## Key points

- The team chose to build on the in-context learning of frontier models instead of training an end-to-end agent model, and the author ties this to slow fine-tuning loops in the BERT era.
- The agent framework was rebuilt four times while the team learned how to shape context; KV-cache hit rate is treated as a key metric.
- As tools multiply, and the author says MCP adds to this, they mask tools with response prefill instead of removing them from context.

See [[context-engineering]].
