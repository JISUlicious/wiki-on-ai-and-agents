---
title: "Agent Skills for LLMs: Architecture, Acquisition, Security, and the Path Forward"
type: source
created: 2026-05-29
updated: 2026-05-29
sources:
  - agent-skills-survey-xu-2026.md
arxiv_id: "2602.12430"
authors:
  - Renjun Xu
  - Yang Yan
first_author: Renjun Xu
year: 2026
introduces:
  - "[[agent-skills]]"
  - "[[skill-acquisition]]"
tags:
  - 2026
  - survey
status: complete
importance: high
---

# Agent Skills for LLMs: Architecture, Acquisition, Security, and the Path Forward

**Source file**: [[agent-skills-survey-xu-2026]]
**arXiv**: [arXiv:2602.12430](https://arxiv.org/abs/2602.12430) (v3, 17 Feb 2026, cs.MA)
**Authors**: Renjun Xu (corresponding, rux@zju.edu.cn), Yang Yan
**Affiliation**: [[zhejiang-university|Zhejiang University]], China
**Companion repository**: [scienceaix/agentskills](https://github.com/scienceaix/agentskills) ("awesome agent skills" resource collection)
**Year**: 2026

## Summary

This is **the first — and, as of this ingest, the only — dedicated academic survey of the agent-skills paradigm**. Where excellent surveys already exist on LLM agents broadly, on tool use, and on GUI agents, none had specifically examined the *skill abstraction layer* that emerged as a unifying architectural primitive after [[anthropic|Anthropic]]'s October 2025 launch of [[agent-skills|Agent Skills]]. Xu and Yan position the work explicitly against that gap and provide the field's first consolidated taxonomy. For the wiki this page is the **academic anchor** for [[agent-skills]] and [[skill-acquisition]]: it grounds concepts the wiki has been tracking from product announcements and blog posts in a citable, peer-style synthesis.

The survey defines a **skill** as a *"composable package of instructions, code, and resources that agents load on demand"* — a self-contained directory containing a structured instruction file ([[skill-md-format|SKILL.md]]), optional scripts, reference documents, and assets, that the agent discovers, loads, and follows when a relevant task arises. The framing is deliberately architectural: it contrasts skills against three prior paradigms of capability extension — prompt engineering (2022–2023; ephemeral, non-modular), [[tool-use|tool use]] / [[function-calling|function calling]] (2023–2024; atomic, execute-and-return), and now *skill engineering* (2025–present; higher-order bundles of procedural knowledge). The defining distinction from tools: **tools execute and return a result, whereas skills prepare the agent to solve a problem** by injecting procedural knowledge, modifying execution context and tool permissions, and enabling progressive disclosure. The whole narrative is built around the **transition from monolithic language models to modular, skill-equipped agents** — encoding procedural knowledge in loadable packages rather than baking it into model weights via fine-tuning.

The field is organized along **four axes**: (i) **architectural foundations** — the [[skill-md-format|SKILL.md]] specification, three-level progressive context loading, and the complementary roles of skills and [[mcp|MCP]] (skills supply "what to do," MCP supplies "how to connect"); (ii) **skill acquisition** — a four-way taxonomy spanning human authoring, [[reinforcement-learning|reinforcement learning]] with skill libraries (SAGE), autonomous skill discovery (SEAgent), structured skill bases (CUA-Skill), and compositional synthesis (Agentic Proposing); (iii) **deployment at scale** — the computer-use-agent ([[computer-use-agent|CUA]]) stack, [[gui-agent|GUI]] grounding advances, and benchmark progress on [[osworld|OSWorld]] and [[swe-bench|SWE-bench]]; and (iv) **security** — synthesis of three concurrent empirical studies (notably a finding that **26.1% of community-contributed skills contain at least one vulnerability**) motivating the survey's original **Skill Trust and Lifecycle Governance Framework**.

The survey's intellectual lineage is explicit. It credits [[voyager|Voyager]] as the antecedent skill library for embodied agents (LLM-generated programs stored and composed in Minecraft), CREATOR and "LLMs as Tool Makers" for self-created tools, and Toolformer for self-taught tool use — while drawing the key contrast that the *Agent Skills* paradigm emphasizes **human-authored, portable, governed** packages built for production deployment across heterogeneous platforms, rather than model-generated skills in constrained research environments. Across four months Anthropic's `anthropics/skills` repository accumulated 62,000+ GitHub stars, partner skills (Atlassian, Figma, Canva, Stripe, Notion) entered a curated directory, and structurally identical architectures were independently adopted by other frontier providers — a "rapid convergence" the survey reads as industry standardization of procedural-expertise packaging.

## The Survey's Taxonomy (Section Structure)

The paper organizes the field across its four axes plus background and a research agenda:

### §2 Background and Scope — From Prompt Engineering to Skill Engineering
Three paradigms of capability extension: **prompt engineering** (2022–2023, ephemeral) → **tool use / function calling** (2023–2024, atomic execute-and-return) → **skill engineering** (2025–present, higher-order procedural bundles). Relationship to prior work: [[voyager|Voyager]], CREATOR, LLMs-as-Tool-Makers, Toolformer as antecedents, distinguished by the production-grade, human-authored, portable, governed nature of Agent Skills.

### §3 Architectural Foundations
- **§3.1 The [[skill-md-format|SKILL.md]] specification** — a directory with a SKILL.md whose YAML frontmatter declares `name` + `description`.
- **Three-level progressive disclosure** (the defining architectural innovation):
  - **L1 — Metadata** (always loaded): YAML frontmatter, ~30 tokens/skill, preloaded into the system prompt — "a table of contents."
  - **L2 — Instructions** (loaded on trigger): the full SKILL.md body, 200–2k tokens — "the chapter content."
  - **L3 — Resources** (dynamic, on-demand, unbounded cost): scripts, reference docs, assets, loaded only if L2 calls them — "the technical appendices."
  - Framed as *"putting together an onboarding guide for a new hire"* (quoting Zhang, Lazuka & Murag).
- **§3.2 Skill execution lifecycle** — a two-phase trigger: (1) instructions/resources injected as a hidden meta-message; (2) execution context modified (pre-approved tools, file permissions activated). Skills modify the agent's *preparation*, not its output directly.
- **§3.3 The agentic stack: Skills and [[mcp|MCP]]** — orthogonal, complementary layers (Table 1). Skills = procedural knowledge, directory w/ SKILL.md, loaded by the agent on trigger, modifies context + permissions, filesystem-based. MCP = tool connectivity, server w/ endpoints (JSON-RPC 2.0; primitives: tools, resources, prompts), loaded by client on config, session-based.
- **§3.4 Advanced tool use integration** — Anthropic's Nov 2025 features: Tool Search Tool (−85% token overhead), Programmatic Tool Calling (79.5%→88.1% accuracy on Opus 4.5), Tool Learning.

### §4 Skill Acquisition and Learning — four/five modalities (Table 2)
- **§4.1 Human-authored skills** — direct authoring; a `skill-creator` meta-skill in [[claude-code|Claude Code]] scaffolds new skills; enterprise/partner-directory curation pipeline (62k+ GitHub stars).
- **§4.2 [[reinforcement-learning|RL]] with skill libraries — SAGE** (Skill Augmented GRPO for self-Evolution): "Sequential Rollout" preserves skills across chains of similar tasks; a Skill-integrated Reward credits reusable skill creation. On AppWorld: 72.0% TGC, +8.9% abs. over baseline GRPO, −26% steps, −59% tokens.
- **§4.3 Autonomous skill discovery — SEAgent**: World State Model + Curriculum Generator + specialist-to-generalist training; on five novel OSWorld apps, success 11.3%→34.5% (+23.2 pts over UI-TARS).
- **§4.4 Structured skill bases — CUA-Skill**: human computer-use expertise encoded as parameterized execution/composition graphs (typed parameters, preconditions, composability); 57.5% SOTA on WindowsAgentArena.
- **§4.5 Compositional skill synthesis — Agentic Proposing**: dynamic composition of modular reasoning skills; a 30B solver reaches 91.6% on AIME 2025.
- **§4.6 Skill compilation (multi-agent → single-agent)** — Li 2026: multi-agent systems can be "compiled" into single-agent skill libraries with lower token/latency, but skill-selection accuracy exhibits a **phase transition** beyond a critical library size.

### §5 Deployment: The Computer-Use Agent ([[computer-use-agent|CUA]]) Stack
- **§5.1 [[gui-agent|GUI]] agent architectures** — [[ui-tars-2|UI-TARS / UI-TARS-2]] (47.5% OSWorld, 73.3% AndroidWorld), Agent S2 (Mixture-of-Grounding), OpenCUA / AgentNet (NeurIPS 2025 Spotlight; OpenCUA-72B 45.0% on OSWorld-Verified).
- **§5.2 GUI grounding advances** — UGround (ICLR 2025 Oral; 10M elements), Jedi (4M examples; OSWorld 5%→27%), Yuan et al. (7B model 47.3% ScreenSpot-Pro, beating 72B UI-TARS with 3k samples), GUI-Actor (coordinate-free grounding).
- **§5.3 Benchmark landscape** (Table 3) — [[osworld|OSWorld]] climbed from single digits (early 2024) to *beyond human-level* (72.6% vs 72.36% human) by Dec 2025; harder settings (ScreenSpot-Pro, OS-Marathon long-horizon, CoAct-1 hybrid GUI-code) still expose gaps. [[swe-bench|SWE-bench]] Verified: 79.2% (Claude Opus 4.6).

### §6 Security of Agent Skills (the first consolidated treatment)
- **§6.1 [[prompt-injection|Prompt injection]] via skills** (Schmotz et al. 2025) — "trivially simple" injections via malicious SKILL.md/scripts; system guardrails bypassed when a benign "Don't ask again" approval carries over to related harmful actions. Exploits the implicit-trust model: loaded skill instructions are treated as authoritative context.
- **§6.2 Vulnerabilities at scale** (Liu et al. 2026a) — SkillScan analysis of 31,132 of 42,447 collected skills: **26.1% contain ≥1 vulnerability** across 14 patterns / 4 categories (prompt injection; data exfiltration 13.3%; privilege escalation 11.8%; supply-chain). Script-bundling skills are **2.12× more likely** to be vulnerable (OR=2.12, p<0.001); 5.2% show high-severity malicious-intent patterns.
- **§6.3 Confirmed malicious skills** (Liu et al. 2026b) — behavioral verification of 98,380 skills; 157 confirmed malicious with 632 vulnerabilities; two archetypes — *Data Thieves* (credential exfiltration via supply chain) and *Agent Hijackers* (instruction manipulation). One industrialized actor accounts for 54.1% of confirmed cases via templated brand impersonation.
- **§6.4 Skill Trust and Lifecycle Governance Framework** (the survey's original contribution) — three components:
  - **Verification gates G1–G4**: G1 static analysis → G2 LLM-based semantic intent-mismatch classification → G3 behavioral-sandbox side-effect audit → G4 permission-manifest validation (declared vs observed capability).
  - **Trust tiers T1–T4** (graduated, least-privilege, mapped to provenance): T1 unvetted/community = instructions-only, full isolation, no tools; T2 community-reviewed = read-only, no code execution, user confirm; T3 org-vetted = declared tools + scoped file access, no network; T4 vendor-certified = full capability. **T1/T2 are never granted script execution** (directly addressing the 2.12× script-bundling risk).
  - **Lifecycle trust evolution**: runtime monitoring promotes clean skills, demotes/revokes anomalous ones — a reputation system analogous to package ecosystems.
  - **Architecture-aware mapping**: governance maps directly onto the three progressive-disclosure levels (L1 metadata exposed at T1; L2 instructions at T2+; L3 scripts require T3/T4).

### §7 Open Challenges and Research Agenda — seven challenges
1. **Cross-platform portability** — skills authored for [[claude|Claude]] implicitly depend on Claude-specific capabilities; needs a universal runtime or skill compilation.
2. **Skill selection at scale** — the phase transition (Li 2026) makes routing combinatorially hard as libraries reach hundreds/thousands of skills.
3. **Skill composition and orchestration** — conflict resolution, resource sharing, failure recovery for multi-skill workflows remain underdeveloped.
4. **Capability-based permission models** — replace implicit trust with explicit per-skill capability declarations/grants.
5. **Skill verification and testing** — skills lack the unit-test/CI tooling of software packages; automated "does what it claims and nothing more" verification is open.
6. **Continual skill learning without catastrophic forgetting** — Shenfeld et al. 2026 find self-distillation promising; interaction between dynamically loaded skills and base capabilities is poorly understood.
7. **Evaluation methodology** — need metrics for skill reusability, composability, and maintainability rather than just task completion.

### §8 Discussion / §9 Conclusion
The paradigm is a shift from **monolithic intelligence to modular expertise** — skills as a digital analogue of organizational SOPs, open standards creating network effects, but security challenges (the 26.1% rate) reflecting the openness-vs-safety tension of every software ecosystem. A key research direction: bridge acquisition and deployment by making *learned* skills (SAGE/SEAgent, currently model-internal) **externalizable as portable, auditable artifacts** like human-authored SKILL.md files.

## Key Points

- **First and only dedicated academic survey of the agent-skills paradigm** — explicitly carved out against prior surveys on LLM agents, tool use, and GUI agents, all of which omit the skill abstraction layer. Serves as the wiki's citable academic anchor for [[agent-skills]].
- **Skill definition** — *"composable packages of instructions, code, and resources that agents load on demand"*: a directory with a [[skill-md-format|SKILL.md]] plus optional scripts/resources, discovered and followed on demand. **A skill is not a model or a prompt template.**
- **Skills vs. tools (the load-bearing distinction)** — tools *execute and return a result*; skills *prepare the agent to solve a problem* by injecting procedural knowledge and modifying execution context + permissions before the agent responds.
- **Three-paradigm evolution** — prompt engineering → tool use/function calling → skill engineering, each a higher-order, more durable/modular form of capability extension.
- **Three-level progressive disclosure is the defining architecture** — L1 metadata (~30 tok, always loaded) / L2 instructions (200–2k tok, on trigger) / L3 resources (unbounded, on demand); minimizes context cost while keeping arbitrarily deep procedural knowledge reachable.
- **Skills and [[mcp|MCP]] are orthogonal, complementary layers** — skills supply *what to do* (procedural intelligence), MCP supplies *how to connect* (tool connectivity); together they form the emerging agentic stack.
- **Four-modality skill-acquisition taxonomy** — human authoring; [[reinforcement-learning|RL]] with skill libraries (SAGE: +8.9% on AppWorld, −59% tokens); autonomous discovery (SEAgent: 11.3%→34.5% on OSWorld); structured skill bases (CUA-Skill: 57.5% on WindowsAgentArena); compositional synthesis (Agentic Proposing: 91.6% on AIME 2025).
- **A phase transition limits single-agent skill libraries** (Li 2026) — multi-agent systems compile into single-agent skill libraries, but selection accuracy collapses past a critical library size — a fundamental scaling limit on how many skills one agent can manage.
- **First consolidated security analysis** — three concurrent empirical studies show skills are a novel, high-trust attack surface: 26.1% of 42,447 community skills carry ≥1 vulnerability; script-bundling skills are 2.12× riskier; confirmed malicious skills exist at industrialized scale (one actor = 54.1% of cases).
- **Original Skill Trust and Lifecycle Governance Framework** — four verification gates (G1–G4) → four graduated trust tiers (T1–T4) → least-privilege deployment permissions, architecture-aware (tiers map to the L1/L2/L3 disclosure levels) and dynamic (runtime promotion/demotion).
- **Seven open challenges** define the frontier: cross-platform portability, skill selection at scale, composition/orchestration, capability-based permissions, verification/testing, continual learning without forgetting, and skill-quality evaluation methodology.
- **Bridging acquisition and deployment** is the framed grand challenge — making learned, model-internal skills externalizable as portable, auditable artifacts.

## Entities Mentioned

- [[zhejiang-university]] — author affiliation (Renjun Xu, Yang Yan)
- [[anthropic]] — originator of the Agent Skills standard (Oct 2025 launch; Dec 2025 open standard) and [[mcp|MCP]]
- [[claude]] / [[claude-code]] — primary deployment surface; host of the `skill-creator` meta-skill and the `anthropics/skills` directory

## Concepts Discussed

- [[agent-skills]] — the canonical concept this survey academically anchors
- [[skill-acquisition]] — the four-modality acquisition taxonomy (the survey's §4)
- [[skill-md-format]] — the SKILL.md specification (frontmatter + body + bundled resources)
- [[skill-library]] — the persistent store of reusable skills (SAGE, CUA-Skill, Voyager lineage)
- [[mcp]] — the complementary connectivity layer of the agentic stack
- [[voyager]] — the antecedent embodied skill library the survey credits as a precursor
- [[reinforcement-learning]] — the basis for SAGE and SEAgent skill learning
- [[self-improving-agent]] — SAGE/SEAgent self-evolution; the "externalize learned skills" research direction
- [[code-as-harness]] — the broader "code as operational substrate" view; skills as reusable code modules in the harness
- [[computer-use-agent]] / [[gui-agent]] — the primary deployment domain (§5)
- [[osworld]], [[swe-bench]], [[ui-tars-2]] — benchmarks/agents anchoring the deployment analysis
- [[tool-use]], [[function-calling]] — the prior paradigms skills supersede
- [[prompt-injection]] — the core skill security threat (Schmotz et al.); plus data exfiltration, privilege escalation, and supply-chain attack categories (no dedicated pages yet)

## Notable Quotes

> "The transition from monolithic language models to modular, skill-equipped agents marks a defining shift in how large language models are deployed in practice. Rather than encoding all procedural knowledge within model weights, agent skills—composable packages of instructions, code, and resources that agents load on demand—enable dynamic capability extension without retraining."

> "The distinction from traditional tools is architectural: tools execute and return results, whereas skills prepare the agent to solve a problem by injecting procedural knowledge, modifying execution context, and enabling progressive disclosure of information."

> "The agent skills paradigm represents a shift from monolithic intelligence to modular expertise. ... The skill ecosystem is currently in its 'pre-governance' phase, and the decisions made in the coming months about verification pipelines, permission models, and trust hierarchies will shape its trajectory for years."

## References

_Original source: `sources/agent-skills-survey-xu-2026.md`_
_arXiv: [arXiv:2602.12430](https://arxiv.org/abs/2602.12430) (v3, 2026-02-18)_
_Companion: [scienceaix/agentskills](https://github.com/scienceaix/agentskills)_

Key works synthesized (selected from the survey's 36 references):
- Zhang, Lazuka & Murag (2025) — Anthropic "Equipping agents for the real world with agent skills" → wiki: [[anthropic-agent-skills-2025]]
- Anthropic (2024) — Model Context Protocol → [[mcp]]
- Wang et al. (2023) — Voyager [arXiv:2305.16291] → [[voyager]]
- Wang et al. (2025) — SAGE: RL for self-improving agent with skill library [arXiv:2512.17102]
- Sun et al. (2025) — SEAgent: self-evolving computer-use agent [arXiv:2508.04700]
- Chen et al. (2026) — CUA-Skill [arXiv:2601.21123]
- Jiao et al. (2026) — Agentic Proposing [arXiv:2602.03279]
- Li (2026) — When single-agent with skills replace multi-agent systems [arXiv:2601.04748]
- Qin et al. (2025) — UI-TARS [arXiv:2501.12326] → [[ui-tars-2]]
- Schmotz, Abdelnabi & Andriushchenko (2025) — Agent Skills enable trivially simple prompt injections [arXiv:2510.26328]
- Liu et al. (2026a) — Agent Skills in the Wild: vulnerabilities at scale [arXiv:2601.10338]
- Liu et al. (2026b) — Malicious Agent Skills in the Wild [arXiv:2602.06547]
