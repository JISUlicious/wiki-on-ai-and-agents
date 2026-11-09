---
title: "Decision API worked example: request → prompt → label read → answer"
type: query
created: 2026-11-09
updated: 2026-11-09
sources:
  - ollama-decision-models-2026-10-13.md
  - decision-api-server-implementations-2026-10-12.md
status: complete
importance: medium
---

# Decision API worked example: request → prompt → label read → answer

**Question:** Find, or build, a real example showing how a decision / System One API turns a request into a prompt and the prompt into an answer.

Not run live: no decision server was installed on the machine. The example combines a **real** request/response pair from Ollama's docs with the **actual prompt-building code** in SGLang and Ollama. Labels below: **real** (from a source), **from code** (traced in source code), **illustrative** (constructed). Background: [[decision-api-internals-prompt-output-confidence]] and [[decision-api-server-implementations]].

## 1. The real request and response (Ollama + Nimble)

From Ollama's decision-model docs, using [[nimble|Nimble]] on [[ollama|Ollama]] (`POST /v1/systemone`). **Real.**

```json
{
  "model": "nimble",
  "state": { "ticket": "I was charged twice. Please refund the extra payment." },
  "questions": {
    "team":    { "type": "choice", "instructions": "Which team should handle this ticket?",
                 "criteria": { "billing": "Payments and refunds",
                               "technical": "Bugs and integrations",
                               "other": "None of the above" } },
    "refund":  { "type": "noul",  "instructions": "Does the customer explicitly ask for a refund?" },
    "urgency": { "type": "score", "instructions": "How urgent is this ticket?",
                 "criteria": ["Routine", "Soon", "Urgent"] }
  }
}
```

```json
{
  "answers": {
    "team":    { "type": "choice", "choice": "billing",
                 "probabilities": {"billing": 0.985, "technical": 0.012, "other": 0.003},
                 "confidence": 0.922 },
    "refund":  { "type": "noul", "noul": 0.997 },
    "urgency": { "type": "score", "score": 0.815,
                 "legend": {"0": "Routine", "1": "Soon", "2": "Urgent"},
                 "probabilities": {"0": 0.378, "1": 0.429, "2": 0.193},
                 "confidence": 0.046 }
  },
  "usage": { "input_tokens": 841, "output_tokens": 4 }
}
```

`output_tokens: 4`: no text is generated. Those tokens were used internally to read the answers.

## 2. One prompt per question

### Path A: SGLang and vLLM (generic wording, any chat model)

From SGLang's `_render_question()` (**from code**). The state becomes compact JSON, then a blank line, then the question block. For `team`:

```text
{"ticket":"I was charged twice. Please refund the extra payment."}

Question: Which team should handle this ticket?
A: billing - Payments and refunds
B: technical - Bugs and integrations
C: other - None of the above
Answer with the letter of one option only.
```

The user message is wrapped in the model's chat template with thinking forced off. **Illustrative**, for a Qwen3-style template:

```text
<|im_start|>user
{"ticket":"I was charged twice. ..."}

Question: Which team should handle this ticket?
...
Answer with the letter of one option only.<|im_end|>
<|im_start|>assistant
<think>

</think>

▮  ← answer position: the next-token distribution is read here
```

The other two questions:

```text
refund  →  Is the following true? Does the customer explicitly ask for a refund?
           Answer with yes or no only.                       labels: yes / no
urgency →  Question: How urgent is this ticket?
           0: Routine
           1: Soon
           2: Urgent
           Answer with the number of one level only.         labels: 0 / 1 / 2
```

[[vllm|vLLM]]'s wording is similar. For yes/no it keeps `Question:` instead of "Is the following true?" and always prints the bare `yes`/`no` labels.

### Path B: Ollama + Nimble (the model's training format)

From Ollama's `decisionPrompts()` (**from code**). The state and **all** questions go into one JSON object. Each prompt then names the single field to answer:

```text
{"context": "{\"ticket\": \"I was charged twice. ...\"}", "schema": [ …team…, …refund…, …urgency… ]}

Requested field: "team"
```

The code confirms the `context` and `schema` keys, the `Requested field:` line, and that `<` and `>` are escaped. It shows option strings such as `"billing: Payments and refunds"`, `"level 0: Routine"` and `"false: no, the statement does not hold"`, but **not** the full layout of each schema entry. Ollama labels every question type with letters, including noul (`A` = false, `B` = true).

## 3. The read: one pass, softmax over the labels only

Before reading, every server checks that each label (`A`, `B`, `C`) is **exactly one token** after the prompt, and refuses the model otherwise. For `team`, only three next-token logits matter:

```text
p = softmax(logit("A"), logit("B"), logit("C")) = (0.985, 0.012, 0.003)  →  choice = "billing"
```

How each server gets those logits:

| Server | Method |
|---|---|
| [[sglang\|SGLang]] | Prefill only, zero new tokens; logprobs of the label IDs |
| vLLM | One greedy decode step (discarded) with `logprob_token_ids` |
| Ollama (llama.cpp) | +100 logit bias on the candidates, top-k = number of candidates, one token |
| Ollama (MLX) | Unembeds only the candidate rows |

**The real response's numbers, recomputed:**
- `urgency.score` = 0·0.378 + 1·0.429 + 2·0.193 = **0.815**. This is an expected level, not a label: "Soon" is the most likely single level, but the 0.378 on "Routine" pulls the score down.
- `team.confidence` = 1 − H(p)/ln 3 = **0.922** (Ollama's normalised entropy).
- `urgency.confidence` = **0.046**. The distribution is nearly flat, so confidence is close to zero although `score` looks precise.

## 4. Same probabilities, three "confidence" values

Taking `team`'s distribution (0.985 / 0.012 / 0.003) on each server:

| Server | Formula | `confidence` |
|---|---|---|
| Ollama | `1 − H/ln K` | **0.922** (real) |
| SGLang `/v1/systemone` | `(n·max(q) − 1)/(n − 1)` | **0.978** (computed) |
| vLLM | `p_top × label_mass` | depends on label mass; **0.955** if the labels hold 97% of the full-vocabulary probability (illustrative) |

A threshold such as "act above 0.95" would pass on SGLang, fail on Ollama, and depend on label mass on vLLM. Pin the server version and calibrate thresholds on your own labelled data ([[calibration]]).

## Related

- [[system-one-model]] · [[logit-scoring]] · [[jev]] · [[openai-decisions-api]] · [[nimble]] · [[ollama]] · [[sglang]] · [[vllm]]
- [[decision-api-internals-prompt-output-confidence]] · [[decision-api-server-implementations]] · [[decision-api-vs-llm-and-server-implementations]]

## References

- [[ollama-decision-models-2026]]: the request/response pair (Ollama docs)
- [[decision-api-server-implementations-2026]]: SGLang `_render_question`, `_encode_labels`; Ollama `decisionPrompts`, `Scorer`
