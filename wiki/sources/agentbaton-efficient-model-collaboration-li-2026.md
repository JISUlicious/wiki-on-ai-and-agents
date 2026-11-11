---
title: "AgentBaton: Efficient Model Collaboration in Agentic Tasks"
type: source
created: 2026-11-10
updated: 2026-11-10
sources:
  - agentbaton-efficient-model-collaboration-li-2026.md
arxiv_id: "2610.35959"
venue: arXiv preprint (cs.MA)
authors:
  - Junlong Li
  - Jindong Jiang
  - Yonggan Fu
  - Jiarui Yao
  - Hanrong Ye
  - Wonmin Byeon
  - Peter Belcak
  - Hao Zhang
  - Pavlo Molchanov
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# AgentBaton: Efficient Model Collaboration in Agentic Tasks

**Source:** Junlong Li, Jindong Jiang, Yonggan Fu, Jiarui Yao, Hanrong Ye, Wonmin Byeon, Peter Belcak, Hao Zhang, Pavlo Molchanov ([[nvidia|NVIDIA]]; HKUST; [[uiuc|UIUC]]) — [arXiv:2610.35959](https://arxiv.org/abs/2610.35959), cs.MA, v1 25 Oct 2026. Preprint.

## Summary

Routing a whole task to one model up front fails for long-horizon agents, because what a task needs changes as the workspace changes. The paper instead studies **one-time handoffs inside a single trajectory** between a capable cloud model and an efficient model. The capable models are GPT-5.6 Sol, Kimi K3, GLM-5.3 and others. The efficient models are Qwen3.8-27B running locally, and GPT-5.6 Luna. The benchmarks are Terminal-Bench 2.1 and DeepSWE v1.1, run in mini-SWE-agent.

The two directions behave differently. **Efficient → capable** handoffs usually do worse than running the capable model alone, sometimes at higher cost. **Capable → efficient** handoffs at the right moment keep, or even improve, accuracy at a fraction of the cost. The right moment is after the capable model has produced a substantive implementation and got concrete feedback, so what remains is localized repair or validation.

## Key points

- **Context transfer.** The history can be passed in-context, or as a `.jsonl` logfile the receiver inspects with shell tools. Logfiles clearly help efficient→capable handoffs. For capable→efficient the better choice depends on the model.
- **Prompting does not work.** Told to hand off when ready, Kimi K3 never did (0 of 113 tasks), GLM-5.3 did on 6 of 113, and GPT-5.6 Sol always did but still cost $3.75 against $5.32 for running alone.
- **AgentBaton** is a Qwen 3.5 2B detector that reads the trajectory after each turn and outputs HANDOFF_READY or HANDOFF_NOT_READY. It is trained on 4,000 SWE-rebench V2 trajectories from five capable models, with handoff points annotated retrospectively by Codex.
- **Results.** Across 11 model pairs it cuts **API cost by 65.9% on average** while largely keeping accuracy. It gains up to +8.8 points over capable-only, with four of five capable models beating their own baseline on DeepSWE when handing to Luna. It extends the accuracy–cost Pareto frontier across reasoning-effort settings.
- **Against routers.** A query-level router picked the cheap model for only one task. AgentBaton costs 46.7–86.6% less than the per-turn MTRouter at matched or better accuracy, which supports the claim that frequent switching piles up trajectory errors.

## Caveats

- Locally deployable models are **counted at zero API cost**, which flatters the savings for Qwen handoffs. Hardware and energy costs are excluded.
- There is a single scaffold (mini-SWE-agent), and the handoff labels come from an LLM annotator.

## Concepts & entities

- [[learned-orchestration]] · [[agentic-workflow-patterns]] · [[multi-agent]] — handoff timing as a learned policy.
- [[reasoning-effort]] · [[inference-time-compute]] · [[edge-inference]] — the cost–accuracy frontier.
- [[context-engineering]] · [[context-compaction]] — logfile vs in-context transfer.
- [[terminal-bench]] · [[swe-bench]] · [[llm-agent]].
- [[nvidia]] · [[uiuc]] · [[dgx-spark]] · [[qwen]] · [[gpt-6-astra]] · [[openai-codex]].

## References

- [arXiv:2610.35959](https://arxiv.org/abs/2610.35959)
