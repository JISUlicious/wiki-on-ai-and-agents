---
title: Tool Creation
type: concept
created: 2026-05-29
updated: 2026-05-29
sources:
  - llm-as-tool-makers-cai-2023.md
  - craft-yuan-2023.md
status: draft
importance: medium
tags:
  - 2023
---

# Tool Creation

**Tool creation** is the pattern in which an agent *authors its own* reusable tools or functions at runtime — typically as code — rather than being limited to a fixed, human-provided tool set. Having written a tool, the agent caches it (often keyed by a natural-language description) and retrieves it for later, related tasks. This is the "skills as self-authored tools" thread: a one-time, more capable model can act as a *tool maker* that amortizes its effort across many cheaper *tool user* invocations. It connects the toolmaking literature to operational [[skill-library|skill libraries]] and the broader [[agent-skills]] packaging convention.

- **LATM** ([[llm-as-tool-makers-cai-2023|Cai et al. 2023]]) splits the workflow into a strong toolmaker that writes a Python utility plus tests, and a cheaper tool user that calls the cached utility — trading model cost for reuse.
- **CRAFT** ([[craft-yuan-2023|Yuan et al. 2023]]) builds *specialized toolsets* offline by abstracting solutions into reusable functions, then retrieves them at inference time by task similarity.
- Self-authored tools are stored and indexed for retrieval, making tool creation the executable-code sibling of [[experiential-learning]]'s natural-language insight distillation.
- The pattern underpins agents that treat [[code-as-harness|code as their action and abstraction layer]].

## Related

- [[skill-library]] — where created tools accumulate.
- [[library-learning]] — the formal program-synthesis counterpart.
- [[agent-skills]] — packaging convention for discoverable, reusable behavior.

## References

- [[llm-as-tool-makers-cai-2023]]
- [[craft-yuan-2023]]
