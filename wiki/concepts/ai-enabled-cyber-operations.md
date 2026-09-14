---
title: AI-Enabled Cyber Operations
type: concept
created: 2026-09-14
updated: 2026-09-14
sources:
  - anthropic-threat-intelligence-report-september-2026.md
status: complete
importance: high
---

# AI-Enabled Cyber Operations

Offensive cyber activity in which language models do real work across the kill chain: reconnaissance, tool development, exploitation, data processing and persistence. The primary evidence in this wiki is Anthropic's September 2026 [[anthropic-threat-intelligence-report-september-2026|threat report]], which covers operations it disrupted between December 2025 and August 2026. All actors used Claude Haiku, Sonnet or Opus. The report found none on Claude Fable or Mythos, and says Mythos "has a series of safeguards in place that greatly reduce its ability to perform harmful cyber tasks."

The report's headline claim is that "AI has collapsed the labor and tooling gap that used to separate well-resourced, state-sponsored operations from individual operators." A hacktivist with stolen API keys, loosely connected criminals and a state espionage operator each sustained multi-victim campaigns that would have needed many skilled operators a year earlier.

## How the report measures harm

**Uplift** is the report's term for "how much more harm was caused with AI versus without AI," assessed along three axes: **speed**, **scale** and **depth**. The report argues the larger risk is not AI writing exploits at scale but AI across the whole kill chain, "where adversaries can operate faster, across a broader and deeper surface area, with fewer resources."

One consequence for investigators: "sophistication has stopped being a reliable signal of who is behind an operation."

## Four shifts

**1. From assistant to orchestrator.** The report describes a spectrum. At one end the model is an engineering assistant writing malware or phishing kits. Next is directed execution, where a human picks targets and the model runs commands, harvests credentials and exfiltrates data. Then come multi-agent frameworks running reconnaissance, exploitation and theft in parallel for hours or days. At the far end, collection fleets run on schedules with no human involvement. Humans kept three decisions throughout: target selection, monetization and review of results.

**2. "Vibe hacking."** In the report's words, operators "direct AI to achieve general goals like using a credential for an entity or retrieving data from a broad set of targets, then allow the AI to evaluate the environment, author and execute scripts, provide summaries, and repeatedly execute until the task is complete. Very often, the operator may not directly understand each target environment." It is the attacker's mirror of delegating a goal to a coding agent.

**3. The tradecraft diffused.** An autonomous operating model the report first documented for a suspected state campaign in November 2025 "has now proliferated across every class of actors we investigated." Public offensive agent frameworks such as PentAGI "reproduce much of the same scaffolding for anyone who downloads them."

**3b. Autonomy that outlives the session.** GTG-10007 is the report's fullest picture of this. Operators "routinely ran 'agent swarms,' where a lead AI agent decomposed reconnaissance and post-exploitation work and dispatched it to many subagents running in parallel." Target lists, harvested credentials, engagement state and standing instructions were "saved across working sessions, so each session could be resumed mid-campaign with the program's accumulated context." Its zero-day foundry loaded firmware into a decompiler through a tool server, walked cross-reference chains over thousands of decompile calls, formed hypotheses against a knowledge base it curated over time, wrote exploit code, tested it against lab copies of the product, and iterated until it worked. The collection and vulnerability-research workflows "kept operating while its owners were away."

**4. The cost moved back onto defenders.** A Russian espionage operator's monitoring agents watched for its malware being flagged, then had Claude "systematically identify, modify and redeploy the detected artifacts." The report's conclusion: "AI has inverted the cost back onto defenders. Previously, defenders might have been able to slow an attacker's operational tempo via the deployment of a new detection. Now, at least in theory, capable adversaries can 'close the loop.'"

## Autonomy is not severity

