---
title: Constitutional AI
type: concept
created: 2026-05-14
updated: 2026-05-14
sources:
  - bai-constitutional-ai-2022.md
builds-on: "[[rlhf]]"
status: complete
importance: high
---

# Constitutional AI

Constitutional AI (CAI) is [[anthropic|Anthropic]]'s technique for training a harmless AI assistant **without human labels of harmful outputs**. Introduced by [[constitutional-ai-bai-2022|Bai et al. (Dec 2022)]]. The only human input is a written set of principles (a **constitution**); the model generates the rest of the training signal by critiquing and revising its own outputs against those principles.

## The two-phase pipeline

1. **Critique-Revision SFT**:
   - Sample model responses on prompts (often adversarial).
   - For each response, prompt the model to *critique* it against a randomly-chosen constitutional principle.
   - Prompt the model to *revise* its response based on the critique.
   - Fine-tune the original model on revised responses.

2. **[[rlaif|RL from AI Feedback (RLAIF)]]**:
   - Sample response pairs from the SFT model.
   - Use an evaluator LM to *choose* which response better satisfies the constitution.
   - Train a preference model on the AI-chosen pairs.
   - Run [[rlhf]]-style RL with this preference model as the reward.

## The constitution

A list of ~16 principles, e.g.:
- "Choose the response that is the most truthful, honest, and ethical."
- "Choose the response that is the most harmless and non-discriminatory."
- "Choose the response that least encourages illegal, unethical, or immoral activity."

Sources include the UN Universal Declaration of Human Rights, Apple's Terms of Service, DeepMind's Sparrow principles, and original Anthropic principles. The constitution is **explicit and inspectable** — a key argument for the method's transparency.

## Why it matters

- **Scalable oversight**: as AI capabilities outgrow what typical human labelers can confidently judge, AI feedback becomes a way to extend oversight.
- **Behavioral transparency**: critiques and AI-feedback rationales (often produced with [[chain-of-thought-prompting|CoT]]) make the alignment process inspectable.
- **Non-evasive harmlessness**: the trained model engages with harmful queries and *explains* its objections rather than refusing opaquely. This is a more useful kind of safety than blanket refusal.
- **The Claude lineage**: CAI is the alignment recipe behind [[claude|Claude]] from 2023 onward. Real-world Claude training uses extensions of this approach plus newer techniques.

## Variants and successors

- **RLAIF in the wild**: many open-weight models now use AI-judge-generated preference data for at least part of their post-training (Llama-3, Mistral, etc.).
- **Constitutional Pluralism** (Anthropic 2024+): different constitutions for different user contexts.
- **Iterative CAI**: re-do the pipeline with the latest model as both judge and student.

## CAI as a guardrail; steering as the complement

CAI is a **training-time [[guardrailing|guardrail]]** — it shapes the model's preferences via constitutional principles before deployment. The complementary inference-time approach is **[[steering]]**: directly modify the trained model's activations to suppress harmful directions or amplify aligned ones. The two are not in tension; they target different surfaces.

The [[emotion-concepts-anthropic-2026|2026 emotion-concepts paper]] makes the connection explicit: it shows that **CAI post-training measurably shifts the activation profile of emotion concepts** in Claude Sonnet 4.5 — moving the model toward lower-arousal, lower-valence emotion vectors. The safety effect of CAI training is partly implemented as something that *looks like* steering — except it's baked into weights rather than added at inference time. This convergence suggests CAI and explicit steering interventions are different views of a single underlying process.

Future variants are likely to **mix the surfaces**: CAI-style training + targeted inference-time steering on safety-relevant feature directions ([[sparse-autoencoder|SAE]]-identified or [[contrastive-activation-addition|CAA]]-derived) at deployment time.

## References

- [[constitutional-ai-bai-2022]]
- [[emotion-concepts-anthropic-2026]] — CAI post-training shifts measurable in activation space
- [[steering]] — complementary inference-time approach
