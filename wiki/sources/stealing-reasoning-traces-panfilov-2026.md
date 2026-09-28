---
title: "Stealing Reasoning Traces from Proprietary LLM APIs"
type: source
created: 2026-08-17
updated: 2026-08-17
sources: []
arxiv_id: "2608.09867"
year: 2026
authors:
  - Alexander Panfilov
  - David Schmotz
  - Ilia Shumailov
  - Luca Beurer-Kellner
  - Joachim Schaeffer
  - Ameya Prabhu
  - Jonas Geiping
  - Maksym Andriushchenko
tags:
  - 2026
  - paper
  - security
status: complete
importance: high
---

# Stealing Reasoning Traces from Proprietary LLM APIs

**Alexander Panfilov, David Schmotz, Ilia Shumailov, Luca Beurer-Kellner, Joachim Schaeffer, Ameya Prabhu, Jonas Geiping, Maksym Andriushchenko** (ELLIS Institute Tübingen, Max Planck Institute for Intelligent Systems, Tübingen AI Center / University of Tübingen, Snyk, MATS) — [arXiv:2608.09867](https://arxiv.org/abs/2608.09867), cs.CR / cs.AI / cs.LG, 10 Aug 2026.

## Summary

Frontier providers no longer show you the model's [[chain-of-thought]]. To keep multi-turn [[reasoning]] and agentic tool loops coherent without exposing it, they hand the client an **opaque encrypted reasoning block** and expect it back on subsequent requests (the pattern the [[responses-api]] and its Anthropic/Google equivalents standardized). The block is the provider's answer to two problems at once: anti-distillation and stateless serving.

The paper's finding is that these blocks are **not bound to the session, the user, or the model that produced them**. Within a single provider's ecosystem they are "fully compatible and interchangeable," and that portability is itself the vulnerability. The authors state the root cause as a *security asymmetry inside a model family*: the frontier sibling carries heavy refusal training and anti-distillation defenses, its cheap sibling does not, and a valid authenticated block moves freely across that gap. Nothing is broken cryptographically and the safeguarded model is never jailbroken directly — the weak sibling is simply a legitimate holder of the decryption capability and has far less reason to refuse.

Two adversary profiles follow mechanically. A **first-party attacker** generates their own blocks by querying the capable model normally, then replays them through the cheap decoder. A **third-party attacker** never touches the strong model at all: they collect blocks that *other people* published — agent trajectories committed to GitHub and [[huggingface|Hugging Face]] — and use cross-user compatibility to turn any compatible model into a decryption oracle for strangers' private reasoning.

Providers deployed mitigations after disclosure, and the paper's headline experiments are explicitly **no longer reproducible** as of publication.

## Threat model

- **Prerequisite for both profiles:** ordinary paid API access to *any* compatible "decoder" model inside the target provider's ecosystem. No privileged access, no key compromise, no model weights.
- **First-party:** attacker owns the source traces; goal is distillation and jailbreak-adjacent extraction of hidden reasoning from a model they are already allowed to query.
- **Third-party:** attacker owns nothing; goal is reading other users' reasoning out of blocks those users published without understanding what was inside them. This is the one that generalizes badly, because the victims are people who thought they were publishing an opaque blob.
- The paper is careful that the encrypted block is *authenticated* — the attack is a **replay/confused-deputy** problem, not a cipher break.

## Key points

**Cross-model compatibility (Table 1)** — measured over three ecosystems; every provider tested was affected:

| Provider | Weakest compatible decoder | Observed compatibility |
|---|---|---|
| Anthropic (Claude) | Haiku 4.5 | Traces from every tested model replay across the family; Fable 5 is the one exception (its traces are not accepted by the others) |
| OpenAI (GPT) | GPT-5.6 Luna | The GPT-5.6 series accepts traces from all earlier generations tested; earlier generations do **not** accept newer ones |
| Google (Gemini) | Gemini Robotics 1.6 | Fully interchangeable — every source × target cell compatible, including Flash Lite and the robotics model |

Gemini is the permissive extreme (a complete compatibility matrix); OpenAI is the most constrained, with generational one-way compatibility. Notably the weakest *accepting* model in each family is a cheap tier — the economics favor the attacker, since the oracle is the cheapest model on the menu.

**Extraction fidelity** — the authors evaluate on **120 Codeforces problems** per model, comparing thinking-token counts the API billed for the source generation against the API-reported token count of the reconstructed trace fed back as input. Their claim is that "the two counts track each other closely across all tested models." They report **no correlation coefficient and no percentage-agreement figure**, and they state the honest caveat directly: "In the absence of a ground-truth reasoning trace and the stochasticity of the generation process, we cannot guarantee that the extracted thoughts correspond exactly to a model's private reasoning." Treat this as strong evidence of high-fidelity reconstruction, not as a proven verbatim 1:1 recovery.

**Secret extraction in the wild** — the load-bearing empirical result, and the one that does not depend on fidelity being perfect:

| Quantity | Value |
|---|---|
| Public agent trajectories collected (GitHub + Hugging Face) | 6,708 |
| Reconstructed reasoning traces | 315,320 |
| Blocks containing ≥1 privacy leakage | 0.3% (1,028) |
| Trajectories leaking ≥1 real sensitive item | **4.9% (328)** |
| Total recovered | **367 PII artifacts, 182 credentials** |

Broken out for genuine (non-benchmark) user sessions: 62 distinct API keys, 33 passwords, 24 access tokens, 7 private keys, 30 personal emails, 6 non-localhost IP addresses, 130 names, 36 postal addresses. The sharpest number in the paper: **64 of the 704 artifacts recovered from genuine sessions are entirely absent from the visible chat history** — i.e. the reasoning channel leaks material that no amount of scrubbing the *visible* transcript would have caught. The per-block rate looks reassuring (0.3%) and the per-session rate does not (4.9%), because a single session touching a credential contaminates many blocks.

Cost of the third-party attack is trivial: decoding 10k traces at 12k-token input/output windows is estimated at **~$720** in nominal API spend. Total project spend was ~$30,000 in API credits.

**Four attack vectors** — the paper enumerates:

1. **Distillation** — circumventing anti-distillation protections; recovering the reasoning traces that a provider deliberately hides to prevent [[knowledge-distillation]] into a competitor's model.
2. **Secret extraction** — the public-corpus scan above; a privacy/data-exfiltration attack on third parties.
3. **Invisible [[prompt-injection]]** — a payload placed inside a block that the client, the logs, and human reviewers all see as ciphertext. Demonstrated qualitatively across a model pair with an [[llm-agent]] instructed to exfiltrate a file after every change. **No quantified success rate is reported** — proof-of-concept only.
4. **Jailbreaking** — surfacing hazardous content the model reasoned about while its visible answer stayed benign, demonstrated with a paraphrased HarmBench prompt. Again qualitative; **no success-rate numbers**.

Vector 3 is the structurally nastiest: it defeats the standard [[guardrailing]] assumption that inputs can be inspected. An encrypted block is un-inspectable *by design*, which makes it a perfect carrier for [[indirect-prompt-injection]] — the security control and the attack surface are the same field.

**Mitigations proposed** — defense in depth, roughly in order of how much they actually fix:

- **Server-side reasoning storage** — stop handing the client an encrypted payload at all. The only structural fix; the rest are containment.
- **Cryptographic contextual binding** — embed user and conversation identifiers inside the envelope so a block is only valid in the context that produced it. Directly kills cross-user and cross-session replay.
- **Cross-model isolation and velocity detection at the API gateway** — infrastructure guardrails against the replay pattern.
- **Provider-side revocation** — track and revoke compromised trace signatures.
- **Model-level defenses** — train models to recognize and refuse prompts designed to transcribe hidden reasoning.

The authors concede a structural limit: any model that can decrypt a block can in principle be coerced into acting as an oracle for it, so model-level refusal training is a mitigation, not a solution. Binding and server-side storage are the fixes that change the shape of the problem.

**Disclosure status** — disclosed before publication to the affected major API providers plus Microsoft and Hugging Face, with full technical details and preliminary scan findings. All providers acknowledged receipt, and the authors report they were subsequently **unable to launch the same attacks**; the paper notes its own Figure 1 results "are no longer reproducible" because of provider mitigations. The paper credits Green (2026) with disclosing the underlying trace-interchangeability behavior in **May 2026**, at which point providers reportedly did not acknowledge any security implications from side channels or replay. Recovered secrets were handled in an isolated environment and deleted after [[llm-as-a-judge]] classification and aggregate counting.

**Stated limitations** — (1) evaluation is pinned to specific API versions available as of early July 2026, and provider cryptography is proprietary and changes without announcement; (2) no ground truth for the reasoning traces, so exact-token faithfulness is unverifiable; (3) the wild-corpus scan is a preliminary sample, not an exhaustive audit; (4) the authors expect *private* datasets — local agent transcripts — to be considerably worse affected than the public corpora they could measure, which means 4.9% is plausibly a floor rather than an estimate.

## Assessment

The durable contribution is not the specific attack — that is patched — but the design lesson: **an encrypted reasoning block is a bearer token, and bearer tokens must be bound to a principal.** Providers reached for encryption to solve a business problem (anti-distillation) and inherited a security property they never specified (portability), and the paper shows the gap between "confidential" and "non-replayable" is where the whole vulnerability lives. The second lesson is about the reasoning channel as a *leak surface* independent of the attack: 64 artifacts present in reasoning and absent from the visible transcript means hidden CoT is accumulating sensitive material that neither users nor their redaction tooling can see.

## Related

- [[chain-of-thought]] — the hidden object; the whole vulnerability exists because providers decided CoT must not be shown.
- [[reasoning]] — reasoning-mode APIs are what introduced the client-held encrypted block.
- [[responses-api]] — the stateless-serving pattern that makes returning an encrypted reasoning block necessary.
- [[knowledge-distillation]] — the commercial motive for hiding traces; attack vector 1 is anti-distillation circumvention.
- [[prompt-injection]] / [[indirect-prompt-injection]] — attack vector 3; encrypted blocks are un-inspectable carriers, which breaks input-scanning defenses.
- [[guardrailing]] — the defense class this most directly undermines: you cannot filter what you cannot read.
- [[llm-agent]] — the demonstrated long-horizon poisoning target; agent trajectories are also the public corpus that leaked.
- [[red-teaming]] — attack vector 4 recovers hazardous reasoning behind a benign final answer, a blind spot for output-only red-teaming.
- [[llm-as-a-judge]] — used to classify recovered PII and credentials at scale.
- [[anthropic]], [[openai]], [[google]] — the three affected ecosystems; all three received disclosure and shipped mitigations.
- [[encrypted-reasoning-trace]] — the artifact itself; no page yet.
- [[jailbreaking]] — attack vector 4; no page yet.

## References

- [arXiv:2608.09867](https://arxiv.org/abs/2608.09867)
- Green (2026) — prior disclosure of reasoning-trace interchangeability, May 2026 (cited in the paper's disclosure section).
