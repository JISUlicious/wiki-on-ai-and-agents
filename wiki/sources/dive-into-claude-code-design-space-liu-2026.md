---
title: "Dive into Claude Code: The Design Space of Today's and Future AI Agent Systems"
type: source
created: 2026-06-03
updated: 2026-06-03
sources:
  - dive-into-claude-code-design-space-liu-2026.md
arxiv_id: "2604.14228"
authors:
  - Jiacheng Liu
  - Xiaohan Zhao
  - Xinyi Shang
  - Zhiqiang Shen
first_author: Jiacheng Liu
year: 2026
tags: [2026]
status: complete
importance: medium
---

# Dive into Claude Code: The Design Space of Today's and Future AI Agent Systems

*Surfaced via the alphaXiv / NLP-newsletter digest.*

## Summary

A source-level architectural study of **Claude Code** (analyzing its publicly available TypeScript source, v2.1.88), compared against **OpenClaw**, an independent open-source agent system answering the same design questions from a different deployment context. The authors distill five human values motivating the architecture (human decision authority, safety & security, reliable execution, capability amplification, contextual adaptability) and trace them through thirteen design principles to concrete implementation choices. The core is a simple while-loop (call model, run tools, repeat); most complexity lives in the systems around the loop. They close with six open design directions for future agent systems.

## Key points

- Core agentic loop is a simple while-loop; the bulk of the code is the surrounding systems.
- **Permission system**: seven modes plus an ML-based classifier for per-action safety evaluation.
- **Context management**: a five-layer compaction pipeline; append-oriented session storage.
- **Four extensibility mechanisms**: MCP, plugins, skills, and hooks; plus subagent delegation and orchestration.
- OpenClaw comparison shows deployment context flips design answers: per-action safety → perimeter access control, single CLI loop → embedded runtime in a gateway control plane, context-window extension → gateway-wide capability registration.
- Cites Anthropic's internal survey (132 engineers): ~27% of Claude Code-assisted tasks were work that would not have been attempted otherwise. From VILA Lab, MBZUAI.

## Concepts & entities

- [[claude-code]]
- [[code-as-harness]]
- [[openclaw]]
- [[llm-agent]]
- [[mcp]]
- [[agent-skills]]
- [[context-assembly]]
- [[anthropic]]

## References

- [arXiv:2604.14228](https://arxiv.org/abs/2604.14228)
- Source text: `sources/dive-into-claude-code-design-space-liu-2026.md`
