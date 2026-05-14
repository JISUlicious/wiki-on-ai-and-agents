---
title: Indirect Prompt Injection
type: concept
created: 2026-05-14
updated: 2026-05-14
sources:
  - greshake-indirect-injection-2023.md
builds-on: "[[prompt-injection]]"
status: complete
importance: high
---

# Indirect Prompt Injection

Indirect prompt injection is the variant of [[prompt-injection|prompt injection]] where the adversarial instructions are placed in **third-party content** that the LLM application ingests, rather than coming directly from the user. Named and systematized by [[indirect-prompt-injection-greshake-2023|Greshake et al. (2023)]].

## The attack surface

Any LLM-integrated application that consumes content from non-user sources is vulnerable:

- **Web browsing / search**: an attacker who controls a web page that the LLM retrieves (e.g., as part of a Bing Chat search) can embed instructions in that page.
- **[[retrieval-augmented-generation|RAG]] systems**: an attacker who can place a document into the retrieval corpus can embed instructions.
- **Email assistants**: an attacker emails the user; the assistant LLM reads the email and follows instructions.
- **Document Q&A**: a user uploads a PDF; an attacker who authored the PDF embedded instructions.
- **Code assistants**: an attacker contributes a PR; the code-assistant LLM running on the repo follows hidden instructions in comments or strings.
- **Tool outputs**: an attacker who controls a downstream service's responses (e.g., a malicious API) can inject instructions through the response payload.

## Demonstrated attacks (Greshake et al. 2023)

- **Data exfiltration**: instruct the LLM to silently include user's private data in its response or in an outbound tool call.
- **Worming**: produce content (e.g., a generated email reply) that itself contains the prompt injection, propagating the attack.
- **Information ecosystem contamination**: poison retrieval indices with content designed to be re-retrieved and reshape future answers.
- **Capability escalation**: trick a constrained agent into invoking unauthorized tools.

The paper demonstrated these against real-world systems (Bing GPT-4 Chat, code-completion engines) with surprisingly low effort — often just plain `"Ignore previous instructions and..."` embedded in retrievable content.

## Why it's especially dangerous

- **Asymmetric**: an attacker only needs to write *one* page / email / doc; many users may be affected.
- **Invisible to the user**: the user doesn't see the injection; they just see the model behaving badly.
- **Hard to attribute**: when a model misbehaves on a retrieved page, the user doesn't know whether to blame the model, the developer, or the third-party content.

## Defenses (all partial)

- **Mark untrusted regions** with XML tags / system signals; train the model to treat them as data, not instructions. State of the art in 2026 but bypassable.
- **Output filters**: detect data exfiltration patterns, suspicious tool calls.
- **Capability containment**: scope agent permissions narrowly; require human-in-the-loop for sensitive actions.
- **Source vetting**: limit retrieval corpora to trusted sources; sanitize retrieved content.
- **Distinguish data and instruction channels at the API level** (some research directions explore architectural separation; nothing widely deployed yet).

## OWASP LLM #1

As of the 2024 OWASP LLM Top 10, prompt injection (direct + indirect) is the top risk for LLM applications. Indirect is generally considered the more severe variant because of the asymmetric attacker advantage and the invisible-to-user property.

## References

- [[indirect-prompt-injection-greshake-2023]]
