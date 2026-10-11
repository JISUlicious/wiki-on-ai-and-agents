---
title: "Harness Engineering for Self-Improvement (Lil'Log)"
type: source
created: 2026-10-07
updated: 2026-10-08
sources:
  - harness-engineering-self-improvement-weng-2026.md
authors:
  - "[[lilian-weng]]"
introduces:
  - "[[agentic-harness-engineering]]"
year: 2026
venue: "Lil'Log"
url: https://lilianweng.github.io/posts/2026-07-04-harness/
tags:
  - 2026
  - blog
  - survey
status: complete
importance: high
---

# Harness Engineering for Self-Improvement (Lil'Log)

**Source**: `sources/harness-engineering-self-improvement-weng-2026.md` ([lilianweng.github.io/posts/2026-07-04-harness](https://lilianweng.github.io/posts/2026-07-04-harness/))
**Author**: [[lilian-weng]]
**Date**: **2026-07-04**

## Summary

A survey of **harness engineering** as the near-term route to **recursive self-improvement (RSI)**. Weng traces RSI to I. J. Good (1965) and Yudkowsky (2008). She argues the practical path is not a model rewriting its weights. It is models improving the *deployment system* around them, which in turn enables auto-research and better successor models.

**Definition:** "A harness is the system surrounding a base model that orchestrates execution and decides how the model thinks and plans, calls tools and acts, perceives and manages context, stores artifacts, and evaluates results."

**What's new relative to the 2023 component model:** the early-agent formula was "agent = LLM + memory + tools + planning + action", her own [[llm-powered-autonomous-agents-weng-2023|2023 framing]]. Harnesses add:
- **workflow design ("e.g. loop engineering")**;
- evaluation;
- permission controls;
- persistent state management.

The result is "closer to runtime and software system design". She draws an operating-system analogy: encapsulate complexity, keep interfaces simple, and expect standardization.

> [!note] "Loop engineering"
> This post uses the phrase "loop engineering" as an example of workflow design. It is an earlier dated use than [[nvidia-oo-agents-furgale-2026|NOOA]]'s "programmable loop engineering". It does not define or coin the term.

## Harness design patterns

1. **Workflow automation:** a goal-oriented plan → execute → observe/test → improve loop. Karpathy's autoresearch repo and the Codex agent loop are her examples.
2. **The file system as persistent memory:** keep durable state (logs, diffs, traces) in files rather than in the context.
3. **Sub-agents and background jobs:** parallelism should be "explicit and inspectable", with outputs stored as files and status records. See [[subagents]].
4. **Case study:** coding-agent tool sets have converged across Claude Code, Codex, OpenCode and Cursor. They cover file discovery and editing, shell, git/LSP, MCP and skills, web, background cron jobs, and agent delegation.

## Harness layer vs core intelligence

**Her prediction:** harness engineering moves toward *meta-methodology*, where the harness itself is the optimization target. Many harness improvements may eventually be "internalized into core model behavior", much as prompt tricks faded once instruction tuning improved. The interface to context and tools remains.

**The optimization target has moved:** instruction prompts → structured context → workflow → harness code → optimizer code.

## Harness optimization: the research she surveys

- **Context engineering:**
  - [[agentic-context-engineering|ACE]]: a generator, a reflector and a curator maintain an itemized playbook.
  - Meta Context Engineering (Ye 2026): evolves context-management *skills* in a bi-level loop.
  - [[meta-harness-lee-2026|Meta-Harness]]: a coding agent proposes whole harnesses, reading its history through grep and cat.
- **Workflow design:**
  - AI Scientist and ScientistOne;
  - [[autodata-agentic-data-scientist-kulikov-2026|Autodata]], with challenger, weak and strong solvers, and a verifier;
  - [[adas|ADAS]], a meta-agent search over agent code;
  - AFlow, MCTS over workflow graphs.
- **Self-improving harnesses:**
  - **STOP** (Zelikman 2023) recursively improves the *improver*. It helped GPT-4 but degraded GPT-3.5 and Mixtral: "the base model must be capable enough".
  - **Lin et al. 2026:** the ability to *produce* useful harness edits is flat across models, while the *benefit* from an evolved harness is non-monotonic, with middle-tier models gaining most.
  - **[[self-harness-harnesses-that-improve-themselves-zhang-2026|Self-Harness]]:** mine weaknesses into verifier-grounded failure patterns, propose bounded edits, and accept only with no regression on both held-in and held-out data.
  - **[[agentic-harness-engineering-lin-2026|AHE]]:** three observability pillars and seven harness components. The verifier and configuration are read-only to block reward hacking. Every edit is a falsifiable prediction.
- **Evolutionary search:**
  - Promptbreeder, [[gepa|GEPA]];
  - AlphaEvolve, ThetaEvolve, ShinkaEvolve, DemoEvolve;
  - [[darwin-godel-machine|DGM]]: SWE-bench Verified 20% → 50%, Polyglot 14.2% → 30.7%, with a fixed model;
  - [[hyperagents-zhang-2026|Hyperagents]].
  - These work "when candidate solutions are automatically evaluable".
- **Joint optimization with weights:** SIA (Hebbar 2026), whose evidence she calls "provisional" because of confounds, and Continual Harness (Karten 2026).

## Future challenges (her list)

1. **Weak and fuzzy evaluators.** Loops work where metrics are objective; research taste is not.
2. **Context and memory lifecycle.** "Context engineering will and should become a core part of intelligence."
3. **Negative results** are under-represented in training data, so models are poor at abandoning hypotheses.
4. **Diversity collapse** in evolutionary and RL loops.
5. **Reward hacking.** "The evaluator and permission control should likely sit outside the loop that evolves harness."
6. **Long-term success**, e.g. repository maintainability, is not captured by sandbox RLVR.
7. **The role of humans.** Humans should "move up the stack, not be removed from the loop".

Auto-research failure modes she cites (Trehan & Chopra 2026): bias toward training-data defaults, implementation drift, memory degradation, over-optimism ("numerical duct tape"), lack of tacit domain knowledge, and weak scientific taste.

## Why it matters for the wiki

This is a trusted-source synthesis of the wiki's 2026 harness cluster ([[agentic-harness-engineering]], [[self-improving-agent]], [[harness-vs-model-attribution]]). Its emphasis on keeping the evaluator *outside* the loop matches the wiki's [[verifier]] thesis.

For lineage, it ties the 2023 component model (planning, memory, tools) to the 2026 harness model (plus workflow and loop, evaluation, permissions, state).

## Concepts & entities

[[agentic-harness-engineering]] · [[self-improving-agent]] · [[context-engineering]] · [[agentic-context-engineering]] · [[subagents]] · [[darwin-godel-machine]] · [[adas]] · [[verifier]] · [[harness-vs-model-attribution]] · [[lilian-weng]]
