---
title: "Not what you've signed up for: Compromising Real-World LLM-Integrated Applications with Indirect Prompt Injection"
type: source
created: 2026-05-14
updated: 2026-05-14
sources:
  - greshake-indirect-injection-2023.md
arxiv_id: "2302.12173"
venue: AISec '23
authors:
  - Kai Greshake
  - Sahar Abdelnabi
  - Shailesh Mishra
  - Christoph Endres
  - Thorsten Holz
  - Mario Fritz
first_author: Kai Greshake
introduces:
  - "[[indirect-prompt-injection]]"
year: 2023
tags:
  - 2023
status: complete
importance: high
---

# Not what you've signed up for: Compromising Real-World LLM-Integrated Applications with Indirect Prompt Injection

**Source**: `sources/greshake-indirect-injection-2023.md` (extracted from `greshake-indirect-injection-2023.pdf`)
**arXiv**: [2302.12173](https://arxiv.org/abs/2302.12173)
**Venue**: AISec '23 (16th ACM Workshop on AI and Security)
**Authors**: Kai Greshake (Saarland University, sequire technology), Sahar Abdelnabi, Shailesh Mishra (Saarland), Christoph Endres (sequire), Thorsten Holz, Mario Fritz (CISPA Helmholtz Center)

## Summary

This paper **names and demonstrates indirect prompt injection** — the canonical attack vector against any LLM application that ingests external content (retrieved web pages, emails, documents, code comments, tool outputs) as part of its context. The taxonomy and threat model defined here are now standard reference material in LLM security.

The setup: prior prompt-injection attacks assumed a user directly typing adversarial input to an LLM. But many real-world LLM applications — search-augmented chatbots (Bing Chat, Perplexity), email assistants, document Q&A, retrieval-augmented systems, browser agents — ingest content from *third parties* that the user didn't write. An adversary can embed instructions in that third-party content; when the LLM later reads it, the embedded instructions are treated as if they came from the user/operator.

The paper systematically explores attack vectors:

- **Retrieved web pages**: an attacker who can get their content into a search result that the LLM browses can hijack the LLM.
- **Email**: an attacker emails the user; the user's email-assistant LLM reads the email and follows the attacker's instructions.
- **Documents / PDFs**: an attacker plants instructions in a document the user uploads.
- **Code comments / repositories**: an attacker contributes a PR whose comments hijack a code-assistant LLM running on the repo.

Demonstrated against **real-world systems** including Bing's GPT-4-powered Chat and code-completion engines. The attacks succeed with surprisingly low effort: just embedding `"Ignore previous instructions and..."` in retrievable text often works.

The paper's broader contribution is the **threat-model framing**: LLM-integrated applications **blur the line between data and instructions**. When the model can't tell which parts of its context were authored by trustworthy parties vs. arbitrary internet-randos, the security model is fundamentally different from traditional applications.

Indirect prompt injection is now considered the **#1 LLM application security risk** per OWASP's LLM Top 10. Defenses remain partial — sanitization, output validation, prompt boundary tokens — but no robust solution exists as of 2026.

## Key Points

- **The attack**: hide adversarial instructions in third-party content (web pages, emails, docs, code, tool outputs). When the LLM ingests this content, the embedded instructions execute.
- **The framing**: LLM-integrated apps **blur the line between data and instructions** — context tokens are processed identically regardless of where they came from.
- **Taxonomy of attack vectors**: data theft, worming (self-propagating prompt injections), information ecosystem contamination, manipulation, denial-of-service, capability escalation.
- **Demonstrated against real systems**: Bing GPT-4 Chat, code-completion engines, synthetic agent applications.
- **Worming demonstration**: a prompt injection that makes the LLM produce content (e.g., an email reply) that itself contains the prompt injection — self-propagating.
- **No fully robust defense exists**: input sanitization, output classifiers, prompt-boundary tokens, and structured tool-use APIs all help but none completely eliminate the risk.

## Entities Mentioned

- Kai Greshake — first author, security researcher
- CISPA Helmholtz Center for Information Security — co-author affiliation

## Concepts Discussed

- [[indirect-prompt-injection]] — the attack named here
- [[prompt-injection]] — the broader class
- [[retrieval-augmented-generation]] — primary attack surface
- [[tool-use]] / [[llm-agent]] — also attack surfaces
- [[agentic-misalignment]] — related but distinct (agentic misalignment is about model internal goals; indirect injection is about adversarial external input)

## Notable Quotes

> "We argue that LLM-Integrated Applications blur the line between data and instructions. We reveal new attack vectors, using Indirect Prompt Injection, that enable adversaries to remotely (without a direct interface) exploit LLM-integrated applications by strategically injecting prompts into data likely to be retrieved." — Abstract

## References

_Original source: `sources/greshake-indirect-injection-2023.md`_
