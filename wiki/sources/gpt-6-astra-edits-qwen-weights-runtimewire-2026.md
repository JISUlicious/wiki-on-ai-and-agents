---
title: "GPT-6 Astra edited 0.016% of Qwen's weights and its math score went up 6x"
type: source
created: 2026-11-03
updated: 2026-11-03
sources:
  - gpt-6-astra-edits-qwen-weights-runtimewire-2026.md
authors:
  - Ryan Merket
year: 2026
venue: Runtime Wire (news)
status: complete
importance: low
tags:
  - 2026
  - news
---

# GPT-6 Astra edited 0.016% of Qwen's weights (Runtime Wire, 2026)

**Ryan Merket**, Runtime Wire, published 2 Nov 2026 ([article](https://runtimewire.com/article/gpt-6-astra-edits-qwen-weights-weight-space-programming)). A news report on [[weight-space-programming-takehi-2026|Takehi et al.'s blog post]], based on that post and the authors' X thread. The primary source is the blog post; this page records what the article adds and where it differs.

## What it reports

- MIT CSAIL researchers (Rikiya Takehi, Jayden Teoh, Alex Zhang, [[omar-khattab|Omar Khattab]]) say [[gpt-6-astra|GPT-6 Astra]] traced [[qwen|Qwen3.5-0.8B]]'s reasoning loops to layers 15 and 19 and removed them by editing 131,072 weights (0.016%), with no gradient descent.
- MATH-500 rose from 7.9% to 48.1%, GSM8K from 13.8% to 67.1% and IFEval from 21.0% to 62.3%, all with 50 forward passes. At 10 or 25 passes the share of answers hitting the token limit stayed near 87%.
- It summarises the smaller experiments (naming four games from weights, fixing a stuck car policy, writing MinAtar players from scratch).
- It stresses the [[gemma|Gemma 3 270M]] negative result: the method removed a defect that hid an ability and did not add a new one.

## Its framing

- **Headline vs. body.** The headline's "math score went up 6x" is accurate arithmetic (7.9 → 48.1), but the body itself says the gain "says as much about how badly the 0.8B model was looping" as about added skill.
- **"No independent replication."** The article says none accompanies the post. The blog post does report that a Hugging Face staff member rebuilt the edited model from the logs and re-ran its evaluation. That re-checks one released edit, not the method, so the two statements are compatible but the article omits it.
- **Practical reading.** If reproduced, small open models could be repaired with targeted weight edits instead of retraining. The near-term value is diagnosing and patching, not adding capability.

The article contains no information beyond the blog post on the method itself.

## References

- [Runtime Wire article](https://runtimewire.com/article/gpt-6-astra-edits-qwen-weights-weight-space-programming)
- [[weight-space-programming-takehi-2026]] (primary source)
