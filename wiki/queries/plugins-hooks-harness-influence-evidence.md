---
title: "Plugins, Hooks and Harness Engineering: Influence Evidence"
type: query
created: 2026-11-02
updated: 2026-11-02
sources:
  - my-ai-adoption-journey-hashimoto-2026.html
  - pi-coding-agent-zechner-2025.html
  - agentic-harness-engineering-lin-2026.md
  - harness2-recursive-agent-harnessing-xu-2026.md
  - harness-engineering-self-improvement-weng-2026.md
  - graph-engineering-llm-agents-system-intelligence-feng-2026.md
  - sol-pi-scaling-auto-research-loops-efficient-agent-harness-liu-2026.md
status: complete
importance: high
---

# Plugins, Hooks and Harness Engineering: Influence Evidence

Evidence for [[ai-agent-influence-graph]]. **Confirmed** means the later side's primary source states the use. **Weak** means citation or mention. **Unconfirmed** means date order or similarity only.

## Product dates

| Product | First code | Public | Wiki before this check |
|---|---|---|---|
| [[pi-mono|Pi]] | monorepo 2025-08-09; coding agent 2025-10-17 | npm 2025-10-22; write-up 2025-11-30 | 2024-12-01, no support found |
| [[hermes-agent|Hermes Agent]] | 2025-07-22 | launch 2026-02-25/26, secondary only; first tag 2026-03-12 | 2025 |
| [[openclaw|OpenClaw]] | 2025-11-24 as Warelay | names: Clawdis 2025-12-03, Clawdbot 2026-01-04, Moltbot 2026-01-27, OpenClaw 2026-01-30 | 2025 |

Commits are listed in [[agent-products-repo-history-2026]].

## Product relations

| Edge | Grade | Kind | Evidence |
|---|---|---|---|
| Pi → OpenClaw | Confirmed, 2025-12-05 to 2026-05-27 | Runtime embed, now adapted code | Commit `690113dd73a`; old `docs/pi.md`; commit `bb46b79d3c1`; `THIRD_PARTY_NOTICES.md`; `pi-tui` remains |
| Hermes → OpenClaw | Confirmed, narrow | Adapted code | `repetition-guard.ts`; `THIRD_PARTY_NOTICES.md` |
| Pi → Hermes | Confirmed, narrow | RPC mapping | `programmatic-integration.md`, issue #360 |
| OpenClaw → Hermes | Compatibility | Importer | Hermes README, migration guide |
| Claude Code → Pi | Weak for design; confirmed for compatibility | Reaction, skill compatibility | [[pi-coding-agent-zechner-2025]]; commit `09bca967` |
| Claude Code → Hermes | Confirmed by commit title only | CLI likeness, importer | Commit `bc76a032ba`; README |
| Agent Skills → all three | Confirmed | Spec support | Pi `docs/skills.md`; Hermes `features/skills.md`; OpenClaw `docs/tools/skills.md` |
| MCP → all three | Confirmed | Support | Pi `docs/mcp.md` (from 2026-09-29); Hermes `features/mcp.md`; OpenClaw `docs/cli/mcp.md` |
| ACP → Hermes, OpenClaw | Confirmed | Support | `features/acp.md`; `docs/cli/acp.md`. Not found for Pi |
| AGENTS.md → all three | Confirmed | Support | Zechner post; Hermes context files; OpenClaw `docs/concepts/agent.md` |
| ReAct, Aider → any of the three | Not found | | Aider appears only in passing in Hermes; a "ReAct" search of Hermes drowned in React matches, so that one is inconclusive |
| [[gepa-agrawal-2025|GEPA]] → Hermes ecosystem | Confirmed for a separate repo | Skill optimizer | [[hermes-agent-self-evolution-nous-2026]]. Not shown for the built-in learning loop |

## Plugin, hook and callback cases

