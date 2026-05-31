---
title: "Agent Skills Across Vendors — Capability-Packaging Compared"
type: comparison
created: 2026-05-31
updated: 2026-05-31
sources:
  - anthropic-agent-skills-2025.md
  - anthropic-engineering-skills-2025.md
  - agentskills-io-spec-2025.md
  - openai-codex-skills-2025.md
  - agent-skills-survey-xu-2026.md
status: complete
importance: high
tags:
  - 2026
---

# Agent Skills Across Vendors — Capability-Packaging Compared

How the major AI vendors package **reusable, composable agent capabilities** — instructions + code + resources an agent loads to perform a class of tasks. Anthropic's [[agent-skills|Agent Skills]] (`SKILL.md`) set the reference shape; this page maps every other vendor's nearest equivalent against it, drawn from authoritative vendor documentation and the [[agent-skills-survey-xu-2026|Xu & Yan 2026 survey]].

The headline finding: **the SKILL.md format is genuinely converging into a cross-vendor standard.** OpenAI Codex independently shipped `SKILL.md`; the [[agentskills-io-spec-2025|agentskills.io]] open standard lists 40+ adopters. But "skill" as a *word* has drifted across communities, and the underlying packaging models still differ sharply — portable files vs. hosted configs vs. OpenAPI action schemas vs. JSON manifests.

## The reference model — Anthropic Agent Skills

A skill is a **directory** whose entry point is a [[skill-md-format|SKILL.md]] file: YAML frontmatter + a markdown instruction body, plus optional bundled `scripts/`, `references/`, `assets/`. Defining traits:

- **Portable files** — version-controlled, copied between machines, not hosted
- **Progressive disclosure** — L1 metadata in system prompt (~100 tokens) → L2 body on trigger (<5000 tokens) → L3+ nested files navigated on the filesystem (effectively unbounded), per [[anthropic-engineering-skills-2025]]
- **Open spec** — [[agentskills-io-spec-2025|agentskills.io]], 6 frontmatter fields (`name`, `description`, `license`, `compatibility`, `metadata`, `allowed-tools`)
- **Code as tool + doc** — bundled scripts run via bash without loading into context; only output consumes tokens

## Cross-vendor matrix

| Vendor unit | Form | Instructions | Code/resources | Portable file | Open spec | Progressive disclosure |
|---|---|---|---|---|---|---|
| **Anthropic Agent Skill** | folder + SKILL.md | yes | yes | yes | yes (agentskills.io) | **yes (L1/L2/L3+)** |
| **OpenAI Codex Agent Skill** | folder + SKILL.md | yes | yes (scripts/refs/assets) | yes | adopts agentskills.io | **yes (~2% cap)** |
| **OpenAI GPTs** | hosted config | yes | knowledge files + actions | no | no | no |
| **OpenAI AgentKit** | versioned workflow graph | per-node | tool nodes | no (hosted) | no | no |
| **OpenAI Apps SDK** | MCP server | no | hosted tools + UI | no | MCP-based | no |
| **Google Gemini CLI Extension** | gemini-extension.json + GEMINI.md | yes | MCP + commands | yes (Git) | repo-public | **yes ("only when needed")** |
| **Google Gemini Gem** | hosted config | yes | knowledge only | no | no | no |
| **Google A2A AgentSkill** | JSON descriptor | no (advertises) | no | n/a | yes (Linux Foundation) | no |
| **Google Vertex ADK Tool** | code function | no | yes | partial | OSS framework | no |
| **Microsoft Declarative Agent** | JSON manifest | yes (≤8000 chars) | via action plugins | yes | public JSON Schema | no |
| **Microsoft Semantic Kernel plugin** (ex-"skill") | functions | prompt fns | yes | yes | aligns OpenAI plugin spec | no |
| **AWS Bedrock Action Group** | OpenAPI + Lambda | no | yes | partial (S3) | OpenAPI-based | no |
| **AWS Bedrock AgentCore Gateway** | MCP tool/target | no | yes (hosted) | no | MCP | partial (semantic selection) |

## Reading the matrix

**Three genuine SKILL.md-format adopters.** Only [[openai-codex-skills-2025|OpenAI Codex]] and Google's **Gemini CLI Extensions** match Anthropic on all the defining traits — portable files, instructions + bundled code, on-demand loading. Codex literally uses `SKILL.md` (divergence is cosmetic: `.agents/skills` path vs `.claude/skills`, an `agents/openai.yaml` metadata file, ties to the [[agents-md|AGENTS.md]] ecosystem). Gemini CLI Extensions bundle what Google's own docs call "Agent Skills — a specialized set of instructions and workflows the model activates only when needed."

**Hosted-config equivalents.** OpenAI **GPTs** and Google **Gems** are the closest *in spirit* (instructions + knowledge + actions) but furthest in *portability* — they're proprietary, hosted, single-platform, and not loaded on demand into an arbitrary agent. GPTs predate everything (Nov 2023) as the original "instructions + knowledge + tools" bundle.

**Manifest equivalents.** Microsoft **Declarative Agents** (M365 Copilot) package instructions (≤8000 chars) + scoped capabilities + actions into a versioned JSON manifest with a public schema — the closest enterprise analog, but no progressive disclosure and no portable code bundle.

