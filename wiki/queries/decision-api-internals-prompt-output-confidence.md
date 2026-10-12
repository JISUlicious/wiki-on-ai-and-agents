---
title: "Decision API internals: prompt construction, output-token control, confidence (SGLang, vLLM, Ollama)"
type: query
created: 2026-10-12
updated: 2026-10-12
sources:
  - decision-api-server-implementations-2026-10-12.md
status: complete
importance: high
---

# Decision API internals: prompt, output token, confidence

**Question:** Deep dive into how each server builds the input prompt, how it handles and controls the output token, and where it computes confidence.

The answer is traced through source code: SGLang v0.5.21, vLLM `main` (2026-10-12) and Ollama v0.40.3. Excerpts are in `sources/decision-api-server-implementations-2026-10-12.md`. The overview is in [[decision-api-server-implementations-2026]] and [[decision-api-server-implementations]].

## The pipeline all three share

```
request (state + typed questions)
  └─ per question: render ONE prompt  →  tokenize  →  check every label is exactly one token
       └─ one forward pass, read the next-token distribution at the answer position
            └─ keep only the label tokens' logits/logprobs  →  softmax over labels
                 └─ choice = argmax · score = Σ i·p_i · noul = p(yes)  →  confidence
```

None of them reads several questions from one causal pass: **one question = one prompt row**. The rows share the state prefix, which the KV or prefix cache makes cheap. See [[logit-scoring]].

---

## 1. How the input prompt is built

| | SGLang | vLLM (main) | Ollama |
|---|---|---|---|
| **Template source** | The model tokenizer's **Jinja chat template** via `apply_chat_template([{user}], add_generation_prompt=True)`. Built-in non-Jinja templates are refused. | The server's chat renderer (`preprocess_chat`): the model's template or `--chat-template` | The **model's own system prompt**, plus its native chat template (`ApplyChatTemplate`) or Ollama's renderer |
| **Message layout** | One **user** message: `input`, a blank line, then the question block | Optional **system** message (`instructions`), then a **user** message: `state`, a blank line, the question block | **Model-specific training format**, see below |
| **Question block** (generic) | `Question: …` / `A: name - description` … / `Answer with the letter of one option only.` For scores: `0: level`… and "Answer with the number of one level only." For yes/no: `Is the following true? …` and "Answer with yes or no only." | Similar wording: `Question: …`, `A: name - desc`, "Answer with the letter of one option only." / "… number of one level only.". Noul keeps the `Question:` line (no "Is the following true?" lead-in), lists `yes`/`no` (with descriptions if given), and ends "Answer with yes or no only." | **Nimble:** `{"context": state, "schema": [fields]}` + `\n\nRequested field: "<name>"`, with `<` `>` escaped. **Tev1:** `{"state", "question", "options":[{label,key,description}]}`. Clef and Strands have their own encoders. All JSON is serialized like Python `json.dumps(ensure_ascii=False)`. |
| **Labels** | choice `A–Z` (two-letter `AA…` up to 255, if the tokenizer allows); score `0–9`; yes/no `yes`/`no` | choice `A–Z` (max 26); score `0–9`; noul `yes`/`no` | **Letters for every type:** choice `A, B, C…`; score levels `A, B, …`; noul `A`=false/"No", `B`=true/"Yes". 2–26 candidates (Tev1 ≤ 24). |
| **Structured state** | Objects and arrays → compact JSON (`separators=(",",":")`) | `state_text()` | Compact JSON (`json.Compact`); strings used as-is |
| **Thinking** | The template's reasoning toggle is forced **off**. Templates that always reason, or that leave a reasoning block open at the answer position, are refused (checked by rendering a sentinel reply). | `enable_thinking: false`; a request that turns it on is refused ("a read is the reply's first token") | `Think: false` when rendering |
| **Prompt pinning** | `PROMPT_FORMAT_VERSION = 1`; a request can pin `prompt_format_version` | — | Fixed per model encoding (`decision.type` metadata) |
| **Length guard** | Refuses when the prompt ≥ context length, because auto-truncation would cut off the answer position | engine limits | `MaxTokens = context length` passed to the scorer |

**The design difference:**
- **SGLang and vLLM** *write their own generic question wording* and rely on a capable chat model to follow it.
- **Ollama** *replays the exact prompt each fine-tuned decision model was trained on*.

That is why Ollama only serves tagged decision models, while the other two serve general chat models.

---

## 2. How the output token is handled and controlled

**The label-token check** (all three): tokenize `prompt` and `prompt + label`. The label is valid only if it adds **exactly one new, distinct token** and leaves the prompt's tokens unchanged.
- SGLang: `label_token_id`. To keep this cheap, it re-tokenizes only the text after the last added special token (`label_context`).
- vLLM: `reply_label_ids`, computed **once** on a probe prompt (`"x"`). Each real prompt must end with the same tail, or the request is refused.
- Ollama: tokenizes `prompt + candidate` and `candidate` alone, and requires "exactly one ordinary token".

**How the distribution is obtained:**

