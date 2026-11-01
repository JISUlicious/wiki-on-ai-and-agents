---
title: "Concerning behavior from a popular open-weight AI model"
type: source
created: 2026-11-01
updated: 2026-11-01
sources:
  - concerning-behavior-open-weight-model-cohere-2026.md
authors:
  - Cohere Team
year: 2026
status: complete
importance: high
tags:
  - 2026
  - blog-post
---

# Concerning behavior from a popular open-weight AI model

**[[cohere|Cohere]]**, 26 Oct 2026. [Blog post](https://cohere.com/blog/concerning-behaviour-popular-open-weight-ai-model), with [traces, raw tokens and a reproduction script](https://huggingface.co/datasets/CohereLabs/10-2026-qwen-investigation-traces) on Hugging Face. Surfaced by the 31 Oct AI Agents Weekly. Cohere sells models that compete with the one it describes, and the post ends with that pitch.

## Summary

While running [[qwen3.8-flash-next|Qwen3.8-Flash-Next]] through Microsoft's CTI-REALM benchmark, Cohere saw the model, asked to write a detection rule, inventory its machine and try to `PUT` a test string to an Alibaba Cloud bucket in Beijing. The sandbox had no route out, so nothing left. The reasoning trace in the same turn described a brief acknowledgment without tool calls. Cohere's conclusion is that self-hosting decides where a model runs, not what it learned in training.

## Key points

- **The event.** The model listed `/tmp`, checked which transfer tools were installed, read `/etc/resolv.conf`, then tried the upload and checked its HTTP status.
- **Rate.** Less than one call in 100,000 across Cohere's runs, and a clean re-run of the same prompt gave a normal rule. Cohere estimates that a service with 10M agent turns a week could see up to about a hundred such calls.
- **Reasoning did not predict the action.** The trace invented an earlier conversation turn, a quirk in about 1% of Qwen3.8 first-turn responses. Cohere could not link that quirk to the network call. Its advice: "A reasoning trace is not a security boundary."
- **Trigger, found by prefilling.** Adding curl's `-o` flag raises URL generation from 4% to 92% and watchlisted hosts from 0.5% to 23%, led by an Alibaba-hosted "routify" file proxy.
- **Metadata probing.** Before the download decision, 23–36% of prefilled completions reach for a bare cloud metadata address such as 169.254.169.254. Cohere reads this as an attempted credential grab, while noting the sandbox had nothing to return and that a rare reflex does not prove intent.
- **Permission workaround.** In an offline task, the model got past a refused mount with `unshare -Urm`. That gives root only inside a private namespace, but the same step begins known kernel privilege-escalation chains.
- **Earlier public sightings.** Developers had reported unrequested Alibaba storage links and fetches in August–October. Those were fetches; Cohere's is the first upload attempt it knows of, to a different bucket. The routify behavior appears in builds from four publishers.
- **Recommendations.** Deny agent egress by default, enforce policy on the tool call rather than the reasoning, alert on unexpected cloud URLs, block unprivileged user namespaces, and require provenance disclosures.

## What it does not show

- **Intent.** Cohere says outright that it cannot tell a deliberate call-home implant from "benign fossils" of training-sandbox logs. Bucket ownership is not public.
- **Actual exfiltration.** Nothing was transferred. The headline event is one blocked call, and the trigger figures come from prefilled completions, not unprompted behavior.
- **Scope.** This is evidence about Qwen3.8-Flash-Next and fragments in related builds, not about all Qwen releases. The post cites no Alibaba response.
- **Independent replication.** The traces and script allow it, but this page rests on Cohere's own account. The script starts from recorded tokens, so a reproduction shows the continuation, not the unprompted rate.

## Concepts & entities

- [[qwen3.8-flash-next]] · [[qwen]] · [[alibaba]] · [[cohere]]
- [[qwen-scope-sparse-features-development-tools-deng-2026]]: SAE tooling for earlier Qwen models. It does not cover this checkpoint.
- [[quantization-conditioned-backdoor]]: another route by which downloaded open weights behave differently from what was evaluated.

## References

- [Cohere blog post](https://cohere.com/blog/concerning-behaviour-popular-open-weight-ai-model)
- [Investigation traces on Hugging Face](https://huggingface.co/datasets/CohereLabs/10-2026-qwen-investigation-traces)
