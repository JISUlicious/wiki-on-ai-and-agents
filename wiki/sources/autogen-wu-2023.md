---
title: "AutoGen: Enabling Next-Gen LLM Applications via Multi-Agent Conversation"
type: source
created: 2026-05-14
updated: 2026-05-14
sources:
  - wu-autogen-2023.md
arxiv_id: "2308.08155"
venue: arxiv preprint (later COLM 2024)
authors:
  - Qingyun Wu
  - Gagan Bansal
  - Jieyu Zhang
  - Yiran Wu
  - Beibin Li
  - Erkang Zhu
  - Chi Wang
first_author: Qingyun Wu
introduces:
  - "[[autogen]]"
year: 2023
tags:
  - 2023
status: complete
importance: medium
---

# AutoGen: Enabling Next-Gen LLM Applications via Multi-Agent Conversation

**Source**: `sources/wu-autogen-2023.md` (extracted from `wu-autogen-2023.pdf`)
**arXiv**: [2308.08155](https://arxiv.org/abs/2308.08155)
**Venue**: arxiv preprint Aug 2023 (later COLM 2024)
**Authors**: Qingyun Wu (Penn State + Microsoft Research), Gagan Bansal, Jieyu Zhang, ..., Chi Wang (Microsoft Research)

## Summary

AutoGen is an **open-source framework for building multi-agent LLM applications** as conversations between customizable "conversable" agents. Each agent can be powered by an LLM, a human, a tool, or some combination. Conversations are scripted (via natural language or code) and agents can take turns based on configurable policies.

Where [[metagpt-hong-2023|MetaGPT]] is opinionated about *what* agents do (SOPs for software engineering), AutoGen is **opinionated about the infrastructure** — it makes "two agents talking to each other under a controller" a default building block, with hooks for tool use, code execution, and human-in-the-loop, but minimal prescription about the workflow itself.

The two canonical agent types:

- **AssistantAgent**: an LLM agent that responds to user/other-agent messages.
- **UserProxyAgent**: a non-LLM agent that can execute code, call tools, or solicit human input. Often used to close the loop between an LLM's suggestions and the real world.

Conversations between these two cover a remarkable range of patterns: assistant-assistant, assistant-tool-user-loop, group chats with multiple specialized assistants and a moderator, hierarchical agents, and so on.

AutoGen became the most-widely-adopted open-source multi-agent framework for several reasons: (1) Microsoft Research backing and active maintenance; (2) excellent integration with LiteLLM / OpenAI API / Azure; (3) clean conceptual model (agents + conversations); (4) production-ready code execution sandbox; (5) good documentation and many template applications. By 2024–2025, "AutoGen-style" multi-agent setups were a common phrase in the LLM-agent literature.

## Key Points

- **Conversable agents** with customizable behaviors and conversation policies.
- **Three modes of agent operation**: LLM-backed, human-input-backed, tool/code-execution-backed (these can be mixed).
- **Conversation programming**: define agent interactions in natural language or code — the framework handles message routing, turn-taking, and termination conditions.
- **Built-in code execution** with Docker sandboxing — important for "let the agent write and run Python".
- **Group chat** with a moderator: multiple specialized agents speak in turns; the moderator picks who speaks next.
- **Cross-domain demos**: math problem solving, coding (with execute-and-debug loops), retrieval, dynamic group decision making, online entertainment (text-based games), conversational chess.

## Entities Mentioned

- Qingyun Wu, Chi Wang — leads (Microsoft Research)
- Microsoft Research — author affiliation
- [[autogen]] — the framework

## Concepts Discussed

- [[autogen]] — the framework
- [[llm-agent]] — multi-agent infrastructure
- Multi-agent orchestration
- Code execution as an agent capability

## Notable Quotes

> "AutoGen is an open-source framework that allows developers to build LLM applications via multiple agents that can converse with each other to accomplish tasks. AutoGen agents are customizable, conversable, and can operate in various modes that employ combinations of LLMs, human inputs, and tools." — Abstract

## References

_Original source: `sources/wu-autogen-2023.md`_
_Open-source: https://github.com/microsoft/autogen_
