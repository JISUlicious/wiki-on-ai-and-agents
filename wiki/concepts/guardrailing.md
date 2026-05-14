---
title: Guardrailing
type: concept
created: 2026-05-14
updated: 2026-05-14
sources:
  - rlhf-christiano-2017.md
  - constitutional-ai-bai-2022.md
  - red-teaming-ganguli-2022.md
  - indirect-prompt-injection-greshake-2023.md
status: complete
importance: high
---

# Guardrailing

Guardrailing is the **external-mechanism** practice of restricting an LLM's behavior at the boundaries of the system: at the input, at the output, at the tool surface, or via training-time preference signals. It is the **counterpart to [[steering]]**, which intervenes inside the model's forward pass.

The wiki's existing guardrailing literature spans several layers:

## Training-time guardrails (post-training)

- **[[rlhf]]** — Reinforcement Learning from Human Feedback ([[rlhf-christiano-2017|Christiano et al. 2017]] → [[instructgpt-ouyang-2022|InstructGPT 2022]]). Bake in helpfulness, harmlessness, honesty via preference optimization. The dominant mainstream alignment recipe.
- **[[constitutional-ai]]** — RLHF with AI-generated preferences against a written constitution ([[constitutional-ai-bai-2022|Bai et al. 2022]]). Anthropic's variant.
- **[[rlaif]]** — Generalized AI-feedback alignment training.
- **[[dpo]]** — Simplified preference-optimization loss ([[dpo-rafailov-2023|Rafailov 2023]]).
- **[[instruction-tuning]]** — Supervised fine-tuning on (instruction, response) pairs.

## Inference-time guardrails (input / output)

- **Input filtering / sanitization** — strip or modify prompts before they reach the model.
- **Output classifiers** — separate models inspect generations and block or redact.
- **System-prompt boundaries** — `<system>` vs. `<user>` tag separation, with the model trained to respect the boundary. Partial mitigation against [[prompt-injection]].
- **Structured tool-use APIs** — restrict tool calls to a declared JSON schema; reject malformed actions. (OpenAI function calling, Anthropic tool use, MCP.)

## Capability / deployment guardrails

- **Sandboxing** — restrict file-system, network, process access for code the agent runs. (Claude Code's `@anthropic-ai/sandbox-runtime`, OpenClaw's tiered Docker, Hermes Agent's 7 backends.)
- **Permission systems** — rules + classifiers + hooks that decide what the agent can do. ([[claude-code|Claude Code]]'s layered permission stack.)
- **Capability containment** — narrow the toolbox; require human-in-the-loop for sensitive actions.

## Auditing / red-teaming

- **[[red-teaming]]** — Adversarial probing pre-deployment ([[red-teaming-ganguli-2022|Ganguli 2022]]).
- **Continuous evaluation** — TruthfulQA, MMLU, dangerous-capability evals (METR, ARC Evals, AISI).
- **Output monitoring** — log and audit deployed model outputs.

## Threats that drive guardrailing

- **[[hallucination]]** — confident fabrication.
- **[[prompt-injection]]** / **[[indirect-prompt-injection]]** — adversarial inputs hijacking the model.
- **[[sycophancy]]** — pleasing-the-user at truth's expense.
- **[[reward-hacking]]** — optimizing proxy reward without satisfying intent.
- **[[agentic-misalignment]]** — agents pursuing goals contrary to operator intent.

## How guardrailing and [[steering]] differ

| | Guardrailing | [[steering]] |
|---|---|---|
| **Surface of intervention** | External (inputs, outputs, system prompts, permissions, training-time preferences) | Internal (activations, attention heads, SAE features) |
| **Operates by** | Restricting, refusing, sandboxing, filtering, preference-shaping | Adding / subtracting vectors in the residual stream |
| **Controls** | What is *allowed* to happen | What is *likely* to happen at the algorithmic level |
| **Data requirements** | Thousands of preferences (RLHF) or substantial engineering (filters, sandbox) | Few hundred contrastive examples (CAA) or unsupervised (SAE) |
| **Inspectability** | Log audits, output samples | Activation values, feature dictionaries |
| **Failure modes** | Brittle to adversarial inputs; opaque refusals | Can degrade fluency; off-target correlations |
| **Composability with the other** | Yes | Yes |

## How they're combined in 2026

Modern alignment stacks **layer both surfaces**. [[claude|Claude]]'s training is [[constitutional-ai|CAI]] (guardrailing); its [[emotion-concepts-anthropic-2026|emotion-vector interventions]] are steering. [[claude-code|Claude Code]]'s permission/sandbox is guardrailing; the underlying model is RLHF/CAI-trained guardrailing-on-the-inside. The convergence: **guardrails at the boundaries, steering at the substrate.**

## References

- [[rlhf-christiano-2017]]
- [[instructgpt-ouyang-2022]]
- [[constitutional-ai-bai-2022]]
- [[red-teaming-ganguli-2022]]
- [[indirect-prompt-injection-greshake-2023]]
