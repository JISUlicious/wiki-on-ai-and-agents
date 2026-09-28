---
title: "E-Commerce Bench: Evaluating LLM Agents on Long-Horizon Autonomous Business Operation"
type: source
created: 2026-09-07
updated: 2026-09-07
sources: []
arxiv_id: "2608.30730"
authors:
  - Wei Fan
  - Xinjie Shen
  - Xudong Guo
  - Jianhong Tu
  - Yang Su
  - Yinger Zhang
  - Lianghao Deng
  - Fengyu Wang
  - Baohua Dong
  - Yangqiu Song
  - Dayiheng Liu
year: 2026
status: complete
importance: high
tags:
  - 2026
---

# E-Commerce Bench: Evaluating LLM Agents on Long-Horizon Autonomous Business Operation

**Wei Fan, Xinjie Shen** (equal contribution)**, Xudong Guo, Jianhong Tu, Yang Su, Yinger Zhang, Lianghao Deng, Fengyu Wang, Baohua Dong, Yangqiu Song, Dayiheng Liu** — [[qwen|Qwen Team]], [[alibaba|Alibaba Group]]; Taobao & Tmall Group, Alibaba Group (Wang, Dong); HKUST (Song) — [arXiv:2608.30730](https://arxiv.org/abs/2608.30730), cs.LG / cs.CL, submitted 31 Aug 2026, v1. Code: `github.com/QwenLM/E-CommerceBench`. No venue given.

## Summary

A 365-day continuing business simulation in which an LLM agent runs up to four online stores on a simulated Chinese marketplace, starting from a **¥100,000 stake** and scored on end-of-year total assets. The design goal is stated in RL vocabulary: most agentic benchmarks are **episodic** (a goal, a terminal state, a graded deliverable), while deployment looks **continuing** (no terminal state, an accumulated score, failures that only appear over hundreds of days).

Two design commitments distinguish it from every sibling.

**Determinism on both sides of the market.** Customer purchases and returns follow a fixed multi-factor demand model whose only randomness is one episode-shared seed. Supplier behavior runs on a **deterministic negotiation kernel** that fixes every price, concession, and accept/reject decision — an LLM is used *only to verbalize* those decisions as dialogue. The paper's claim is that this removes sampling noise from the counterparty, so an outcome difference "entirely belongs to the agent policy rather than to chance." This is a direct methodological attack on the Vending-Bench line, where LLM-driven suppliers make runs stochastic and manipulable.

**Seven scoring dimensions, not one number.** End-of-year assets is the primary score, but six capability axes are ranked independently — negotiation quality, fraud avoidance, cash flow and solvency, operational efficiency, operations execution, and learning over the horizon. The thesis is that a single financial scalar hides where a model is weak, and the results bear it out: **no model is strong on all seven**, and six of the seven vendor-family leaders fall below the 18-model median on at least one axis.

The headline is a deliberate embarrassment of the leaderboard: **GPT-5.6 Sol earns the most, growing ¥100,000 into ¥1,431,425 (≈14×), yet ranks 16th of 18 on fraud avoidance and earns ¥363 per tool call against Fable5's ¥479.** Among open-weight models **Qwen3.8-Max-Preview leads at ¥416,252, 38% above [[glm-5-2|GLM 5.2]] (high)**, and is the only model in the field that measurably bargains prices *down* over the year.

## Architecture — four layers

**1. Agent loop layer.** A turn-based loop over simulated time. One model call emits a batch of tool calls that execute in order. Every tool but the day jump costs **simulated minutes out of a 600-minute working day (08:00–18:00)** — 10 minutes for a balance check, 60 for opening a store, 30 for one `chatbox` message regardless of how many suppliers it addresses. This makes the tool list a budget rather than a menu, and it is the mechanism that forces exploration/exploitation tradeoffs: "Looking something up draws on the same clock as acting on it." An episode caps at **4,000 turns** or ends after three tool-free turns; the longest of the 90 episodes took **3,599 turns**, so no run hit the cap.

**Context management** is uniform and disclosed to the agent verbatim. A single tokenizer counts every model against the same **128,000-token budget**; eviction fires at **120,000 tokens** with a **60,000-token release target**, dropping whole tool-call groups oldest-first while sparing the system message, the first user turn, and the two newest groups. Across the 90 episodes, **1,495 eviction passes ran**. Loss scales with tool traffic rather than turn count: Claude Opus 4.7 finishes in 432 turns and discards ~4 windows' worth, while Gemini 3.5 Flash takes 2,628 turns and loses nearly 19; Kimi K2.6 at 1,017 turns loses the same 3.5 windows as Opus 4.7 at 432, because it drags in 431 tokens per turn against Opus 4.7's 1,058, and GPT-5.5 loses 28 windows at 2,759 tokens per turn.

**Persistent memory** is a store of at most **20 notes** (title + content) living in the environment, beyond eviction's reach, fully curated by the agent. Reading it back is cheap in minutes but expensive in tokens, since retrieved entries re-enter the context and become evictable again.

**2. Tool layer.** **18 tools** in seven groups. Notably, `chatbox` is the *only* tool that can create a purchase order, so every unit of inventory arrives through free-form text with fenced `negotiate` blocks — routine mail, honest offers, and fraudulent pitches all cost the same 30 minutes through that one channel.

**3. Sales and economy engine.**

- **Multi-store**: **12 store types**, each covering a different category set (9 categories down to 1); **at most four trade at once**, one per type. ¥500 to open, ¥60–130/day to run; leftover stock keeps accruing storage unless liquidated at **10% of purchase cost**; reopening restarts reputation at the floor.
- **Three-account deferred settlement** — the mechanism most likely to kill a run. Costs leave the bank the day incurred (a purchase order in full at deal close), while revenue only enters escrow *after shipping*, matures into the platform wallet **nine days later**, and reaches spendable cash only on an explicit withdrawal. A worked example: a 40-unit lot costs ¥2,447 on day 0 and returns its first cash on **day 16**. **Ten consecutive days closing with a negative bank balance ends the episode as bankrupt** — so an agent selling profitably can still fail by leaving revenue in the wallet.
- **Demand** is a product of named factors: category-and-store-type base × price response × weekend uplift × promotion × seasonality × calendar event × reputation, then trimmed by **two capacity ceilings** (per-category and per-store) and capped at shelf stock. One of the four price-elasticity families is **quadratic and symmetric about the public reference price**, so in those categories discounting *loses* demand. The capacity terms can dominate everything: for one accessory SKU on a promotion Saturday they cut a compounded expectation of 128.2 units to the 9 actually sold, the category term alone taking a factor of 7.5. Consequence: **stacking homogeneous SKUs cannibalizes rather than adds**.
- **Returns** have four channels — natural category rate, supplier defects, pricing above reference (1.3× reference multiplies the rate by 1.50), and shipping speed (slow multiplies by 1.30). Two of the four answer to the agent, which is what makes the "controllable return rate" a management score rather than a catalog property.
- **Reputation** multiplies demand directly; a floored store gets **15%** of top-tier demand. Credit accrues at shipment, not sale. Returns and cancellations subtract weighted 0.6 and 1.0, so a missed deadline costs more than a change of mind.
- **Dynamic events**: **10 market events and 8 promotions** on fixed dates, rarely uniform in direction (a storm that lifts food cuts fashion). News arrives as the event begins; a promotion is announced a week ahead **without its dates or size**. Promotion boost peaks at a 30% discount. Suppliers retire **after a hidden number of filled orders**, so a counterpart whose floor the agent has learned can vanish before the next re-order.

**4. Data layer.** **6,886 products in 60 categories, 576 suppliers (424 honest, 152 fraudulent)**, desensitized from or synthesized against real Taobao & Tmall logs. Every supplier serves exactly one category; no category holds fewer than 7 honest and 2 fraudulent, and fraud coverage stays flat across catalog depth. **Promotion names are fictional** — a deliberate guard so a model recognizing China's largest shopping festival cannot price from memory instead of evidence. The supplier list is fixed at release rather than redrawn per episode.

**Information asymmetry is engineered, not incidental.** Market lookups return *labels* where quantities exist: a profit-potential grade instead of demand capacity, a margin class instead of the wholesale ratio, a sentence about returns instead of the per-SKU rate. Hidden parameters differ deliberately in learnability — the best price won bounds a supplier's floor from above, a SKU's return rate becomes readable once stock moves, but an **elasticity parameter stays hidden all year** (four unprinted factors multiply the same observation) and a store type's earning ceiling shows only after operating it for a year.

### The negotiation kernel and adversarial suppliers

Each sourcing cycle is a session `z = (s, j, c)` — the c-th between supplier s and SKU j — a bilateral bargaining problem under incomplete information with no fixed round limit; a deadline scale supplies the time pressure, and past its midpoint a supplier may walk from an under-floor offer. A worked example from the evaluation: Ridge Express quotes ¥149.81 against a market reference of ¥199.75, over a hidden floor of ¥101.27 — **¥98.48 for the two sides to divide**.

Sessions repeat, and that is the point. A closed session reopens under a fresh kernel instance and seed but the **same persistent type and reservation price**, so the best price already won is a monotonically tightening upper bound on the supplier's floor. Closed deals are **not re-surfaced**, so the agent must keep its own price history or do without it — which turns the memory store into the load-bearing component of the learning axis.

**152 of the 576 suppliers are fraudulent**, in two classes:

| Class | Where the damage lands | Mechanism |
|---|---|---|
| Pre-deal | Inside the negotiation | Reservation price inflated toward **1.5× the honest cost floor**; still inside the range an honest supplier might quote, so only the narrative (membership fees, phantom discounts, manufactured urgency) gives it away |
| Post-deal | At fulfillment | Honest floor and quote, then defection: short delivery of **60–70%** of paid quantity, or defective stock returning at **≥0.40 or double the natural rate, capped at 0.95** |

No single observation exposes a fraudster. Opening quotes are uninformative by construction and the behavior family is latent and shared with honest suppliers; the only bargaining-side cue is **concession rate** — fraudulent kernels hold out longer than the hardest honest preset — and that signal lives only in the shape of a multi-round exchange, so **an agent that accepts early forfeits its one detection channel**. Defective rates are blended across pooled stock, so interleaving honest and fraudulent deliveries dilutes the evidence and no observed return rate points back to a culprit.

## Setup

18 models (8 proprietary, 10 open-weight by the authors' assignment) × **5 independent episodes = 90 episodes**, one fixed world, provider default settings with no temperature or top_p sent and thinking force-enabled where exposed. All models share the same 18 tools, context policy, and memory store. One renderer voices every supplier for every model. Five episodes is acknowledged as a small sample, so every mean is printed with its standard deviation. Bankrupt episodes **stay pooled into the means**.

## Results

**End-of-year leaderboard** (¥k, mean of 5; ↑/↓ mark the better direction):

| Tier | Model | Assets ¥k | std | CSE⁺ ↑ | BadSpend% ↓ | Drawdown/peak ↓ | ¥/call ↑ | Ctrl. return pp ↓ | AnchorRatio ↓ | Calls | Turns | Bankrupt |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Prop. | **GPT-5.6 Sol (max)** | **1,431** | 314 | 0.672 | 18.48 | 0.278 | 363 | 4.46 | 1.217 | 3,668 | 1,367 | 0/5 |
| Prop. | Fable5 (max) | 805 | 188 | 0.772 | 3.46 | 0.141 | **479** | 0.22 | 1.573 | 1,469 | 945 | 0/5 |
| Prop. | GPT-5.5 | 702 | 616† | 0.700 | 16.59 | 0.591 | 192 | 0.90 | 1.043‡ | 3,143 | 1,306 | 2/5 |
| Prop. | Claude Opus 4.8 (max) | 498 | 231 | 0.662 | 5.41 | 0.130 | 266 | 0.20 | 1.309 | 1,497 | 812 | 0/5 |
| Prop. | Claude Opus 4.7 (max) | 259 | 111 | **0.811** | **0.12** | 0.189 | 156 | 1.75 | 1.421 | 1,023 | 432 | 0/5 |
| Prop. | Claude Opus 4.6 (max) | 258 | 266 | 0.649 | 14.98 | 0.587 | 129 | 0.76 | 1.268 | 1,221 | 788 | 2/5 |
| Prop. | Gemini 3.5 Flash | 190 | 79 | 0.777 | 2.13 | 0.450 | 36 | 0.38 | 0.918 | 2,508 | 2,628 | 0/5 |
| Prop. | Gemini 3.1 Pro | 130 | 130 | 0.660 | 12.58 | 0.688 | 19 | 2.03 | 1.376 | 1,605 | 1,141 | 2/5 |
| Open | **Qwen3.8-Max-Preview** | **416** | 111 | 0.713 | 6.13 | 0.242 | 173 | 0.38 | **0.834** | 1,826 | 962 | 0/5 |
| Open | GLM 5.2 (high) | 301 | 124 | 0.693 | 1.99 | **0.127** | 137 | 0.59 | 1.434 | 1,467 | 672 | 0/5 |
| Open | Kimi K3 | 265 | 110 | 0.632 | 5.87 | 0.247 | 123 | 2.01 | 1.507 | 1,334 | 878 | 0/5 |
| Open | GLM 5.1 | 226 | 192 | 0.662 | 6.49 | 0.231 | 94 | **0.09** | 1.557 | 1,333 | 852 | 0/5 |
| Open | DeepSeek-V4-Pro-Preview (max) | 190 | 100 | 0.654 | 8.99 | 0.166 | 68 | 1.54 | 1.235 | 1,337 | 678 | 0/5 |
| Open | Qwen3.7-Max | 165 | 134 | 0.616 | 9.85 | 0.433 | 59 | **−0.05** | 1.433 | 1,108 | 782 | 0/5 |
| Open | GLM 5.2 (max) | 115 | 71 | 0.632 | 19.68 | 0.360 | 10 | 2.24 | 1.362 | 1,430 | 931 | 0/5 |
| Open | Kimi K2.6 | 70 | 14 | **0.596** | 17.22 | 0.377 | −27 | 0.42 | 1.548 | 1,087 | 1,017 | 0/5 |
| Open | Qwen3.6-Plus | 47 | 28 | 0.625 | 10.68 | 0.583 | −40 | 3.03 | 1.333 | 1,325 | 1,023 | 0/5 |
| Open | Qwen3.5-Plus | **1.1** | 11 | 0.625 | **20.11** | **0.979** | **−92** | **5.88** | **1.860** | 1,076 | 800 | **4/5** |

† population estimate; ¥689k on the sample estimator. ‡ averages three episodes — GPT-5.5's two bankrupt runs never re-ordered from a supplier they had already dealt with.

**A factor of 1,264 separates top from bottom** (GPT-5.6 Sol to Qwen3.5-Plus), and because the environment is deterministic given the action sequence, that range belongs to the agents. Proprietary systems take the top four places; Qwen3.8-Max-Preview places fifth overall. **Ten of the ninety episodes ended insolvent**, six of them under closed-source models, all ten from just four models (GPT-5.5, Claude Opus 4.6, Gemini 3.1 Pro, Qwen3.5-Plus) — the same four that surrendered more than half of peak assets.

### The dimension results, which are where the paper earns its keep

**Negotiation (CSE⁺ — share of the reference-to-floor range captured; 1.0 = buying at the floor).** No model reaches the floor. Claude Opus 4.7 leads at **0.811**, Kimi K2.6 trails at **0.596**. **Accepting the opening quote already scores 0.50**, so the entire field lands only **0.10–0.31 above simply taking the first offer**. Which counterpart a model draws matters far less than which model it is (spread 0.051 across six behavior templates vs. 0.204 across models). Agreement separates nobody: of **12,060 concluded honest sessions only 73 end without a deal**, and 72 of those are the agent's own rejection. Agents systematically **avoid the hardest bargainers** — the toughest of six honest templates covers 48 of 424 honest suppliers and takes just **4.8% of the ¥55.1M of honest procurement**, against ≥14% for each easier template.

**Fraud avoidance.** Every model loses money to fraud; spending spans **more than 160-fold**. The counterintuitive finding: **losing money to fraud does not cost a model its year** — the cleanest buyer (Claude Opus 4.7) finishes mid-pack on assets, and the profit leader is 16th of 18 on BadSpend%. The reason is structural: a pre-deal scam inflates a price that still sits inside the honest range, and a post-deal scam still delivers most of the goods. **Defective lots take 53.4%** of all money reaching fraudulent suppliers; membership-fee scams land **three times in the entire evaluation**. Models do not differ in *whom they approach* — fraudulent suppliers take close to their list share of every model's outreach — they differ in **how often an approach becomes an order: 4.0% for Claude Opus 4.7 against 31.7% for GPT-5.6 Sol.** The discipline is in walking away, not in screening. Caveat the authors state: across five episodes the **standard deviation of BadSpend% exceeds its own mean for 15 of the 18 models**.

**Cash flow.** No episode sees cash in the wallet before **day 15**, and **49 of the 90 bottom out in January**. Two GPT-5.5 episodes went bankrupt on **day 17, before a single yuan had settled**. Seventeen episodes went negative at a morning check across 177 such days, and ten of those became bankruptcies — but on **66 of those days the wallet already held the shortfall**, i.e. the agent was solvent and merely hadn't withdrawn. **No tool reports the overdraft streak**, so an agent cannot count the mornings it has failed. A large buffer buys survival and caps growth: three models held the bank above ¥15,000 every episode and finished sixth, tenth, and eleventh, while the two biggest earners ran the thinnest buffers of the fourteen solvent models.

**Operational efficiency.** Fable5 turns ¥479 per call against GPT-5.6 Sol's ¥363 **on 59.9% fewer calls**. Three models average a *loss* per call, ¥−92 at worst. Where calls go describes policy better than their count: **shipping, day-advancing, withdrawing, and publishing absorb 71.8% of all calls — a daily loop rather than a plan.** Reading state takes 14.7% and **supplier conversation just 6.0%**, so the one channel that can change what an item costs draws less than a fifteenth of the effort. **Pricing revision is 0.66% of all calls**; nine of 18 models average under three price-revision calls a *year*, leaving listed prices to stand while season, promotion, and reputation move demand underneath them. The **memory store draws 2.1% of calls** — Claude Opus 4.8 gives it the largest share (4.7%, 69.6 calls/year against 20 slots), GPT-5.5 the most calls outright (135.2), and **Qwen3.5-Plus wrote to it in none of its five episodes**.

**Operations execution.** Controllable return rate runs from **−0.05 pp (Qwen3.7-Max) to 5.88 pp (Qwen3.5-Plus)**; the natural rate underneath is larger (5.31–21.25 pp) and follows what a model chose to sell. Twenty-six of 90 episodes carry a negative rate. Crucially the axis is not a proxy for skill: **GPT-5.6 Sol books 4.46 pp and the year's largest refund bill (¥1.62M) and still finishes first**, because added returns are worth buying when the price causing them pays for them. Freight tier "reads as sampling, not policy" — the archive holds **92,182 fast dispatches, 171,642 standard, and exactly one slow**, with 72 of 88 selling episodes putting ≥98% in one tier, yet nine models swing over 30 points in fast share between their own episodes.

**Learning over the horizon — the paper's most damning axis.** AnchorRatio measures whether a repeat order beats a reshuffling of the prices that agent already won from that supplier; **1.0 means indistinguishable from random ordering of its own prices**. Only **two of 18 models beat 1.0**: Qwen3.8-Max-Preview at **0.834** and Gemini 3.5 Flash at **0.918**. The field median is **1.369**; Qwen3.5-Plus reaches 1.860. **Fifteen models miss their own permutation null by more than two standard deviations in the expensive direction** — the signature of an unrevised anchor on an early price. A second, independent reading agrees: **16 of 18 models capture less surplus in the second half of an episode than in the first**, across **8,647 repeat purchases** of the same item from the same honest supplier. And supplier choice drifts the wrong way — **fraudulent suppliers take a larger share of concluded sessions late in the year than early for 17 of the 18 models**.

The authors name three candidate mechanisms and honestly decline to separate them: eviction destroys the tool result carrying a price already won; the eviction-proof memory store is barely used; and no gradient reaches inside an episode, so improvement must come from an agent reasoning over its own record.

### Error analysis

Ten failure patterns get one detection rule each. **Repeat business with a known-bad counterparty is where the money goes: 943 of the 1,141 agreements struck with fraudulent suppliers sit in a supplier-item pair dealt with twice or more.** No message names the supplier behind a bad lot, so the loss lands in the balance rather than arriving as a signal. **Research is re-derived rather than remembered**: of 2,180 catalog and supplier queries, **746 repeat arguments already issued that episode**, costing 7,460 simulated minutes — but that is only 83 minutes per episode against a 219,000-minute year, so the paper correctly calls this cheap rather than fatal.

Three commonly blamed patterns **do not survive the data**: store churn does not separate the field; models that join more promotions end richer (so event neglect explains nothing); and **late-year collapse holds only in part** — growth drops after the first quarter at a paired **t = −3.95**, but the closing window is flat rather than recovering. Failures are field-wide, not concentrated in weak models: **every model trips at least five of the ten rules, Gemini 3.1 Pro all ten, and tripping fewer does not make a model richer.**

## Reconciliation with [[merchantbench-long-term-coherence-shi-2026|MerchantBench]]

These are direct siblings — both 365-day e-commerce agent simulations grounded in real Alibaba marketplace data, published a month apart (MerchantBench 31 Jul 2026 on 1688 wholesale; this one 31 Aug 2026 on Taobao & Tmall retail), and **both from Alibaba Group**. E-Commerce Bench cites MerchantBench directly, characterizing it as a data-grounded retail benchmark that "add[s] real products and open code while dropping negotiation and adversarial dynamics altogether," with suppliers arriving as stochastic events. Its comparison table credits MerchantBench with real data, reproducibility, and open source, and marks it absent on negotiation, multi-agent, and adversarial dynamics, at ~700 steps/run over 1 operating unit.

**Do they agree on how badly agents do?** On their headline framings they appear to disagree violently — MerchantBench says the best agent reaches **27.3% of mean human final net assets**, while this paper reports a **14× multiplication of the opening stake**. The disagreement is an artifact of what each compares against, and working it out is the useful exercise.

**1. The decisive difference is that E-Commerce Bench has no human baseline.** MerchantBench ran three inexperienced humans through one 365-day run each; this paper runs no humans and no rule-based controller. A multiplier has no ceiling attached to it, so "14×" carries no information about competence.

**2. Normalize both to the same units and the apparent conflict inverts.** MerchantBench opens each run with RMB 2,000 cash plus a RMB 1,000 security deposit, and its terminal score `B_T + D_T + I_T + Q_T` counts the deposit — so the opening position is ¥3,000. Its best configuration (Hermes Qwen3.7-Max) finished at RMB 59.46k. *(Arithmetic below is mine; neither paper reports a multiplier for MerchantBench.)*

| Benchmark | Opening position | Best agent | Agent multiplier | Human | Rule-based |
|---|---|---|---|---|---|
| MerchantBench | ¥3,000 | ¥59,460 | **≈19.8×** | ¥217,610 (**≈72.5×**) | ¥24,480 (≈8.2×) |
| E-Commerce Bench | ¥100,000 | ¥1,431,425 | **≈14.3×** | — | — |

**MerchantBench's best agent multiplied its capital *more* than E-Commerce Bench's best — and was still only 27.3% of an amateur human.** So the two benchmarks do not disagree about agent competence at all. They agree, and E-Commerce Bench simply has no instrument that can express it. Reported in MerchantBench's units, GPT-5.6 Sol's 14× would be an unmeasured quantity; reported in E-Commerce Bench's units, MerchantBench's headline failure would read as a 20-fold triumph.

**3. But the multipliers are not interchangeable either, and the direction of the bias is worth stating.** MerchantBench is **single-item dropshipping with no held inventory** — no capital is tied up in stock, and procurement happens at order time. E-Commerce Bench requires buying inventory up front, storing it, and liquidating leftovers at 10% of cost. It also starts agents with a **33× larger stake**. A smaller, less capital-intensive base is easier to multiply, which cuts *against* reading MerchantBench's 19.8× as evidence of stronger agents. The honest conclusion is the reductio: **the growth multiple is not a competence measure in either benchmark**, and E-Commerce Bench's abstract leans on one because it has nothing better.

**4. On overlapping models the two agree more than the headlines suggest — but only under one of MerchantBench's two harnesses.** Restricting to the six models both evaluate:

| Rank | E-Commerce Bench | MerchantBench (ReAct) | MerchantBench (Hermes) |
|---|---|---|---|
| 1 | GPT-5.6 Sol | GPT-5.6 Sol | Qwen3.7-Max |
| 2 | Claude Opus 4.8 | Claude Opus 4.8 | GPT-5.6 Sol |
| 3 | GLM 5.2 (high) | GLM-5.2 | GLM-5.2 |
| 4 | DeepSeek-V4-Pro-Prev. | Kimi K2.6 | Claude Opus 4.8 |
| 5 | Qwen3.7-Max | Qwen3.7-Max | Kimi K2.6 |
| 6 | Kimi K2.6 | DeepSeek-V4-Pro | DeepSeek-V4-Pro |

The **top three match exactly** between E-Commerce Bench and MerchantBench-under-ReAct — a real cross-benchmark convergence over two different marketplaces, two economies, and two scoring functions. The tails diverge (Kimi K2.6 is last here at ¥70k but fourth under MerchantBench-ReAct), and MerchantBench-under-Hermes reorders the top entirely, putting **Qwen3.7-Max first** where E-Commerce Bench places it **twelfth of eighteen**. Version and configuration differences (GLM 5.2 "high" vs. plain, DeepSeek-V4-Pro-Preview vs. -Pro) confound some of this.

**5. That last row is a live threat to E-Commerce Bench's rankings, and the paper does not address it.** MerchantBench's central methodological finding is that **harness choice moves final assets by −4.1% to +187.8% depending on the backbone**, and that it reorders the leaderboard. E-Commerce Bench holds **one fixed harness** for all 18 models — the same 18 tools, the same 120k/60k eviction policy, the same 20-slot memory store. Its rankings are therefore properties of *model × this harness*, not of models, and by MerchantBench's own measurement that qualifier is large enough to flip first place. This is the most important unaddressed limitation in the paper, and it is unaddressed precisely because the two papers were written in parallel.

**6. Where they genuinely disagree: the dominant failure mode.** MerchantBench's headline diagnosis is **Activity Decay** — agents progressively cease to operate the store, with Sustained Window Rate falling as low as 10.6% and one run taking no action in 355 of 523 remaining decision windows. E-Commerce Bench's error analysis **explicitly tests and partially rejects** late-year collapse: growth drops after Q1 (paired t = −3.95) but the closing window is flat rather than deteriorating, and all 90 episodes ended on the calendar or in bankruptcy rather than by going quiet. Its fatal mode is instead **bankruptcy from inventory bought and never sold** (one Qwen3.5-Plus episode fills four stores in January and ends in May), which MerchantBench structurally cannot exhibit because dropshipping holds no inventory.

Two structural explanations, and one caveat that undercuts both. Structurally: (a) MerchantBench gives a free decision window every 12 simulated hours that an agent can silently skip, whereas E-Commerce Bench charges minutes for every tool and requires the agent to *drive* the clock, so passivity does not look the same; (b) MerchantBench scores a single terminal scalar over ~730 windows, so dormancy is invisible until year-end, while E-Commerce Bench's per-dimension diagnostics would surface a disengaging agent as collapsing tool-call counts. The caveat: **E-Commerce Bench reports no SWR-equivalent metric at all**, so it cannot actually confirm or refute MerchantBench's finding — the absence of a dormancy result here is an absence of measurement, not evidence of absence.

**7. One thing E-Commerce Bench settles that MerchantBench could not.** MerchantBench reported coefficients of variation up to 55.1% over three runs and conceded that individual model rankings were weakly supported, leaving open how much of that variance came from its stochastic supplier events. E-Commerce Bench removes environment stochasticity almost entirely — fixed demand model, deterministic negotiation kernel, only renderer wording varies — and **the spread persists**: GPT-5.5 has a std of ¥616k on a mean of ¥702k, and Claude Opus 4.6 ¥266k on ¥258k. **Variance in these simulations is a property of agent policy, not of the environment.** That is a genuine cross-benchmark advance and it strengthens MerchantBench's caution rather than dissolving it.

**8. Method convergence.** Both papers independently conclude that a single financial scalar is inadequate — MerchantBench by adding SWR and a trace-derived failure taxonomy, E-Commerce Bench by adding six ranked capability axes. Both find agents underperform by **disengagement or omission rather than by gaming the metric**, so [[reward-hacking]] is conspicuously absent from both.

## Limitations

The paper has no limitations section; these are stated in scattered caveats or observed:

- **No human or rule-based baseline.** The single largest gap, and the reason the "14×" headline is uncalibrated. MerchantBench's rule-based controller beat 6 of 16 LLM configurations; nothing here tests whether a trivial policy would beat much of this field.
- **One harness, one prompt regime, no harness ablation** — see reconciliation point 5.
- **Five episodes per model, and the paper says so.** Standard deviations frequently exceed differences between adjacent models; BadSpend%'s std exceeds its mean for 15 of 18 models; the AnchorRatio for GPT-5.5 averages three episodes rather than five.
- **The negotiation metrics are rates over a session set the agent chose itself.** CSE⁺ and %Oracle weight no deal by quantity, so they measure per-deal bargaining skill rather than procurement cash saved, and two models compare only if their portfolios carry similar bargaining ranges. Abandoned sessions enter no denominator — **10.9% of sessions were opened mid-year and never returned to** and cost nothing in any metric.
- **Self-evaluation.** This is Qwen Team's benchmark, and its open-weight leader and sole strong performer on the learning axis is Qwen3.8-Max-Preview. The environment is deterministic, open-source, and drawn from Alibaba's own platform data, all of which mitigates; but the tuning of prompts, tool surface, and information-asymmetry design against the authors' own model family is not audited.
- **Tool-call cost is unpriced.** No fee inside the economy charges for a call, so the bill falls on whoever deploys the agent and the primary score is blind to it — the paper flags this and points at the Berkeley function-calling leaderboard as the counterexample that prints dollar cost beside accuracy.
- **Realism boundaries.** One marketplace, one simulated year, one country, a fixed supplier roster, and a fictionalized promotion calendar. Suppliers "remember" via prompt-injected deal history rather than any persistent model state.

## Related

- [[merchantbench-long-term-coherence-shi-2026]] — the direct sibling; see the full reconciliation above. Same company, same horizon, different marketplace, incompatible headline units.
- [[agent-benchmark]] — a continuing rather than episodic benchmark: no terminal state, an accumulated score, 1,000–4,000 agent actions per run.
- [[agent-evaluation]] — the paper's methodological thesis is that one financial scalar hides capability structure; seven axes, and no model strong on all seven.
- [[llm-agent]] — the agent as a going concern rather than a task solver.
- [[outcome-based-reward]] — end-of-year total assets over a 365-day horizon with no intermediate reward, an extreme credit-assignment setting shared with MerchantBench.
- [[reward-hacking]] — notably absent again: the failure mode is omission (unrevised prices, unwithdrawn revenue, repeat orders from known-bad suppliers), not exploitation of the scoring function.
- [[verifier]] — the simulator is the verifier; correctness is an economic outcome, and the deterministic kernel means the verifier is exactly reproducible.
- [[llm-as-a-judge]] — displaced from scoring entirely, but note the LLM *is* used inside the environment as a dialogue renderer over decisions it does not make. That is an unusual and reusable design: keep the LLM for surface realism, deny it any decision authority.
- [[context-assembly]] — the 128k budget with a 120k trigger and 60k release target is the load-bearing harness choice, disclosed to the agent verbatim so retention becomes a decision.
- [[memory-management]] — a 20-note eviction-proof store, used for 2.1% of calls field-wide and never at all by one model. The learning-axis failure is plausibly a memory failure, and the paper says it cannot separate the two.
- [[long-context-llm]] — 1,495 evictions across 90 episodes; the horizon is far past what long context solves.
- [[catastrophic-remembering]] — the inverse pathology also appears: 746 of 2,180 research queries repeat arguments already issued that episode, because nothing charges for asking twice.
- [[tool-use]], [[function-calling]] — 18 tools priced in simulated minutes, which converts the action space into a budget; 71.8% of all calls land in a four-tool daily loop.
- [[agentic-harness-engineering]] — one fixed harness for all 18 models, which MerchantBench's harness-sensitivity result suggests is a stronger assumption than the paper treats it as.
- [[agentic-misalignment]] — adjacent but distinct: fraud here is done *to* the agent by scripted counterparties, not by the agent. The interesting overlap is that the detection signal (concession rate over multiple rounds) is only visible to an agent patient enough to keep bargaining.
- [[qwen]], [[alibaba]] — authoring team and data source; [[glm-5-2]], [[claude-opus-4-8]], [[deepseek]], [[moonshot-ai]] appear in the evaluated field.
- [[react]] — not used here (E-Commerce Bench supplies its own loop), but it is one of MerchantBench's two frameworks and therefore relevant to comparing the two leaderboards.
- [[swe-bench]], [[osworld]], [[gaia]], [[terminal-bench]] — the episodic contrast class the paper positions against by name.

**Concept gaps noted (not created):** `long-horizon-agent` still has no page — flagged by the MerchantBench ingest and now needed by a second source. There is also no page for *automated negotiation* / bilateral bargaining (the paper's core mechanism, with its own literature: Raiffa, Baarslag, Rubinstein), none for *continuing vs. episodic tasks* as an evaluation distinction, and none for *deterministic simulation as an evaluation methodology* — arguably this paper's most transferable contribution. Vending-Bench, Vending-Bench 2, YC-Bench, RetailBench, and TERMS-Bench are all uningested. `hkust` has no entity page (`hku` is the University of Hong Kong, a different institution, and must not be substituted). Fable5, GPT-5.6 Sol, Kimi K3, and Qwen3.8-Max-Preview have no entity pages.

## References

- [arXiv:2608.30730](https://arxiv.org/abs/2608.30730) — Fan, Shen et al., *E-Commerce Bench: Evaluating LLM Agents on Long-Horizon Autonomous Business Operation*, 31 Aug 2026, v1.
- Code: `github.com/QwenLM/E-CommerceBench`
- Environment data: Taobao & Tmall Group (Alibaba Group), desensitized platform logs.
- Sibling benchmark: [[merchantbench-long-term-coherence-shi-2026]] ([arXiv:2607.28956](https://arxiv.org/abs/2607.28956)).
