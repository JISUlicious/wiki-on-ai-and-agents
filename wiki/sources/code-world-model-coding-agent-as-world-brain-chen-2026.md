---
title: "Code World Model: Coding Agent as World Brain"
type: source
created: 2026-09-05
updated: 2026-09-05
sources: []
arxiv_id: "2608.25927"
authors:
  - Yiwen Chen
  - Guosheng Lin
  - Chi Zhang
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
  - multimodal
---

# Code World Model: Coding Agent as World Brain

**Yiwen Chen, Guosheng Lin, Chi Zhang** (AGI Lab, Westlake University; Nanyang Technological University) — [arXiv:2608.25927](https://arxiv.org/abs/2608.25927), **cs.CV**, 26 Aug 2026, v1. Project page: `buaacyw.github.io/cwm/`.

> [!note] Name collision
> This is **not** Meta FAIR's Code World Model. The paper cites FAIR's CWM (ref [92]) as related work running in the *opposite* direction — mid-training an LLM on Python execution traces so the model learns to simulate code execution, "using world modeling to strengthen code generation." This paper uses code generation to strengthen world modeling.

## Summary

The complaint is epistemic and sharply put. Video world models "learn dynamics from visual observations, which reveal **outcomes** rather than the underlying knowledge, rules, and mechanisms governing world evolution." The gameplay case makes it concrete: every frame of training data was produced by executing fixed game code, **and that code is discarded after rendering** — the model must re-infer the rules from their sparse visual consequences. Worse, many state changes happen off-screen or over horizons far longer than a video model's context (training windows are typically under a minute, while causal processes in a world unfold over in-world days or years). Scaling video data adds more observed outcomes and no direct evidence of mechanism.

The running example is a fantasy world where a player assassinates a city's ruler and leaves. Succession, local order, trade, faction relations, and NPC beliefs all keep evolving *while the city is off-screen*, and become visible only on return. A system must determine "not only how the city should look upon return, but also what happened during the player's absence, why it happened, which consequences remain active, and how the current situation will shape future evolution." No amount of next-frame prediction produces that.

So the paper performs a **functional decomposition**, splitting world state into two coupled halves:

`S_t = (S_t^exe, S_t^vis)`

| | Owner | Update |
|---|---|---|
| **Executable state** `S^exe` — world program, entity attributes, rules, relations, event history | coding agent + code | `S_{t+1}^exe = T_AC(S_t^exe, A_t)` — deterministic, agent-revisable |
| **Visual state** `S^vis` — appearance, motion, fine dynamics that must stay consistent | video model | `S_{t+1}^vis ~ G_θ(S_t^vis, S_{t+1}^exe)` — generative |

This maps cleanly onto the conventional world-model factorization `S_{t+1} ~ p(S_{t+1}|S_t,A_t)`, `O_{t+1} ~ p(O_{t+1}|S_{t+1})`: **code is the transition function, the video model is the observation function.**

**Two computational regimes.** The coding agent handles *sparse but semantically complex* decisions — interpreting new events, reasoning about long-term consequences, selecting mechanisms, revising how the world operates — and translates them into reusable code. Code handles *dense, repetitive, deterministic* work — positions, numeric attributes, collisions, cooldowns, chained triggers — without invoking the model at every step. "Code is therefore neither permanently frozen game logic nor a controller independent of the agent; it is the coding agent's **continuously maintained, executable, and modifiable extension**." Its execution rate is decoupled both from the agent's reasoning frequency and from the video model's frame rate.

**The proxy is the contract between the halves.** Two obvious interfaces both fail. Structured text is elegant and modality-free but current video models "cannot yet reliably translate dense, constantly changing textual specifications into observations with precise frame-wise positions, spatial relations, and camera trajectories" — and a description precise enough would be hopelessly token-inefficient. Building an explicit 3D world gives spatial control but costs production-quality geometry, assets, animation, materials and rendering, and "prematurely constrains the final visual realization." The **proxy** is the middle: a coarse programmable representation of entity positions, poses, trajectories, spatial relations and camera motion, assembled by the agent from a small library of reusable primitives (each "a few lines of code") and rendered by a **lightweight deterministic compiler** into a *proxy video*.

The design constraint that gives the framework its character: **every control signal must be traceable to world state, keeping the state-to-condition path white-box to the coding agent.** No learned 3D generator may fill in the proxy, "because that would introduce a mapping that the coding agent cannot directly inspect or control."

## Key points

**Condition bandwidth is the central design trade-off**, and the binding constraint is the *agent's* competence, not the video model's. "If the proxy encodes a character's articulated motion at the joint level, the coding agent must also control that character's joint trajectories during inference. Current coding agents still struggle to implement such joint-level motion control reliably." So proxy design is framed as **constructability vs grounding strength** — "the minimum sufficient state that the current observation must obey, while leaving final appearance and fine-grained dynamics to the video model." The training-data pipeline follows the same rule: skeletal pose and detailed 3D models are *deliberately excluded* from game proxies because coding agents cannot reproduce them at inference.

**Proxy conditioning is nearly free.** The proxy runs at one quarter of the target resolution per spatial dimension (336×192 against 1344×768), so it adds **1/16 as many visual tokens** as the target video — negligible inference overhead even at frame-wise density.

**The proxy is a prompt, not a fixed pipeline stage.** "In a complete Code World Model, the coding agent itself determines whether to enable, disable, or adjust the proxy" — omit it when text suffices, activate it when an observation must follow exact positions or interaction topology, and adjust which entities, state types, spatial granularity, resolution and condition frame rate are represented. Autonomous switching is *not* implemented; the paper instantiates one fixed proxy design.

**Data pipelines** (the paper's second contribution, and the one with the most transferable value):

| Source | How pairs are built | What it avoids |
|---|---|---|
| **Game** (5.6 h, 157 takes) | target video and runtime state recorded synchronously from the same execution; code reads structured runtime annotations to produce pixel-wise, instance-level proxy annotations with identity preserved across frames | complete 3D models, textures, materials are never retained; the same recording can be **recompiled into proxies of different coverage without recollecting video** |
| **Real** (KITTI-360, proof of concept) | calibrated poses + accumulated semantic 3D reconstruction + 3D object annotations compile the proxy **offline**; static geometry projected to metric depth and normals; capsule skeletons for pedestrians, wireframes for vehicles, low-poly vegetation, coarse extents for buildings; shared depth buffer preserves occlusion | **no action labels and no camera-pose annotations** — action and camera effects are already *in* the proxy video, which the paper argues substantially lowers the annotation burden for real video |

That last point is the real methodological claim: because consequences and camera motion are represented *pictorially* in the proxy rather than as separate labels, the interface sidesteps the ambiguity of defining action granularity in unconstrained real video.

**Implementation** — everything here is a single small fine-tune:

| | |
|---|---|
| Backbone | MiniMax-H3 Ref2VA (official checkpoint) |
| Adaptation | rank-128 [[lora\|LoRA]] across all 50 transformer blocks → **~596M trainable parameters** |
| Data | 9,420 five-second clips sampled at 2-s intervals from 157 takes (~5.6 h; Fig. 4's caption says "five hours of GTA V gameplay") |
| Target | 124 frames, 1344×768, 24 FPS |
| Proxy condition | 124 frames, 336×192, fixed-log depth + categorical semantic-ID maps; encoder sees 11 frames at offsets 0, 12, …, 120 |
| Compute | 8× NVIDIA H800, AdamW, global batch 8, wd 0.01, grad clip 1.0, BF16, [[flash-attention-3-shah-2024\|FlashAttention-3]], gradient checkpointing |
| Schedule | 3 epochs = **3,534 steps**, 100-step warmup, cosine 2e-5 → 1e-6; audio loss disabled |
| Coding agent | **GPT-5.6 Sol** |
| First-frame anchor | GPT Image 2 renders a 1536×864 appearance anchor from the first proxy frame + an agent-written prompt, resized to 1344×768 and fed to both the multimodal encoder and the Ref2VA appearance-reference branch |
| Long video | overlapping 124-frame windows, 34-frame overlap, advancing 90 frames per call; the same anchor is re-supplied each window for global identity |

**The coding agent does not build the world from scratch.** It is given "existing game-engine code that supplies basic player controls, collision handling, and a runtime update loop, together with the same proxy primitives used in training-data construction," and it works by "using existing AAA game scenes and gameplay logic as templates, combining, extending, and rewriting them." The authors are explicit that "building AAA-scale scenes and complex interaction logic entirely from scratch remains difficult for current coding agents."

## Is the *code* the world model, or the *video model*?

**The code is the world model; the video model is the observation model.** That is the paper's whole architectural claim, and it maps exactly onto the classical factorization: `T_AC` is the transition function, `G_θ` is the emission. The paper's own one-line summary — "code determines **what happens** in the world and which consequences persist, while the video model determines **how these outcomes are visually realized**" — is the cleanest statement of it.

But the honest answer needs three refinements, and each one matters for how this page relates to [[world-model]].

**1. It is not "code" so much as the agent–code loop, with the LLM's pretrained knowledge as the prior.** The title says *coding agent as world brain*, not *code as world model*. Code alone is frozen game logic; what makes it a world *model* is that a language model with broad world knowledge decides which mechanisms should exist, when a mechanism must change, and what an event implies three consequences downstream. The world knowledge lives in the LLM; code is the substrate it writes that knowledge into so the knowledge can execute at high frequency without a model call. Take away the agent and you have a game engine; take away the code and you have an unaffordable per-tick LLM.

**2. The video model is not a pure renderer — it owns a slice of dynamics.** The proxy *deliberately* underspecifies: it "retains only the coarse structure and interaction state that the current observation must obey," and "the video model synthesizes **local motion and physical interactions** from its learned priors." So fine-grained physical realization — how a strike connects, how cloth moves, how a body responds to contact — is dynamics, and it lives on the video side. `S^vis` is a real part of the state that must stay consistent, not a throwaway image. The division is better described as **causal/persistent dynamics in code, perceptual/transient dynamics in the video model**, with the proxy as the contract.

**3. It is an explicitly *anti-latent* world model, which puts it in direct tension with the rest of this wiki's world-model cluster.** [[world-model]] records the 2026 trend as "toward **latent** world models (predict compact state, not pixels/tokens)," with [[learn-from-your-own-latents-korchinski-2026]] arguing latent prediction is exponentially more data-efficient than token prediction. Code World Model goes the other way: state is **symbolic, explicit, inspectable, editable, and executable**, and the paper forbids any learned mapping in the state-to-condition path precisely so the agent can inspect it. Persistence, the problem [[latent-spatial-memory-for-video-world-models-wang-2026|Mirage]] solves with a latent spatial memory and that others solve with retrieved frames or event-aware memory, is here solved by *just storing it in a program variable*.

The paper's own positioning against the video-world-model line is the crispest framing: those systems' "central modeling object remains the generated observation sequence: scene-specific state and interaction outcomes are primarily carried by visual history, latent context, or visual memory." CWM moves the state out of the pixels and into a program, and hands the video model only what it needs to render this frame.

**One caveat the paper itself raises.** The state-to-condition problem does not go away if the two models are merged: "Even if they are unified into an omni-modal network, continuously changing external world state maintained by code must still be encoded and supplied to the visual generation process. [[parameter-sharing|Parameter sharing]] may simplify communication between components, but it does not remove the state-to-condition problem." So the contribution is the *interface*, not the two-model implementation.

## Where it breaks

This is a position paper with a prototype, and it should be read that way.

- **There are no quantitative results.** Not one benchmark, metric, baseline number, or ablation table appears in the paper. §4.2 is qualitative — five prompts, six frames each — plus a pointer to comparison videos on the project page, and the comparisons explicitly "do not consider inference latency." Every claim of superiority over action- or camera-conditioned video world models rests on demo footage.
- **The authors state the two limitations plainly.** (1) Compute-limited: small training scale, limited generation quality, **no autoregressive real-time generation implemented**. (2) "Current coding agents still struggle to implement highly complex game mechanisms reliably from scratch," so the prototype "does not demonstrate autonomous construction of a complete open-world game or simulator."
- **The headline capability is the least demonstrated one.** The motivating example is off-screen consequence propagation over long horizons — succession, faction shifts, NPC beliefs evolving while the player is away. Nothing in the experiments tests it. What is demonstrated is that a LoRA-tuned video model follows proxy specifications for positions, trajectories, layout and camera in *simple* interactive worlds.
- **The agent is scaffolded.** Given an engine (controls, collision, update loop), the training-time primitive library, and AAA scenes as templates. The gap between "combining and rewriting templates" and "governing an open world" is the entire thesis and is not measured.
- **Long-video generation is windowed stitching, not a persistent visual state.** 124-frame windows with 34-frame overlap and a re-supplied appearance anchor is a continuity heuristic; the paper does not measure drift, identity preservation, or how far it holds.
- **Real-world evidence is one dataset and offline-only.** KITTI-360, with poses and 3D reconstruction available at data-construction time. It shows proxies *can* be compiled from an existing reconstruction; it does not show a coding agent maintaining a real-world state.
- **Minor inconsistency**: Fig. 4's caption says "only five hours of GTA V gameplay," while §4.1 reports "approximately 5.6 hours of source video" across 157 takes. The game is named only in the caption.
- **The proxy's ceiling is a moving target set by coding-agent competence.** Joint-level articulation is excluded today because agents cannot control joint trajectories; complex objects fall back to a bare bounding box with text supplying identity. Whether the interface stays useful as agents improve — or becomes an unnecessary bottleneck — is the paper's own open question ("as video model control improves, the proxy may become sparser or disappear").

## Related

- [[world-model]] — the concept page this most directly extends and partly contradicts. That page frames the 2026 trend as latent state prediction; this paper argues the opposite for open-ended interactive worlds, keeping state explicit, symbolic and executable precisely so it can be inspected and revised.
- [[latent-spatial-memory-for-video-world-models-wang-2026]] — Mirage solves long-horizon 3D consistency with a *latent* spatial memory; CWM solves the same persistence problem with a *program variable*. The clearest same-problem/opposite-substrate pair in the wiki.
- [[learn-from-your-own-latents-korchinski-2026]] — the data-efficiency case for latent prediction, and therefore the strongest argument against CWM's explicit-state route. Worth reading against each other.
- [[infinite-worlds-versatile-interactions-gao-2026]] — LingBot-World 2.0 is named in §2.1 as "most closely related": a causal video generator augmented with pilot and director agents for semantic actions and mid-rollout intervention. CWM's critique is precise — those agents *supply conditions to* a visual rollout, whereas here the agent "directly participates in the executable evolution of the world."
- [[qwen-agentworld-language-world-models-for-general-agents-qwen-2026]] — language world models for agents; same instinct (language/symbols carry world state) without the visual-realization half.
- [[can-llm-agents-infer-world-models-menaged-2026]] — the inverse task: *inferring* a world model (as an automaton) from interaction. CWM assumes the agent can *author* one; that paper measures whether it can discover one.
- [[agentic-world-modeling-foundations-capabilities-laws-chu-2026]] · [[cosmos-3-omnimodal-world-models-for-physical-ai-nvidia-2026]] · [[looped-world-models-facemind-2026]] — the rest of the wiki's world-model cluster; Cosmos 3's omnimodal unification is exactly the merged architecture §3.2 argues would still face the state-to-condition problem.
- [[code-generation]] · [[codeact-wang-2024]] · [[code-as-policies-liang-2022]] · [[code-as-harness]] · [[code-as-agent-harness-ning-2026]] — the "code as the agent's executable medium" lineage. CWM is that idea pointed at world *state* rather than at actions or the harness: code as the substrate an agent writes its world knowledge into so the knowledge can run at high frequency.
- [[context-as-an-environment-scroll-lin-2026]] — a striking structural rhyme: Scroll makes the *session* an executable environment with a persistent Python kernel; CWM makes the *world* one. Both replace a learned/compressed representation with a program the model writes and re-executes.
- [[voyager-wang-2023]] · [[library-learning]] · [[dreamcoder]] · [[lilo-grand-2023]] — agents that accumulate reusable executable skills; CWM's agent–code loop is the same accumulation applied to world mechanisms rather than to agent capabilities.
- [[embodied-agent]] · [[vision-language-action-model]] · [[from-foundation-to-application-vla-wu-2026]] — the motivating downstream use: real-video priors would let generated environments approach an embodied agent's deployment distribution for training and evaluation.
- [[masked-visual-actions-alzayer-2026]] — the counter-position on interfaces: one video model as both forward and inverse dynamics through a *pixel-space* control interface. CWM instead keeps control symbolic and compiles it *down* to pixels.
- [[agent-world-real-world-environment-synthesis-dong-2026]] · [[envharness-awakening-static-worlds-huang-2026]] — environment-synthesis neighbours; the shared motivation is generating training environments for agents.
- [[multimodal-llm]] · [[minimax-m2-series-minimax-2026]] — MiniMax-H3 Ref2VA is the video backbone.
- [[lora]] · [[flash-attention-3-shah-2024]] — the adaptation and kernel used; rank-128 LoRA over 50 blocks, ~596M trainable parameters on 8× H800.
- [[llm-agent]] · [[tool-creation]] — the coding agent writes and revises its own mechanisms, which is tool creation aimed at world dynamics.
- [[openai]] · [[nvidia]] — GPT-5.6 Sol as the coding agent, GPT Image 2 for the first-frame anchor, H800s for training and inference.

> [!note] Concept gaps this page could not link
> No wiki page for **video world model** / **video generation** as a concept (the wiki has several video-world-model *sources* but no concept page to hang them on — this is the most-needed gap), **diffusion transformer**, **program synthesis** (the WorldCoder / probabilistic-program-induction line the paper positions against in §2.3), **generative 3D world**, or **neural rendering**. KITTI-360, GTA V, MiniMax-H3 and GPT-5.6 Sol have no pages.

## References

- [arXiv:2608.25927](https://arxiv.org/abs/2608.25927) · project page `buaacyw.github.io/cwm/`
- MiniMax-H3 (Ref2VA backbone); GPT-5.6 Sol (coding agent); GPT Image 2 (appearance anchor); KITTI-360 (real-world proxy pairs); LoRA; FlashAttention-3.
- WorldCoder; Curtis et al. (probabilistic-program POMDP estimation); Dainese et al. (LLM-generated Python transition models with MCTS); Lehrach et al. (program functions for general game playing) — the code-as-world-model prior line, §2.3.
- FAIR CodeGen Team — the *other* Code World Model: mid-training an LLM on execution traces so it simulates code, i.e. world modeling in service of code generation rather than the reverse.
- HunyuanVideo, Wan — the diffusion-transformer video generators underpinning the field; LingBot-World 2.0 — the closest interactive video world model.
