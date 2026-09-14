---
title: Illicit Distillation
type: concept
created: 2026-09-14
updated: 2026-09-14
sources:
  - anthropic-threat-intelligence-report-september-2026.md
status: complete
importance: high
---

# Illicit Distillation

Covert, unauthorized [[knowledge-distillation|distillation]] of a frontier model into another lab's model. The legitimate technique is the same; what makes it illicit is the access. Anthropic's September 2026 [[anthropic-threat-intelligence-report-september-2026|threat report]] defines it as "an industrial-scale, covert campaign to extract a model's capabilities and replicate them in another model without authorization," and notes it is "typically enabled by fraud": networks of fake accounts built on stolen credit cards, login credentials and API keys.

The report names seven China-based labs as having run such campaigns against Claude between February and August 2026. It says OpenAI has reported the same activity since early 2025 and Google published a threat tracker on it in 2026. The targets were the most valuable capabilities: agentic work and tool use, coding and data analysis, and logical reasoning.

> [!warning] Provenance: a vendor's allegations against its competitors
> Every attribution on this page comes from Anthropic, which is itself a party to the dispute. The report says the attributions are "high confidence," but it publishes no method for attributing traffic to a lab, and the named labs' responses are not included. No third-party verification exists in this wiki. "Exchanges observed" counts requests, not unique training examples. The observation windows range from 14 days to three months, so the scale figures below cannot be compared as rates. The claim that particular released models were trained on distilled Claude outputs is also Anthropic's, not the model developers'.

## How access is obtained

- **Proxy networks ("transfer stations").** Thousands of accounts made with false identities, fake or stolen cards, and stolen API keys belonging to real companies, used to route around geographic restrictions.
- **Bought transcripts.** Resellers and routing services log their users' exchanges with frontier models, often without those users' knowledge, and sell the logs.
- **Rerouting one's own users.** A lab silently forwards its own customers' requests to Claude, shows Claude's answers as its own model's, and keeps the exchanges.

The third route has a privacy consequence the report stresses. Customers of those labs had their prompts sent to a third party they never chose. The report cites relayed traffic containing names, emails, company data and live credentials of hundreds of end users in at least a dozen languages. Some came through routing services commonly used in the US and Europe.

### What the relayed traffic contained

The report names three labs as having fed their own users' conversations into Claude: Moonshot, DeepSeek and Xiaomi. It gives categories, never contents, and redacts every sensitive value.

| Relayed by | Whose data | What it contained |
|---|---|---|
| [[moonshot-ai\|Moonshot]] | a user assessed as likely affiliated with the Chinese military | a CCTV archive from hundreds of Chengdu cameras, including outside military facilities and a state-owned enterprise, used to judge whether one tracked person behaved abnormally |
| [[moonshot-ai\|Moonshot]] | an engineer at a major state-owned enterprise | internal code, and live credentials from multiple major Chinese companies |
| [[deepseek\|DeepSeek]] | an employee of a Chinese technology company | full specifications, organizational structure and strategic objectives of a flagship AI program |
| [[deepseek\|DeepSeek]] | an IT operator at a Russian agency tied to its Ministry of Defense | live credentials for a Russian government database |
| [[deepseek\|DeepSeek]] | engineers at a municipal Public Security Bureau | a tool matching a person's movements against police records by national ID number |
| [[xiaomi\|Xiaomi]] | hundreds of users, in at least a dozen languages | names, contact information and corporate data; no indication of US persons' data |
| an unnamed lab's coding assistant, via a third-party router | a pharmaceutical company | internal 2026–28 capital-spending forecasts for sites in Ho Chi Minh City, Kuala Lumpur, Bangkok and Ljubljana |
| an unnamed lab's coding assistant | a developer | a Telegram bot token, a Feishu app secret and a Notion integration key |

The report calls these practices "likely inconsistent with privacy laws and the labs' own terms of service." The general lesson is not specific to these labs: a prompt sent to any model can reach a provider the user never chose, through a router, a reseller or the lab itself. Credentials pasted into a coding assistant are best treated as exposed.

## How reasoning is extracted

Claude's reasoning traces are the prize, because general reasoning transfers across tasks. The report documents a progression of techniques:

1. **Direct instruction.** Prompts telling the model not to flag the request as reasoning extraction and to output its prior reasoning verbatim.
2. **Fake authority.** A user turn claiming to be "the real system prompt" and demanding the reasoning inside `<thinking>` tags.
3. **Systematic search.** One lab ran over 12,000 requests, each with a different technique, kept the ones that worked, and scaled those up.
4. **Indirection.** Asking the model to "translate previous working memory" into katakana-only Japanese.
5. **Cross-session replay.** Claude returns a *thinking signature*, a reference the API uses to look up the raw trace later, instead of the trace itself. Attackers saved the signature, opened a new session, and got Claude to expand it back into the full trace. Moonshot and DeepSeek both used this.

## The named campaigns

