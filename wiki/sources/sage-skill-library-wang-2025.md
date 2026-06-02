---
title: "Reinforcement Learning for Self-Improving Agent with Skill Library (SAGE)"
type: source
created: 2026-06-02
updated: 2026-06-02
sources:
  - sage-skill-library-wang-2025.md
arxiv_id: "2512.17102"
authors:
  - Jiongxiao Wang
  - Qiaojing Yan
  - Yawei Wang
  - Yijun Tian
  - Soumya Smruti Mishra
  - Zhichao Xu
  - Megha Gandhi
  - Panpan Xu
  - Lin Lee Cheong
first_author: Jiongxiao Wang
year: 2025
introduces:
  - "[[skill-acquisition]]"
tags:
  - 2025
status: complete
importance: medium
---

# Reinforcement Learning for Self-Improving Agent with Skill Library (SAGE)

**Source:** `sources/sage-skill-library-wang-2025.md`
**arXiv:** [arXiv:2512.17102](https://arxiv.org/abs/2512.17102) (v2 [cs.AI], rev. 10 Mar 2026)
**Code:** [github.com/amazon-science/SAGE](https://github.com/amazon-science/SAGE)
**Authors:** Jiongxiao Wang, Qiaojing Yan, Yawei Wang, Yijun Tian, Soumya Smruti Mishra, Zhichao Xu, Megha Gandhi, Panpan Xu (corresponding), Lin Lee Cheong
**Affiliations:** University of Wisconsin–Madison; AWS Agentic AI ([[amazon]]). First author's work done during an internship at AWS Agentic AI.
**Year:** 2025

## Summary

SAGE (Skill Augmented GRPO for self-Evolution) is a reinforcement-learning framework that trains a [[self-improving-agent]] to accumulate reusable executable skills in a [[skill-library]] across a *chain of sequential tasks*. Prior skill-library agents — notably [[voyager]], Agent Workflow Memory, and Agent Skill Induction — generate skills by LLM prompting after a task completes, which makes skill quality hostage to the base model's instruction-following ability and decouples task execution from skill creation. SAGE instead folds skill generation directly into RL training. It extends [[grpo]] with two components: **Sequential Rollout**, which deploys the agent across a chain of similar tasks per rollout so that skills generated in earlier tasks are preserved in the library and made available to later ones (letting reward signal from successful skill *reuse* back-propagate to the skill *generation* in the earlier task); and a **Skill-integrated Reward** that adds a bonus, on top of the verifiable outcome reward, when a skill generated in the first task is successfully used to complete the second. Following DynaSaur, task solving and skill definition share a unified code format — the agent writes a programmatic function and immediately calls it — built on the [[codeact-wang-2024|CodeAct]] framework rather than the after-the-fact skill induction of earlier work.

This is the earliest entry in the 2025–26 RL-for-skill-libraries cluster and serves as a lineage anchor between [[voyager]] (the prompting-based skill-library ancestor) and later RL-trained skill systems such as [[skill1-shi-2026|Skill1]]. Evaluated on the [[appworld]] benchmark (750 tasks across 250 scenarios of three similar tasks each, well-suited to Sequential Rollout) with Qwen2.5-32B-Instruct as the base model and SFT on Claude 3.5 Sonnet V2 expert trajectories as a warm start, SAGE beats a no-skill-library GRPO baseline on Test Normal by **+8.9% Scenario Goal Completion (60.7 vs 51.8)** while cutting cost: **−26% interaction steps (12.1 vs 16.4)** and **−59% generated tokens (1,475 vs 3,613)**. The RL-trained open-source agent surpasses the Claude expert that produced its SFT data, and skill-usage analysis shows >2x success rate when SAGE-trained agents apply learned skills.

## Key Points

- **RL across sequential task chains, not single tasks.** Sequential Rollout runs the agent through a chain of similar tasks (this paper uses 2-example chains for cost; longer chains discussed in Appendix B). Skills built in task 1 enter the library and are usable in task 2, so reuse reward propagates back to generation.
- **Skill-integrated Reward.** Final reward = verifiable outcome reward + a bonus that fires only when a skill from task 1 is actually used (and both tasks succeed). A −1.0 penalty is applied to responses that emit no code and terminate, enforcing the code-per-interaction format.
- **GRPO extension.** Built on GRPO (Shao et al. 2024); following LOOP (Chen et al. 2025) it drops the KL penalty and does not normalize advantage by reward std. The novelty vs vanilla GRPO: expectation is taken over the task chain, and same-group generations for the second task come from *different* skill libraries.
- **Unified task-solving / skill-generation format.** Following DynaSaur (Nguyen et al. 2024), the agent generates a function then calls it, instead of defining skills only after task completion — avoids context blow-up and the execution/generation inconsistency of post-hoc induction. Built on [[codeact-wang-2024|CodeAct]].
- **Four skill-library operations.** Skill Usage, Skill Generation (define + immediately call), Skill Update (fix a failing skill and recall), and Skill Save (persist successful new/updated skills). Direct API calls allowed when a function is unnecessary.
- **Two-stage training.** Open-source models could not follow the skill-library agent prompt well enough for usable rollouts, so SFT on Claude 3.5 Sonnet V2 expert trajectories precedes SAGE RL. SFT alone does not beat the GRPO baseline; RL is what produces the self-improvement gains.
- **AppWorld headline (Test Normal vs baseline GRPO).** SGC 60.7 vs 51.8 (+8.9%); TGC 72.0 vs 69.2; Avg. Steps 12.1 vs 16.4 (−26%); Avg. Tokens 1,475 vs 3,613 (−59%). State-of-the-art on both Test-Normal and Test-Challenge among prior RL methods without a skill library; >3x SGC over prompting-based skill-library agents with <half the tokens.
- **Skill-usage analysis.** SAGE markedly raises both Skill Usage Rate and Success Skill Usage Rate over the base/SFT stages; SAGE-trained agents show >2x success when using learned skills, despite generating fewer skills than the untrained base model (quality over quantity).
- **Idealized retrieval.** Default evaluation uses "Same Scenario" skill scoping (provided scenario labels), removing the need for a retrieval model during rollout; additional retrieval methods analyzed for the realistic label-free setting.

## Entities Mentioned

- [[amazon]] — AWS Agentic AI, the work's industrial home (most co-authors); first author interned there.
- University of Wisconsin–Madison — first author's academic affiliation.
- [[claude-3-5-sonnet]] (Claude 3.5 Sonnet V2) — expert model used to generate high-quality SFT trajectories.
- Qwen2.5-32B-Instruct — open-source base model trained with SAGE.

## Concepts Discussed

- [[skill-acquisition]] — the core capability SAGE trains: learning to generate, validate, and reuse skills.
- [[skill-library]] — the accumulating store of executable skills; SAGE's central data structure.
- [[agent-skills]] — executable, reusable skill functions composed of action sequences.
- [[reinforcement-learning]] — the training paradigm; SAGE is an RL framework.
- [[grpo]] — the base RL algorithm SAGE extends (Sequential Rollout + Skill-integrated Reward).
- [[self-improving-agent]] — the agent class: continual improvement via skill accumulation across deployment.
- [[codeact-wang-2024|CodeAct]] — the code-as-action agent framework SAGE builds on.
- [[react]] — the baseline agent design (and AppWorld's reference agent) SAGE outperforms.
- [[appworld]] — the long-horizon tool-use benchmark; SGC metric rewards completing all similar tasks in a scenario.
- [[agentic-rl]] — broader category: RL for multi-turn interactive agents.
- [[voyager]] — the prompting-based skill-library ancestor (Minecraft); SAGE's lineage predecessor.
- [[skill1-shi-2026|Skill1]] — RL-skill-library sibling in the 2025–26 cluster.

## Notable Quotes

> "We introduce Skill Augmented GRPO for self-Evolution (SAGE), a novel RL framework that systematically incorporates skills into learning." — Abstract

> "As agents navigate through the task chain, skills generated from previous tasks accumulate in the library and become available for subsequent tasks." — Abstract

> "Rewards signal from the successful usage of skills in later tasks can be back-propagated to the skill generation in the previous tasks." — §3.2.2 Sequential Rollout

> "SAGE … achieves 8.9% higher Scenario Goal Completion while requiring 26% fewer interaction steps and generating 59% fewer tokens, substantially outperforming existing approaches in both accuracy and efficiency." — Abstract

> "Although our final results rely on SFT using expert experience data generated by Claude, our RL approach with the skill library enables open-source models to surpass the expert performance." — §4.2 Main Results

## References

_Original source: `sources/sage-skill-library-wang-2025.md`_
arXiv: <https://arxiv.org/abs/2512.17102>
Code: <https://github.com/amazon-science/SAGE>
