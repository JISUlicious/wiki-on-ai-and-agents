---
title: Self-Consistency
type: concept
created: 2026-05-14
updated: 2026-05-14
sources:
  - wang-self-consistency-2022.md
builds-on: "[[chain-of-thought-prompting]]"
status: complete
importance: high
---

# Self-Consistency

Self-consistency is an inference-time decoding strategy that replaces greedy decoding in [[chain-of-thought-prompting|CoT]] with **sample-and-marginalize**: generate many diverse reasoning paths at non-zero temperature, then take the **majority vote** across the final answers. Introduced by [[self-consistency-wang-2022|Wang et al. (2022)]] at [[google-research|Google Brain]].

## The procedure

```
Sample N (~40) chains-of-thought from the LLM at temperature 0.5–0.7.
Extract the final answer from each chain.
Return the modal answer (majority vote).
```

The bet is that a complex reasoning problem admits **multiple correct reasoning paths leading to the same answer**, and if the model is competent-but-noisy, the correct answer should be the most-popular outcome across diverse samples. Wrong reasoning produces inconsistent answers; right reasoning converges.

## Why it works

- **Marginalization over latent paths** is statistically motivated — closer to the true posterior over answers than a single greedy decode.
- **Diversity averaging** mitigates the brittleness of greedy decoding on hard problems.
- **No model changes**: works on any LLM, wraps any CoT prompt.

## Results

On 8-shot CoT prompts evaluated with PaLM 540B and InstructGPT:
- **GSM8K**: +17.9% over greedy CoT
- **SVAMP**: +11.0%
- **AQuA-RAT**: +12.2%
- **StrategyQA**: +6.4%
- **ARC-Challenge**: +3.9%

## Inference-time scaling lineage

Self-consistency was one of the **first general-purpose inference-time-scaling techniques** in the LLM era. The pattern — "trade more inference compute for more accuracy via sampling + aggregation" — propagated:

- **Best-of-N** sampling with a reward model.
- **Process reward models** (Lightman et al. 2023): supervise intermediate steps, not just final answers.
- **[[tree-of-thoughts|Tree of Thoughts]]** (Yao 2023): explicit search over reasoning paths.
- **LATS** (Zhou 2023): LLM agent tree search with self-evaluation.
- **OpenAI o1** (Sep 2024) and subsequent reasoning models: internalize inference-time search via RL training. The "thinking tokens" the o1-family produce are the descendant of self-consistency's sampled chains, but baked into the model's policy rather than added at decoding time.

## Cost

Roughly **N×** the inference compute of greedy CoT. For N=40, that's 40× more tokens. Generally worth it for hard reasoning tasks; overkill for tasks the model can already do reliably.

## References

- [[self-consistency-wang-2022]]