The report is explicit that the two are separate axes. Autonomy multiplies scale and speed and cuts operating cost, but "several of the most serious compromises we report here came from operations where a human directed every step." Its economic framing: autonomy "compresses the cost side of attacker ROI calculations … while leaving potential payoffs largely unchanged," which "makes previously marginal targets viable and encourages higher-volume, lower-touch operations."

## The measured tempo

| Case | Actor, as characterized by the report | What was measured |
|---|---|---|
| GTG-20006 | Russian state-nexus espionage | more than 20 organizations targeted; over 300,000 national identity records taken from one government; detection-evasion loop run by agents |
| GTG-50014 | financially motivated, suspected ShinyHunters affiliates | 1.8 million Android APKs mass-downloaded and scanned for secrets; over 2,100 Azure AD token sets across 40+ tenants in about 34 hours; one developer token to full cloud admin in roughly three hours |
| GTG-50020 | Russian-speaking, financially motivated | roughly thirty AI companies attacked in about four days with one adapted technique; earlier ransom demands of $1.5–2.5 million |
| GTG-50029 | a single French-speaking hacktivist | internal access to at least 14 of 42 tracked targets; ~140,000 records from one political platform; a 15,000-message mailbox |
| GTG-10007 | Chinese-speaking operators | roughly fifty organizations across eight sectors; "more than a dozen possible zero day findings in a single month" against network appliances; thirteen standing collection agents on a schedule |

Across the cases the report summarizes "breaches completed in two to three hours, and dozens of victims handled in parallel by individual operators."

## AI access as target, loot and compute

A theme new in this report: stolen AI access is simultaneously something to resell, compute to run attacks on someone else's bill, and cover, since the traffic is attributed to the legitimate key owner. Documented routes include compromising AI wrapper services' LiteLLM implementations "with prompt injection to exfiltrate the production API keys," prompt injection of OpenClaw deployments, and fraudulent resellers offering cheap "Claude" access while proxying to another model and harvesting customers' Anthropic credentials. Some actors distributed malware as applications spoofing popular AI harnesses, including [[claude-code|Claude Code]]. The same proxy-and-reseller ecosystem supplies [[illicit-distillation]] campaigns.

The most direct attack on the AI industry itself came from GTG-50020. After injecting malicious instructions into one AI vendor's automated evaluation sandbox, the actor took that vendor's production API keys and went on to attack roughly thirty AI companies in about four days. Its stated goal, pursued across more than a dozen avenues, was access to a pre-release Claude model. The report says every path failed, and that the keys involved were customers' keys stolen from customers' environments, not Anthropic's.

## Attackers use the same agent patterns defenders do

Appendix A of the report is a table of **skills** threat actors built for their workflows, each mapped to a MITRE ATT&CK technique. They are ordinary [[agent-skills]]:

- personas such as an offensive-security engineer, a DevOps engineer standing up C2 and phishing stacks, a Windows implant developer, a frontend designer polishing lure pages, and an "expert iOS Safari exploit researcher";
- flows that invoke a skill, load a `MEMORY.md` index, and update a **dead-end ledger** — "lists confirmed dead ends never to retry," which the report calls "institutional memory for exploit R&D."

A weapons-development cell in the same report ran several Claude instances in fixed roles — one writing code, one researching, one reviewing — "much as a lead would delegate work on a small engineering team." The orchestration, memory and skill patterns this wiki tracks on [[multi-agent]], [[skill-library]] and [[memory-management]] are in adversarial use, unmodified.

## Related

- [[illicit-distillation]] — the same access ecosystem, used to copy capability rather than attack victims.
- [[influence-operations]] — the report's other large harm area.
- [[prompt-injection]] · [[indirect-prompt-injection]] — the entry point into AI wrapper services.
- [[agent-skills]] · [[multi-agent]] · [[memory-management]] · [[claude-code]] — the patterns and tools involved.
- [[red-teaming]] — the pre-deployment counterpart to post-deployment threat intelligence.

## References

- [[anthropic-threat-intelligence-report-september-2026]]