**Schema/connector equivalents.** AWS **Bedrock Action Groups** (OpenAPI + Lambda), Google **A2A AgentSkill** (a discovery descriptor, not a bundle), and the MCP-based units (Apps SDK, AgentCore Gateway) package *what to connect to* rather than *how to do a task* — they sit at the [[mcp|MCP]]/tool layer, complementary to skills rather than equivalent.

## Terminology drift — "skill" has moved around

- **Microsoft used "skills" first** — Semantic Kernel's original capability bundles were "skills" (collections of native + semantic functions), then **renamed to "plugins" in October 2023** to align with the OpenAI plugin spec. Microsoft abandoned the term Anthropic later adopted.
- **Google currently uses "skill"** in two places: A2A's `AgentSkill` descriptor and Gemini CLI's "Agent Skills."
- **RL pre-history**: "skill" in pre-LLM RL meant a temporally-extended sub-policy (an *option* with a termination condition) — a different formalism entirely. The term carries genuine semantic drift across communities (see [[agent-skills-survey-xu-2026]]).

## Academic vs. industry definitions of "skill"

The [[agent-skills-survey-xu-2026|Xu & Yan 2026 survey]] is the first to attempt a unifying academic definition. Two distinct lineages exist:

| Lineage | Definition of "skill" | Rigor | Examples |
|---|---|---|---|
| **Program-synthesis** | A lambda-abstraction extracted by formal refactoring/compression that provably preserves execution semantics | rigorous | [[dreamcoder]] → [[lilo-grand-2023|LILO]] → ReGAL |
| **LLM-agent (code)** | An executable code module, description-indexed, retrieved by embedding, append-only, composable | operational | [[voyager]], [[skill-library]] |
| **LLM-agent (NL)** | Distilled natural-language insight or rule, no executable code | loose | [[expel-zhao-2023|ExpeL]] |
| **Industry (Anthropic et al.)** | A portable file bundle of instructions + code + resources loaded on demand | operational/normative | [[agent-skills]], [[agentskills-io-spec-2025|agentskills.io]] |

The **self-authored tool** thread ([[llm-as-tool-makers-cai-2023|LATM]], [[craft-yuan-2023|CRAFT]]) bridges program-synthesis and LLM-agent lineages: agents that *write their own reusable skills* at runtime.

## Is it accepted / standardized?

- **Open standard exists**: [[agentskills-io-spec-2025|agentskills.io]] (published 2025-12-18, maintained by Anthropic, open contributions). Reference validator `skills-ref`. 40+ adopters listed (Cursor, GitHub Copilot, VS Code, OpenAI Codex, Gemini CLI, Goose, Letta).
- **Academic recognition**: a dedicated survey ([[agent-skills-survey-xu-2026]], Zhejiang University) treats agent skills as a first-class research area as of early 2026.
- **But not universal**: each big vendor still ships its *own* primary packaging unit (GPTs, Gems, Declarative Agents, Action Groups). The SKILL.md convergence is real among coding agents and the open-spec adopters, but the hyperscaler enterprise stacks (M365, Bedrock, Vertex) have not adopted SKILL.md as their native format.

## Verdict

The packaging of reusable agent capability is **mid-convergence**:
- The **coding-agent cluster** (Claude Code, Codex, Gemini CLI, Cursor, Goose) has effectively standardized on SKILL.md via agentskills.io.
- The **enterprise-assistant cluster** (M365 Declarative Agents, Bedrock, Vertex, GPTs, Gems) each retains a proprietary manifest/config, mapping to the same conceptual slots (instructions + capabilities + actions) without sharing a file format.
- **MCP is the orthogonal layer** — every vendor uses or interops with it for *tool connection*, while skills/manifests handle *task instruction*. The two compose.

## Related pages

- [[agent-skills]] — Anthropic's reference model (the concept page)
- [[skill-md-format]] — the file-format spec as cross-agent interop
- [[agentskills-io-spec-2025]] — the formal open standard
- [[agent-skills-survey-xu-2026]] — the academic survey
- [[skill-library]] — the Voyager-lineage executable skill library
- [[mcp]] — the complementary tool-connection protocol
- [[cross-agent-comparison-2026]] — the broader 5-agent comparison

## References

- [[anthropic-agent-skills-2025]] — Anthropic announcement
- [[anthropic-engineering-skills-2025]] — Anthropic engineering deep-dive
- [[agentskills-io-spec-2025]] — the open standard
- [[openai-codex-skills-2025]] — OpenAI's convergent SKILL.md adoption
- [[agent-skills-survey-xu-2026]] — Xu & Yan 2026 academic survey
- Google A2A: a2a-protocol.org/latest/specification (Linux Foundation)
- Microsoft Declarative Agent schema 1.5: learn.microsoft.com (Microsoft 365 Copilot extensibility)
- Microsoft Semantic Kernel "skills→plugins": devblogs.microsoft.com/semantic-kernel (Oct 2023)
- AWS Bedrock Agents action groups: docs.aws.amazon.com/bedrock
