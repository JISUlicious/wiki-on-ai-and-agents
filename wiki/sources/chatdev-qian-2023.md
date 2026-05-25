---
title: "ChatDev: Communicative Agents for Software Development"
type: source
created: 2026-05-25
updated: 2026-05-25
sources:
  - chatdev-qian-2023.md
arxiv_id: "2307.07924"
venue: ACL 2024
authors:
  - Chen Qian
  - Wei Liu
  - Hongzhang Liu
  - Nuo Chen
  - Yufan Dang
  - Jiahao Li
  - Cheng Yang
  - Weize Chen
  - Yusheng Su
  - Xin Cong
  - Juyuan Xu
  - Dahai Li
  - Zhiyuan Liu
  - Maosong Sun
first_author: Chen Qian
year: 2023
introduces:
  - "[[chatdev]]"
tags:
  - 2023
status: complete
importance: high
---

# ChatDev: Communicative Agents for Software Development

**Source**: `sources/chatdev-qian-2023.md` (extracted from arXiv `2307.07924v5`)
**arXiv**: [2307.07924](https://arxiv.org/abs/2307.07924)
**Venue**: ACL 2024 (long paper)
**Authors**: Chen Qian, Wei Liu, Hongzhang Liu, Nuo Chen, Yufan Dang, Jiahao Li, Cheng Yang, Weize Chen, Yusheng Su, Xin Cong, Juyuan Xu, Dahai Li, Zhiyuan Liu, Maosong Sun
**Affiliations**: Tsinghua University (most authors), University of Sydney (H. Liu), BUPT (C. Yang), Modelbest Inc. (D. Li)

## Summary

ChatDev is a **chat-powered, waterfall-style multi-agent framework for end-to-end software development**, in which role-specialized LLM agents — CEO, CTO, programmer, reviewer, tester — collaborate through structured multi-turn dialogue to take a one-line product request all the way to a runnable codebase. The pipeline mirrors the classical software waterfall: **design → coding (write + complete) → testing (code review + system testing)**, with each phase decomposed into subtasks executed as **two-agent instructor/assistant dialogues**. Two ideas hold it together: a **chat chain** that tells agents *what* to communicate (chain-structured workflow with short-term phase memory and long-term cross-phase memory of distilled solutions only), and **communicative dehallucination**, a role-reversal communication pattern that tells agents *how* to communicate — the assistant proactively asks for more specific details before committing to a response, which materially reduces coding hallucinations like unexecutable or placeholder code.

ChatDev was the **multi-agent-coding prototype** that, together with [[metagpt-hong-2023|MetaGPT]] and [[autogen-wu-2023|AutoGen]], defined the early multi-agent code-collaboration literature in mid-to-late 2023. Where [[metagpt-hong-2023|MetaGPT]] enforces structure through SOP-style document handoffs and [[autogen-wu-2023|AutoGen]] provides a general conversational framework, ChatDev's distinctive contribution is the **chain-of-phases workflow with role-reversal dehallucination** and an empirical claim that *language itself* — natural language for design, programming language for debugging — is the unifying bridge for multi-agent collaboration. The [[code-as-agent-harness-ning-2026|code-as-agent-harness survey (Ning 2026)]] cites ChatDev 12 times across its §4 on Multi-Agent Orchestration, treating it as a canonical reference point alongside [[metagpt]] and [[autogen]]. On the authors' SRDD benchmark (1,200 software requirement prompts across Education / Work / Life / Game / Creation), ChatDev beats both GPT-Engineer (single-agent) and MetaGPT on Completeness, Executability, Consistency and an integrated Quality score (0.3953 vs. MetaGPT's 0.1523), and wins ~57% of pairwise comparisons against MetaGPT under GPT-4 judging (88% under human judging).

## Key Points

- **Waterfall pipeline**: 3 phases (Design, Coding, Testing) decomposed into 5 subtasks (design, code writing, code completion, code review, system testing). The waterfall structure is borrowed from classical software engineering and used as the agentic backbone.
- **Role-specialized agents**: CEO, CTO, programmer, reviewer, tester — plus an instructor/assistant role pair instantiated per subtask via the **inception prompting** mechanism (system prompts that cover overview, role, tools, protocol, termination, and forbidden behaviors).
- **Chat chain**: formally, `C = ⟨P¹, P², …⟩`, each phase `Pⁱ = ⟨T¹, T², …⟩`, each subtask `Tʲ = τ·C(I, A)` extracted from a multi-turn dialogue between instructor `I` and assistant `A` until consensus. The chain bridges natural- and programming-language subtasks and gives a transparent intermediate view of the development process.
- **Two-tier memory**: short-term memory carries full dialogue within a single phase; long-term memory carries **only the distilled solutions** across phases — avoiding context blow-up while preserving cross-phase continuity.
- **Communicative dehallucination (CDH)**: a deliberate role reversal where the assistant takes an instructor-like turn — proactively asking for the precise dependency name, class, or detail — before committing to a final answer. Pattern: `⟨I → A, ⟨A → I, I ⇝ A⟩↻, A ⇝ I⟩↻`. Ablation removing CDH drops Quality from 0.3953 to 0.3094.
- **Collaboration over shared program state**: agents cooperate by *talking about* (and revising) the code artifact, not by sharing arbitrary state — the chat-chain's distilled solutions are the bridge between phases.
- **Cost profile**: ChatDev runs ~148s per task on average (vs. 15.6s for single-agent GPT-Engineer, 154s for MetaGPT) using ~23K tokens — slower and pricier than single-agent baselines but still cheaper than MetaGPT while producing more files and a larger codebase.
- **Ablations**: removing role assignments collapses Quality to 0.2212 (the biggest single drop); halting the chain before testing leaves Quality at 0.3717. Roles and CDH are the two load-bearing mechanisms; the chain order matters but degrades gracefully.
- **Backbone**: ChatGPT-3.5 (temperature 0.2), Python 3.11.4 for execution feedback. No fine-tuning — the contribution is entirely in agent orchestration over a frozen LLM.
- **Dataset**: SRDD (Software Requirement Description Dataset) — 1,200 prompts across 5 areas × 40 subcategories × 30 prompts. Released alongside the paper.
- **Open source**: https://github.com/OpenBMB/ChatDev — became a widely-used reference implementation and a fixture of the OpenBMB ecosystem.

## Entities Mentioned

- Chen Qian — first author, Tsinghua University; corresponding contact `qianc62@gmail.com`.
- Zhiyuan Liu, Maosong Sun — corresponding senior authors, Tsinghua University.
- Tsinghua University — primary affiliation (12 of 14 authors).
- Modelbest Inc. — Beijing-based LLM startup (Dahai Li) tied to the OpenBMB ecosystem that hosts the ChatDev codebase.
- [[metagpt-hong-2023|MetaGPT]] — primary multi-agent baseline; ChatDev outperforms it on Quality 0.3953 vs. 0.1523.
- [[autogen-wu-2023|AutoGen]] — contemporaneous Microsoft multi-agent framework (not a direct baseline here, but the third pillar of the 2023 multi-agent-coding literature).
- GPT-Engineer (Osika 2023) — single-agent baseline ChatDev compares against.

## Concepts Discussed

- [[chatdev]] — the framework introduced by this paper
- [[metagpt]] — primary multi-agent comparison
- [[autogen]] — sibling multi-agent framework, contemporaneous
- [[code-as-harness]] — broader category this work fits into
- multi-agent orchestration; chat-chain workflow
- code generation; software engineering with LLMs
- agentic hallucination / coding hallucination and dehallucination via structured communication
- role-playing LLM agents; inception prompting

## Notable Quotes

> "We introduce ChatDev, a chat-powered software development framework in which specialized agents driven by large language models (LLMs) are guided in what to communicate (via chat chain) and how to communicate (via communicative dehallucination)." — Abstract

> "Their utilization of natural language is advantageous for system design, and communicating in programming language proves helpful in debugging. This paradigm demonstrates how linguistic communication facilitates multi-agent collaboration, establishing language as a unifying bridge for autonomous task-solving among LLM agents." — Abstract

> "By sharing only the solutions of each subtask rather than the entire communication history, ChatDev minimizes the risk of being overwhelmed by too much information, enhancing concentration on each task and encouraging more targeted cooperation, while simultaneously facilitating cross-phase context continuity." — §3.1, on the long-term memory design

> "Our communicative dehallucination mechanism features a deliberate 'role reversal', where the assistant takes on an instructor-like role, proactively seeking more specific information (e.g., the precise name of an external dependency and its related class) before delivering a conclusive response." — §3.2

## References

_Original source: `sources/chatdev-qian-2023.md`_
_Open-source: https://github.com/OpenBMB/ChatDev_
_Published: arXiv 2307.07924 (July 2023, v5 June 2024); ACL 2024 long paper._
