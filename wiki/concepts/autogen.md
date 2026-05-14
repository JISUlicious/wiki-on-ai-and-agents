---
title: AutoGen
type: concept
created: 2026-05-14
updated: 2026-05-14
sources:
  - wu-autogen-2023.md
builds-on: "[[llm-agent]]"
status: complete
importance: medium
---

# AutoGen

AutoGen is the open-source multi-agent LLM framework introduced by [[autogen-wu-2023|Wu et al. (2023)]] at Microsoft Research. It treats LLM applications as **conversations between customizable agents** — where each agent can be powered by an LLM, a human, or a tool (or any combination) — and provides the message-routing, turn-taking, and termination infrastructure to make those conversations programmable.

## Core abstractions

- **AssistantAgent**: an LLM-backed agent that responds to messages from other agents.
- **UserProxyAgent**: a non-LLM agent that runs code, calls tools, or solicits human input. Common pattern: pair an `AssistantAgent` (writes code) with a `UserProxyAgent` (executes the code, returns results) in a feedback loop.
- **GroupChat**: multiple agents take turns under a moderator that decides who speaks next based on configurable policies.

The framework provides:

- Built-in code execution with Docker sandboxing.
- LiteLLM-style multi-provider LLM integration.
- Conversation logging, caching, and replay.
- Human-in-the-loop hooks.

## Why it took off

AutoGen became the dominant open-source multi-agent framework of 2023–2024 because:

1. **Generic** — minimal prescription about the workflow, so practitioners can build any pattern.
2. **Microsoft-backed** — active maintenance, integrations, docs.
3. **Production-ready code execution** — Docker sandboxing solves a real ergonomics problem.
4. **Composable** — conversable agents compose into group chats, hierarchies, swarms.

Production multi-agent systems (Devin, SWE-agent, etc.) don't typically deploy AutoGen directly, but they implement AutoGen-style patterns. The "two agents talking under a moderator" pattern AutoGen popularized is now the default building block.

## Vs. MetaGPT

- **[[metagpt]]**: opinionated about *what* agents do (SOPs for software engineering). Less flexible but more guided.
- **AutoGen**: opinionated about *how* agents communicate (turn-taking, message routing) but agnostic about *what*. More flexible; requires you to design your own workflow.

Both papers came out within a week of each other in August 2023; the framing-vs-framework split is partly accidental.

## References

- [[autogen-wu-2023]]