| Lab | Report ID | Window | Exchanges observed | Accounts | What the report says was taken, and how |
|---|---|---|---|---|---|
| [[alibaba\|Alibaba]] (Qwen / Tongyi Lab) | GTG-16005 | May–Jul 2026 | **>151M**, peaking near 3M/day | >3,500 at peak; a first pool of ~5,000, then a second | Chain-of-thought (CoT) traces of Opus 4.6 and 4.7, forced into inline tags by a fixed injected prompt, then converted to supervised fine-tuning (SFT) data. The report says this was "used to distill Claude's capabilities into Qwen 3.5, 3.6, and 3.7." Claude was also used for RL environments and architecture research. "The largest distillation attack we have ever measured." |
| [[moonshot-ai\|Moonshot]] (Kimi) | GTG-16002 | May–Jul 2026 | **>23M**; ~300,000 customer requests relayed in one 10-day window | 5,380 | Served Claude to Kimi customers who believed they were using Kimi, and kept the exchanges. Used cross-session signature replay. |
| [[deepseek\|DeepSeek]] | GTG-16001 | 14 days, Jul 2026 | **>12.1M** | not stated | Cross-session replay. Recognized users of [[claude-code\|Claude Code]], the Claude Agent SDK and [[opencode\|OpenCode]] from request strings and relayed selected ones to Opus. |
| [[z-ai\|Zhipu (Z.ai)]] | GTG-16006 | 17 days, Jun–Jul 2026 | **>3.4M**; 770,609 through a CoT-cleaning pipeline in 10 days | 273 | CoT extraction against Opus 4.8, with Claude also judging and cleaning the harvested traces. Ahead of GLM 5.3, a cyber-capability campaign using CTF challenges built from public vulnerability data. |
| [[xiaomi\|Xiaomi]] (MiMo) | GTG-16008 | 20 days, Mar–Apr 2026 | **>400,000** | >1,500 | Replayed its own users' coding sessions, often from [[openclaw\|OpenClaw]] and OpenCode, through Claude to generate SFT and RL data. The report suggests MiMo-V2-Pro's extended free trial was timed to harvest developer traffic. |
| SenseTime | GTG-16012 or 16003 | not stated | not stated | not stated | Bought harvested Claude transcripts from third-party data vendors, and used Claude to write the distillation pipeline itself. |
| [[minimax\|MiniMax]] | GTG-16012 or 16003 | not stated | not stated | not stated | Ran a proxy service through an undisclosed shell company that offered only Anthropic and OpenAI models, none of its own. |

The report pairs SenseTime and MiniMax under one heading with two IDs and does not say which ID is which.

## What the safeguards evidence actually shows

The report says no misuse involved Claude Fable or Mythos-class models, with one distillation exception. Those two claims rest on different evidence and should not be read the same way.

- **Fable: direct evidence of deterrence.** Zhipu first targeted Fable's cyber capabilities, "eventually gave up," and its employees switched to Opus 4.6 and another US lab's model "expressly because they assessed the safeguards were weaker." That is an observed substitution, the strongest kind of evidence a safeguard can produce.
- **Mythos: mostly an access story.** The report states Mythos 5 and Mythos Preview "are not accessible to the general public." Absence of attempts is what restricted access predicts, whatever the safeguards are.

## Claims that raise the stakes

Two claims in the report go beyond the incidents themselves:

- Anthropic's own research, not published in the report, finds distillation "can deliver significant uplift" in the targeted domains "using fewer exchanges than those harvested in the campaigns described here."
- A model distilled from a frontier model "can help achieve dangerous capabilities, including those in the biological or cyber domains, even when the harvested exchanges contain little about those subjects," and "the robust safeguards that prevent Claude from being misused … do not transfer." In the report's framing, distillation moves capability without moving refusals.

Both are asserted without data. They are the reason the report treats distillation as a security issue rather than only a commercial one.

## Defenses described

1. **Attribute, then enforce.** Rather than banning proxy accounts one by one, link traffic to the responsible organization and act against all of it.
2. **Extraction classifiers.** Built to detect adversarial extraction, and strengthened alongside the Fable 5 launch.
3. **Reasoning summarization.** Claude summarizes its internal reasoning before responding, which makes stolen transcripts less useful as training data.
4. **Preserved thinking** (introduced with Fable 5.1). New API accounts cannot alter the system prompt, tools or messages that precede Claude's reasoning in multi-turn conversations. The reasoning is encrypted, and editing the preceding context was a common way to make Claude reveal it.
5. **Identity verification** when abuse signals appear, such as unauthorized resale or access from unsupported countries. Accounts that fail verification are banned.

The report calls this a layered defense because "no single safeguard can address this issue alone."

## Related

- [[knowledge-distillation]] · [[on-policy-distillation]] — the legitimate technique this abuses.
- [[chain-of-thought]] · [[chain-of-thought-prompting]] — the traces being extracted, and why summarizing them is a defense.
- [[qwen]] · [[moonshot-ai]] · [[deepseek]] · [[z-ai]] · [[minimax]] · [[xiaomi]] — the labs named.
- [[claude]] · [[claude-code]] · [[openclaw]] · [[opencode]] — the model and the harnesses involved.
- [[ai-enabled-cyber-operations]] — the same proxy and reseller ecosystem feeds the cyber cases.

## References

- [[anthropic-threat-intelligence-report-september-2026]]
