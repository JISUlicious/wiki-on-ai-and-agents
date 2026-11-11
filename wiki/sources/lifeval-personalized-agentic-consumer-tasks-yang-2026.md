---
title: "LifeVal: Evaluating AI Model Performance on Personalized Agentic Consumer Tasks"
type: source
created: 2026-11-10
updated: 2026-11-10
sources:
  - lifeval-personalized-agentic-consumer-tasks-yang-2026.md
arxiv_id: "2611.06687"
authors:
  - Jenny Yang
  - Elizabeth Proehl
  - Anjan Bharadwaj
  - Emily Fan
  - Charlotte Cole
  - James Kim
  - Noam Brown
  - Phoebe Thacker
  - Amelia Glaese
  - Tejal Patwardhan
year: 2026
venue: arXiv preprint
tags: [2026, paper]
status: complete
importance: medium
---

# LifeVal: Evaluating AI Model Performance on Personalized Agentic Consumer Tasks

**Source:** Jenny Yang, Elizabeth Proehl, Anjan Bharadwaj et al., Tejal Patwardhan ([[openai|OpenAI]]) — [arXiv:2611.06687](https://arxiv.org/abs/2611.06687), cs.AI, v1 4 Nov 2026. 16 pages. Surfaced via the NLP newsletter (2026-11-08).

## Summary

LifeVal is OpenAI's benchmark for the "digital superassistant": a personalized agent that acts across a user's digital life. Usage metrics only show tasks people already ask for, so LifeVal instead weights tasks by **what consumers say they would pay**. A willingness-to-pay survey of 3,837 US respondents (sampled to Census age/gender/ethnicity and CPS income distributions), cross-checked against the American Time Use Survey and the Multinational Time Use Study, yields 148 activity categories worth about **$1.7 trillion a year** in US willingness to pay. Examples are managing finances, moving, job search, childcare, taxes and government services.

For the highest-value categories, expert task writers (who had done the task themselves, with results seen over a year or more) wrote **169 tasks** grounded in their own de-identified [[chatgpt|ChatGPT]] conversation histories and memories, reference files and extra context. Each task has a rubric written by its author and a reinforcement-learning environment (RLE) with an offline web, so it is personal and subjective yet gradable.

## Key points

- **Five properties:** high value, high context, personalized, agentic (browsing, forms, transactions, artifacts) and objectively gradable given the context.
- **QC:** at least three human review rounds (peer, lead, final sign-off) plus a model loop that attempts the task, classifies failures (rubric, prompt/context, environment, model) and proposes fixes for humans to accept.
- **Grading:** a GPT-5.6 rubric grader that sees the backend state diff and checks claimed artifacts exist, plus a **hack grader** that zeroes any rollout that calls internal RLE APIs. On a 30-rollout subset, humans and the grader agree closely (mean 83.6% vs 84.2%, r = 0.864).
- **Models evaluated:** GPT-5.2, 5.4, 5.5, 5.6 and [[gpt-6-astra|GPT-6 Astra]]. The headline scores are in figures only; the text gives no per-model numbers.
- **Failure trends:** GPT-5.2 alone runs out of tokens. Later models improve with computer-use and connector training. GPT-6 Astra shows more alignment-boundary failures ("required action not completed", "reported inability to verify"). The most stable failure is incorrect information that ignored facts in personal conversations.
- **Personal history is under-used.** Win rates on rubric criteria that need past conversations are 33%, against 48% for memories and 56% for explicit task context. Moving the same facts into past conversations drops it to 42%. Rollouts search history with keyword grep, and apparent successes could be explained by information available elsewhere.
- **Limitations:** time-use data ends in 2021; the survey is US- and English-only; willingness to pay is self-reported; the web cache is not fully frozen.

## Concepts & entities

- [[agent-benchmark]] · [[agent-evaluation]] · [[computer-use-agent]] · [[osworld]] — consumer tasks in RL environments rather than the live web.
- [[long-term-memory]] · [[context-rot]] · [[long-context-llm]] — models fail to retrieve personal facts buried in long chat histories.
- [[reward-hacking]] · [[rl-environment-synthesis]] · [[llm-as-a-judge]] — hack grader, RLEs, rubric autograder.
- [[openai]] · [[chatgpt]] · [[gpt-6-astra]]

## References

- [arXiv:2611.06687](https://arxiv.org/abs/2611.06687)
