---
title: "LangChain callbacks and middleware: first commits"
type: source
created: 2026-11-02
updated: 2026-11-02
sources: []
year: 2025
url: https://www.langchain.com/blog/agent-middleware
tags:
  - 2025
---

# LangChain callbacks and middleware: first commits

- **Callbacks, 2023-01-04.** Harrison Chase's "Add BaseCallbackHandler and CallbackManager" (#478), commit `9e04c34e2`. This is the earliest dated lifecycle-callback case in this evidence set. Tracing followed on 2023-01-26 (#741).
- **Middleware, 2025-09-07.** "middleware support in `create_agent`" (#32828), commit `6e2f46d04`.
- **Announcement, 2025-09-08.** The [Agent Middleware post](https://www.langchain.com/blog/agent-middleware) names LangChain's earlier pre-model and post-model hooks as predecessors, and uses web-server middleware as an analogy for execution order. It does not mention callbacks or [[claude-code|Claude Code]]. This was read through a summarizing fetch.
- **OpenAI Agents SDK, 2025-03-11.** For comparison, `src/agents/lifecycle.py` is in that repository's [initial commit](https://github.com/openai/openai-agents-python/commit/aaec57a4).

See [[langchain]], [[openai-agents-sdk]], [[langchain-history-2024-2025]] and [[plugins-hooks-harness-influence-evidence]].

## References

- [Agent Middleware](https://www.langchain.com/blog/agent-middleware)