| | Engine call | Does it decode? | What comes back |
|---|---|---|---|
| **SGLang** | `GenerateReqInput(input_ids, token_ids_logprob=<label ids>, return_logprob=True, sampling_params={"max_new_tokens": 0}, logprob_start_len=-1)` through `score_prompts` (the `/v1/score` machinery) | **No.** Prefill only; zero new tokens. | Logprobs of exactly the label IDs at the next position (`output_token_ids_logprobs`), over the **full vocabulary**, then a softmax over the labels **÷ temperature** (centred first, against overflow) |
| **vLLM** | `SamplingParams(max_tokens=1, temperature=0.0, logprob_token_ids=<label ids>)`; requires `--logprobs-mode raw_logprobs \| processed_logprobs`; at most 128 IDs | **Yes, one greedy token.** It is discarded, but used for `argmax_is_label`. | The sampler computes `log_softmax` over the **raw logits, before temperature or penalties** (`raw_logprobs`), and `gather_specific_token_logprobs` picks the label IDs. `label_softmax()` then normalizes over the labels. |
| **Ollama, llama.cpp runner** | llama-server `/completion` with `n_predict: 1`, `logit_bias: +100` on every candidate (retried at +1000), `samplers: [top_k, temperature]`, `top_k = #candidates`, `temperature: 1`, `post_sampling_probs: true`, `n_probs = #candidates` | One token is generated and discarded | The bias lifts all candidates by the same amount, so their **relative** logits, and hence the softmax among them, are unchanged. The post-sampling probabilities are the softmax over the candidates' original logits. **The full-vocabulary mass is lost.** It errors if another token still outranks a candidate. |
| **Ollama, MLX runner** | Prefill forward only; take the **last position's hidden state**; **unembed only the candidate rows** (`UnembedCandidates`, optionally at higher precision; otherwise slice the full unembedding) | **No** | Raw candidate logits; the softmax happens in `decision.Answer` |

**Speed controls:**
- **SGLang** scores all questions in one `score_prompts` batch, tokenizing lazily and yielding to the event loop between questions.
- **vLLM** issues one request per question, run concurrently. They share the state prefix, so with prefix caching it is "prefilled once".
- **Ollama's llama.cpp runner** "primes" the shared prefix so llama-server checkpoints exactly at its end. The code comment says "with 8 questions, scoring is about 3x faster".
- **Ollama's MLX runner** keeps a scoring-only prefix cache with snapshots at shared boundaries.

**Off-label signal:**
- SGLang returns `label_mass`, the sum of the labels' full-vocabulary probabilities.
- vLLM returns `label_mass` plus `argmax_is_label` (did greedy decoding pick a label?).
- Ollama returns none: the logit-bias trick (llama.cpp) and the candidate-only unembedding (MLX) never see the rest of the vocabulary.

---

## 3. Where confidence is calculated

| Server | File and function | Formula | Notes |
|---|---|---|---|
| **SGLang `/v1/systemone`** | `entrypoints/systemone/serving.py` → `_answer`, `_choice_confidence`, `_score_confidence` | choice: `(n·max(q) − 1)/(n − 1)` (distance of the top option above uniform). score: `1 − Σ q_i·|i − top| / uniform_spread`. `q` = the probabilities renormalized over the labels. | The PR says these follow "the formulas the System One adapter publishes". Noul returns no confidence, only `noul = p(yes)`. |
| **SGLang `/v1/decisions`** | `openai/serving_decisions.py` → `_build_answer` | **No confidence field.** It returns `probabilities` + `label_mass`. | The docs: threshold on the top probability, validated on your own data |
| **vLLM** | `structured_decisions/question_types.py` → each `QuestionType.answer()` | `probs[top] × label_mass` for choice, score **and noul** | Penalizes reads where the model wanted a non-label token. It is the chosen label's full-vocabulary probability. |
| **Ollama** | `decision/systemone.go` → `(*Compiled).Answer` | `clamp(1 − H(p)/ln K, 0, 1)`, normalized entropy, for choice and score. Noul returns only `p(true)`. | Softmax computed here, from runner logits, with max-subtraction; non-finite logits are rejected |

**Consequences:**
1. **The same model and question give three different "confidence" values** on the three servers. Thresholds don't transfer.
2. **Only vLLM folds in off-label mass.** SGLang exposes it separately; Ollama can't.
3. **All three are statistics of the label distribution, not calibrated correctness probabilities.** SGLang's docs say so explicitly. Calibrate on your own labelled data ([[calibration]]).

## Gotchas found in the code

- **Temperature applies to different things.** SGLang's `/v1/decisions` temperature divides the label logits (its System One route uses 1.0). vLLM's `temperature=0` only picks the greedy token; probabilities come from raw logprobs. Ollama's llama.cpp path fixes `temperature: 1`.
- **The yes/no wording differs.**
  - SGLang and vLLM put `yes`/`no` in the prompt. vLLM's `reply_line` always prints the bare `yes`/`no` label, because without it "models answer 'Yes' or 'No' instead" (code comment; bug #60862). The alternative fix, #60870, counting "Yes"/"No" as labels, was closed unmerged.
  - Ollama uses letters `A`/`B` for false/true.
  - So the same yes/no question is literally a different prompt.
- **Option-order dependence.** Every server maps options to labels in request order; none shuffles. Two-letter labels past 26 (SGLang) "can make answers depend on option order" (#41208).
- **Nondeterminism.** SGLang notes answers "can vary with batch composition and prefix caching".

## References

- [[decision-api-server-implementations-2026]] · [[decision-api-server-implementations]]
- [[logit-scoring]] · [[calibration]] · [[system-one-model]] · [[sglang]] · [[vllm]] · [[ollama]]
