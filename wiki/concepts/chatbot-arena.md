---
title: Chatbot Arena
type: concept
created: 2026-10-06
updated: 2026-10-06
sources:
  - judging-llm-as-a-judge-zheng-2023.md
status: draft
importance: medium
---

# Chatbot Arena

A **crowdsourced evaluation platform**. Users chat with two anonymous models side by side and vote for the better answer, and the votes are aggregated into Elo-style ratings. Run by LMSYS ([[uc-berkeley|UC Berkeley]] and collaborators). It is described and validated in [[judging-llm-as-a-judge-zheng-2023|Zheng et al. (Jun 2023)]], which released 30K Arena conversations with human preferences.

## Role

- **Ground truth for judges.** Arena votes were one of the two human-preference sets used to show that GPT-4 judges agree with people about 80% of the time, the same as people agree with each other. See [[llm-as-a-judge]].
- **Leaderboard.** In 2023–2025 the Arena leaderboard became a widely watched public ranking of chat models (general knowledge; later criticism of it is not ingested here).

## Caveats

- Votes reflect user *preference*, which can reward style and verbosity over correctness.
- The prompt distribution is whatever users happen to ask.

## Related

[[mt-bench]] · [[llm-as-a-judge]] · [[benchmark]] · [[rlhf]]

## References

- [[judging-llm-as-a-judge-zheng-2023]]
