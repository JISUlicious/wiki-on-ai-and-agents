---
title: RLAIF (RL from AI Feedback)
type: concept
created: 2026-05-14
updated: 2026-05-14
sources:
  - bai-constitutional-ai-2022.md
builds-on: "[[rlhf]]"
status: complete
importance: high
---

# RLAIF (RL from AI Feedback)

RLAIF replaces the **human** preference labels in [[rlhf]] with **AI-generated** preference labels. An evaluator LM (often the same base model with a specialized prompt or constitution) compares two candidate responses and picks which is better; the preferences become training data for a reward model and subsequent policy optimization. Introduced as part of [[constitutional-ai-bai-2022|Constitutional AI]] (Bai et al. 2022) and named explicitly there.

## Why use AI feedback

- **Scalability**: humans are expensive and slow. AI feedback can be generated in bulk.
- **Consistency**: AI judges are more consistent across examples than panels of human raters (though they can also be systematically biased in shared ways).
- **Scalable oversight**: as model capabilities outpace what typical humans can confidently judge, AI judges provide a way to extend oversight to harder cases (with the meta-level direction coming from a written constitution).

## The CAI recipe (canonical RLAIF)

1. **Sample candidate response pairs** from the model on adversarial / contested prompts.
2. **AI judge**: prompt an evaluator LM with the prompt, both responses, and the constitution. Ask it to pick the better response.
3. **Preference model**: train on the AI-judged pairs with Bradley-Terry loss, exactly as in [[rlhf]].
4. **RL**: optimize the policy against the preference model with PPO (or modern alternatives like [[dpo]]).

## Vs. RLHF

| | [[rlhf]] | RLAIF |
|---|---|---|
| Preference source | Human raters | AI evaluator LM |
| Cost | High (labor-intensive) | Low (compute-only) |
| Consistency | Moderate (rater disagreement) | High (within a single evaluator) |
| Bias | Aggregate human bias | Evaluator-model-specific bias |
| Updatability | Slow (re-recruit raters) | Fast (re-prompt the evaluator) |

In practice, modern post-training stacks **mix** human and AI feedback: humans for the most important / contested cases; AI for scale.

## Caveats

- **Evaluator bias propagates**: if the AI judge is systematically biased (e.g., prefers longer answers, or pro-AI framings), the trained policy inherits those biases.
- **Constitution-dependent**: AI judges follow the written constitution; getting the constitution wrong = scaled-up production of misaligned preferences.
- **No free lunch on capability**: the AI judge has to actually be competent at evaluating the dimension of interest. RLAIF on math problems works iff the judge can do math better than the policy.

## References

- [[constitutional-ai-bai-2022]]
