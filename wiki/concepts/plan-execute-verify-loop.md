---
title: Plan-Execute-Verify Loop (PEV)
type: concept
created: 2026-09-29
updated: 2026-09-29
sources:
  - code-as-agent-harness-ning-2026.md
  - agentcoder-huang-2023.md
status: draft
importance: medium
---

# Plan-Execute-Verify Loop (PEV)

The **Plan-Execute-Verify (PEV) loop** is the control abstraction that the [[code-as-agent-harness-ning-2026]] survey (§3.4) introduces. It brings planning, debugging, verification and escalation together as a single cybernetic control process running over *executable program state*. It has three phases. **Plans are contracts**: they declare invariants and rollback points. **Execution is sandboxed and permissioned**, in tiers (read-only, sandbox-edit, full-access), with human-in-the-loop gates on the riskiest tier and audit logs throughout. **Verification uses deterministic sensors**: linters, type checkers, tests, fuzzers, static analyzers, CI and runtime monitors. In the survey's words, reliability "comes from governed state transitions, not simply from better repair prompts."

## In this wiki

- **Iterative debugging, reframed.** The survey recasts Self-Debugging, [[reflexion]], LDB and QualityFlow as instances of the verify-and-repair part of PEV, and names AGENTS.md, [[mcp|MCP]] server registries and gateway or policy layers as contract layers.
- **AgentCoder as the canonical example.** [[agentcoder-huang-2023|AgentCoder]] is the most-cited paper in the survey body and the standard example of the generate-test-debug pattern that anchors §3.4. See [[agentcoder]].
- **A place in the harness.** [[code-as-harness]] lists PEV among the system-provided harness mechanisms, alongside planning, memory, tool use and optimization. The [[fundamental-components-of-harness]] query treats it as *the* cybernetic control loop of a harness.
- **Its known weak point.** The survey warns that executable feedback can give false confidence, since a green test is not the full specification. It proposes "evidence bundles" that state what was checked, what was not, and what residual risk remains. See [[verifier]] and [[reward-hacking]].

## Related

- [[code-as-harness]] · [[agentic-harness-engineering]] · [[agent-three-layer-model]] · [[planning]]
- [[reflexion]] · [[self-reflection]] · [[react]] · [[agentcoder]] · [[verifier]] · [[guardrailing]]
- [[program-of-thoughts]] · [[ralph-loop]]

## References

- [[code-as-agent-harness-ning-2026]] (§3.4) · [[agentcoder-huang-2023]]