| Date | Case | Source |
|---|---|---|
| 2023-01-04 | LangChain callbacks | [[langchain-callbacks-middleware-history]] |
| 2023-03-23 | [[chatgpt-plugins|ChatGPT plugins]] | Wiki. Tool extension, not lifecycle hooks |
| 2025-03-11 | OpenAI Agents SDK lifecycle hooks | [[langchain-callbacks-middleware-history]] |
| 2025-06-30 | Claude Code hooks | [[claude-code-hooks-origin-2025]] |
| 2025-09-07 | LangChain middleware commit (GA 2025-10-22) | [[langchain-callbacks-middleware-history]] |
| 2025-10-09 | Claude Code plugins | [[hooks-in-modern-agents]] |
| 2025-12-09 | Pi hooks | [[agent-products-repo-history-2026]] |
| 2026-02-15 | Hermes hooks | [[agent-products-repo-history-2026]] |
| 2026-10-01 | [[claude-code-mods|Claude Code mods]] | [[claude-code-changelog-2026]] |

**No inheritance between vendors is documented**, so these edges are Unconfirmed. Two links within one vendor are stated: LangChain's middleware post names its earlier pre- and post-model hooks, and Claude Code's changelog credits user issue #712. For Claude Code hooks, the stated origin is users and the core team.

## Toward harness engineering

**Hooks and middleware → harness:** these sources treat them as parts of a harness, not as historical ancestors. Grade as Weak if drawn as lineage.

- [[agentic-harness-engineering-lin-2026|AHE]]: middleware is named on p.1 and is one of seven editable component types on p.4. Table 3 (§4.4.1, p.8) gives the component ablation.
- [[harness-r1-learning-to-edit-runtime-harnesses-shao-2026|Harness-R1]]: fixed-patch lifecycle-position ablation, Figure 4(b), p.8; per-benchmark data in Appendix G, noted on p.9.
- [[harness2-recursive-agent-harnessing-xu-2026|Harness²]]: hooks in the component figure on p.3, and plugin component C8 on p.22, described as a hook that enforces behaviour.

**Context engineering → harness engineering:** no succession claim was found.

- The numbered ladder in [[context-engineering]] is the **wiki's own extension**, added in commit `23c17b5`. [[pre-train-prompt-predict-liu-2021|Liu et al.]] end at prompt engineering, in 2021.
- [[graph-engineering-llm-agents-system-intelligence-feng-2026|Feng et al.]] list prompt, context, harness and loop engineering in the abstract as emerging paradigms. §3 traces a progression, with harness engineering extending the model's capability boundary. That is the closest to a sequence, but it is about capability scope and makes no historical claim of inheritance.
- [[harness-engineering-self-improvement-weng-2026|Weng]] treats context engineering as a section inside a harness post. That is inclusion, not succession.

## The term

| Date | Use | Source |
|---|---|---|
| 2025-11-26 | "harnesses" for long-running agents, not the phrase "harness engineering" | [Anthropic](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents), page metadata |
| 2025-11-30 | "coding agent harnesses" | [[pi-coding-agent-zechner-2025]] |
| 2026-02-05 | **"harness engineering", earliest found** | [[my-ai-adoption-journey-hashimoto-2026]] |
| 2026-02-11 | OpenAI post, date second-hand | [[harness-engineering-openai-2026]] |
| 2026-04-28 | AHE v1 (v4 on 2026-05-18), [arXiv:2604.25850](https://arxiv.org/abs/2604.25850) | arXiv |
| 2026-07-04 | Weng's post | post URL |
| 2026-10-09 | Harness² v1, [arXiv:2610.14224](https://arxiv.org/abs/2610.14224) | arXiv |

Earlier uses may exist; no exhaustive search was possible.

## Products as harness cases

- **Pi, Confirmed:** [[sol-pi-scaling-auto-research-loops-efficient-agent-harness-liu-2026|SoL-Pi]] ([arXiv:2609.20519](https://arxiv.org/abs/2609.20519), 2026-09-17) compares with Pi in the abstract and implements its four mechanisms as Pi extensions.
- **OpenClaw, Weak:** [[agent-harness-survey-meng-2026|Meng et al.]] and [[code-as-agent-harness-ning-2026|Ning et al.]] mention it. [[openclaw-rl-train-agent-by-talking-wang-2026]] and [[clawgym-ii-black-box-rl-agent-harness-song-2026]] build on it; these locations were not rechecked.
- **Hermes Agent, not found:** [[hermes-harness-engineering-dev-primitives-jin-2026|HERMES (Jin et al.)]] is an unrelated system despite the name.

## References

- [[agent-products-repo-history-2026]]
- [[claude-code-hooks-origin-2025]]
- [[langchain-callbacks-middleware-history]]
- [[hermes-agent-self-evolution-nous-2026]]
