---
title: "Google ADK × System One / Decisions: integration status (2026-10-12)"
type: source
created: 2026-10-12
updated: 2026-10-12
sources:
  - adk-system-one-integration-status-2026-10-12.md
year: 2026
venue: "GitHub (google/adk-python PR #7235); litellm 1.105.0 and google-adk 2.11.0 package sources"
url: https://github.com/google/adk-python/pull/7235
tags:
  - 2026
status: complete
importance: medium
---

# Google ADK × System One / Decisions: integration status

**Source**: `sources/adk-system-one-integration-status-2026-10-12.md`. It holds the PR metadata and body from the GitHub API, plus file listings and code excerpts from the `litellm` 1.105.0 and `google-adk` 2.11.0 packages.

**Provenance:** the findings were first reported by the **analysis-docs** session while researching ADK for Jisu. This wiki re-checked the PR and both packages on 2026-10-12.

## Findings

**1. No official ADK support. One open community PR.**
- [google/adk-python PR #7235](https://github.com/google/adk-python/pull/7235): "feat(integrations): add TypeSafe AI (Jev) integration".
  - Opened 2026-09-22 by AymanKhan9.
  - **Open, not merged**, 21 files, `mergeable_state: dirty`.
  - The requester reported a failing CLA check. As of 2026-10-12 the `cla/google` status shows **success**, so CLA is no longer the blocker. The conflict state (`dirty`) and the lack of a merge remain.
- **The design argument**, from the PR body: Jev, "given a state and a fixed set of typed questions (Noul/Choice/Score)", returns typed answers with probabilities, "so it doesn't fit `BaseLlm`'s contract". So it is proposed as `google.adk.integrations.typesafe`, alongside the existing langchain, crewai and mongodb integrations (all three exist in 2.11.0's `integrations/`), rather than as a model class.
- **Proposed components:**
  - `TypesafeClassifier`, which wraps `typesafe_sdk`'s `system_one()`;
  - `TypesafeClassifierTool`, which exposes one fixed question as an agent tool;
  - `create_model_router_callback` and `create_tool_gate_callback`, `before_model` / `before_tool` callback factories for **model routing** and **gating risky tools**.
- **Released ADK has nothing:** GitHub code search of the default branch returns 0 hits for both "systemone" and "jev" (verified). The only reference is this PR.

**2. LiteLLM supports decision models, but ADK's LiteLLM bridge can't reach them.**
- **litellm 1.105.0 ships:**
  - `litellm/decisions` and `litellm/systemone` modules, star-imported into `litellm`;
  - provider enums `TYPESAFE` and `CLOUDFLARE`;
  - decision transforms for **TypeSafe, Perplexity and OpenRouter**;
  - in its proxy, **guardrail hooks** for `typesafe` and a generic `decision_model`. This is a guardrail use of decision models: screening requests through a typed decision.
  - **This is a LiteLLM proxy feature, not ADK support.** It matters for ADK only in a setup where a LiteLLM proxy sits in front of ADK.
- **Runtime checks** (requester's, **re-run and confirmed** here with litellm 1.105.0 on Python 3.12):
  - `litellm.decisions` / `adecisions` / `systemone` / `asystemone` exist;
  - `get_llm_provider("typesafe/jev-latest")` returns `('jev-latest', 'typesafe')` and `cloudflare/clef` returns `('clef', 'cloudflare')`;
  - `ProviderConfigManager.get_provider_chat_config(..., LlmProviders.TYPESAFE)` returns **None**, so the TypeSafe provider has no chat config. Decision models are reachable only through the dedicated `decisions` / `systemone` functions.
- **ADK 2.11.0 uses an explicit allowlist of 16 model-name patterns for LiteLLM:** `openai/`, `azure/`, `azure_ai/`, `groq/`, `anthropic/`, `bedrock/`, `ollama/(?!gemma3)` (Gemma 3 goes to a separate class), `ollama_chat/`, `together_ai/`, `vertex_ai/`, `mistral/`, `deepseek/`, `fireworks_ai/`, `cohere/`, `databricks/`, `ai21/`.
  - The list sits in `models/__init__.py`'s lazy-provider registry and also in `lite_llm.py`. `ollama/gemma3*` is routed to a separate `Gemma3Ollama` class.
  - `typesafe/` is **not** on it, and `lite_llm.py` imports `acompletion` (completion calls), not `decisions` (verified).

> [!success] Resolved: the allowlist
> **Resolved 2026-10-12:** a first version of this page "corrected" the requester's list, claiming `ollama/` and `ai21/` were absent. That was this wiki's extraction error: its pattern matched only letters and underscores, so it missed the lookahead entry and `ai21`. The requester's 16-entry list is right. The conclusion is unchanged.

**3. ADK's `tools/model_consult/` is not a connection point.**
- Verified in the 2.11.0 source: `resolve_advisor_llm(model: str | BaseLlm) -> BaseLlm` resolves through `LLMRegistry`, calls `generate_content_async`, and the docstring reads "resolved through ADK's model registry… so any model ADK supports can advise".
- Decision models don't fit `BaseLlm`, so they can't be plugged in this way. This narrows the wiki's earlier "cannot judge" to **"not applicable"**.

## Not verified

- PR #7235's code quality and behaviour (not run).
- The Perplexity Decisions API's primary documentation.

## Why it matters

- **Interface mismatch.** Agent frameworks model every model as a chat-completion `BaseLlm`. Decision models don't fit that contract, so frameworks integrate them as **tools or callbacks** (routing, gating) rather than as models. That is the same harness-side placement the wiki argues in [[system-one-models-and-dual-process-thinking]].
- **The plumbing arrives before the frameworks do.** LiteLLM already routes decisions to several vendors and uses them as guardrails, while ADK has only an unmerged community PR.

## Concepts & entities

[[google-adk]] · [[system-one-model]] · [[jev]] · [[openai-decisions-api]] · [[typesafe-ai]] · [[guardrailing]]
