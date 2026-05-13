---
title: ReAct (Reasoning + Acting)
type: concept
created: 2026-05-13
updated: 2026-05-13
sources:
  - react-yao-2022.md
status: complete
importance: high
---

# ReAct (Reasoning + Acting)

ReAct is a prompting framework introduced by [[react-yao-2022|Yao et al. (2022)]] that combines verbal reasoning with action execution in a single LLM trajectory. The model alternates between:

- `Thought:` — free-form natural-language reasoning, planning, reflection.
- `Act:` — an action to execute against an external environment or tool (search engine, knowledge base, embodied environment, code interpreter, etc.).
- `Obs:` — the observation returned by the environment.

The loop continues until the model emits a special `Finish[answer]` action.

## Example trace

```
Question: Aside from the Apple Remote, what other device can control the
program Apple Remote was originally designed to interact with?

Thought: I need to search Apple Remote and find the program it was
originally designed to interact with.
Act: Search[Apple Remote]
Obs: The Apple Remote is a remote control … originally designed to control
the Front Row media center program…

Thought: Apple Remote was originally designed to control the Front Row
media center program. I need to search Front Row next and find what other
device can control it.
Act: Search[Front Row]
...
Finish[keyboard function keys]
```

## Why it works

- **Grounds [[chain-of-thought-prompting|CoT]]** in external facts, preventing the model from "running off into hallucination."
- **Allows [[chain-of-thought-prompting|CoT]]-style planning** between actions, fixing the brittleness of reflex-style Act-only agents.
- **Self-corrects**: when an action returns an unhelpful observation, the next thought can re-plan.
- **Few-shot capable**: works at 1–6 demonstrations on tasks where RL baselines need thousands of episodes.

## What "Act" can be

Almost anything serializable as text:

- Web search / Wikipedia lookup
- Database / SQL query
- Code execution (Python REPL)
- Calculator / symbolic math
- Embodied robot actions
- HTTP requests / API calls
- Other LLMs (multi-agent setups)

This generality is why ReAct underpins virtually every modern **[[llm-agent|LLM agent]]** framework — AutoGPT, LangChain ReAct agent, OpenAI function-calling agents, Claude's tool-use API.

## Limitations

- **Context length**: long trajectories accumulate observations that consume the prompt window.
- **Brittle action parsing**: malformed `Act:` strings can derail execution. Modern tool-use APIs (OpenAI function calling, Anthropic tool use) replace text parsing with structured JSON arguments.
- **No persistent memory beyond the current trajectory** — unless paired with external memory ([[reflexion|Reflexion]]) or vector stores.

## References

- [[react-yao-2022]]
