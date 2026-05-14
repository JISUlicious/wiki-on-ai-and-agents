---
title: Prompt Injection
type: concept
created: 2026-05-14
updated: 2026-05-14
sources:
  - greshake-indirect-injection-2023.md
status: complete
importance: high
---

# Prompt Injection

Prompt injection is the class of LLM-application attacks where an adversary's text input causes the model to **override its operator-provided instructions** and follow the adversary's instructions instead. The LLM treats all the text in its context as potentially-instructions — there's no syntactic distinction between "the developer's system prompt" and "user input" beyond convention. An attacker who can place text into the context can hijack the model.

Two main variants:

- **Direct prompt injection**: the attacker is the *user* who is supposed to be interacting with the LLM. They prompt the model with adversarial text to bypass its restrictions.
- **[[indirect-prompt-injection|Indirect prompt injection]]**: the attacker is a *third party* who plants malicious instructions in content the LLM ingests from elsewhere (web pages, emails, retrieved docs, tool outputs). Named and systematized by [[indirect-prompt-injection-greshake-2023|Greshake et al. 2023]].

## Why the model treats data as instructions

LLMs are trained to follow instructions in their context. They have **no architectural mechanism** to distinguish "this token came from a trusted source" vs. "this token came from arbitrary internet content." Once an adversary's instruction is in the context window, the model treats it identically to any other instruction.

System prompt boundaries (e.g., `### SYSTEM\n...\n### USER\n...`) are *conventions*, not enforced separators. A determined attacker can include the boundary tokens themselves in their payload, or just write `"Ignore previous instructions and..."` which often works because that pattern is well-represented in RLHF data.

## OWASP LLM Top 10

Direct and indirect prompt injection together top the **OWASP LLM Top 10** as the most-cited LLM-application security risks. As of 2026, no fully robust defense exists.

## Partial defenses

- **Input sanitization**: try to detect and strip injection-like patterns. Easy to bypass.
- **Output classifiers**: a separate model checks the output for compliance. Adds latency; can be bypassed.
- **Structured tool-use APIs**: don't let the model run free-form actions; restrict to a defined toolbox with typed arguments. Limits blast radius but doesn't prevent the injection.
- **Prompt-boundary tokens with training**: train the model to take the system prompt as inviolable. Helps but doesn't eliminate.
- **Capability containment**: don't give the agent sensitive permissions (file write, network access) unless absolutely needed. Defense-in-depth.
- **Data labels with model awareness** (Anthropic XML tags, OpenAI dev-prompt): mark trusted vs. untrusted regions and train the model to respect the distinction. State of the art in 2026 but still partial.

## Why it's an unsolved problem

Distinguishing "instruction" from "data" requires reasoning about *intent*, *trust*, and *context source* — concepts the LLM only has fuzzy access to. As models become more capable instruction-followers, they also become more capable victim-of-instructions. The trade-off isn't fully resolvable without architectural changes (separate channels for instructions vs. data, formally verified output filters, etc.).

## References

- [[indirect-prompt-injection-greshake-2023]]
