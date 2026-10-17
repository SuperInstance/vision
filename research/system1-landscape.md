# The System-1 Decision-Model Landscape (October 2026)

Scope: small, fast, confidence-calibrated models that gate/route/judge expensive LLMs. Researched 2026-10-16 via web search + primary-source fetches. Numbers are as reported by their sources; verification flags noted inline.

## The headline: a new model category crystallized in the last 6 weeks

**TypeSafe Jev** (launched 2026-09-15; founder Diogo Almeida, ex-OpenAI). Non-generative "System One" model: unstructured state in, typed probabilistic decisions out. Parallel sampler (all options scored in one query, max cardinality 255), trained via RLCD (RL for Calibrated Decisions). 70–500 ms e2e, claims 40–200x faster than comparable LLMs; $0.042/MTok input, output tokens free; claims 0% type errors by construction. No public accuracy benchmark on their blog. Sold as early-access API; on OpenRouter (Jev 1.14 / Jev Router); PydanticAI docs integration. *Steal: the "typed schema in, distribution out" API shape — it makes gates unit-testable.*

**OpenAI Decisions API ("Luna")** — announced DevDay 2026-09-29, public beta 2026-10-06. Predefined answers + confidence in ~150 ms. **anth.us critique (2026-10-01)**: on 3,600 ProofWriter problems, Luna's 99%+ confidence answers were right only 68% of the time (ECE 0.32 vs Jev's 0.03–0.04; confidence AUROC 0.66 vs 0.85); 10% answer instability on re-ask (Jev 2–3%); coin-flip accuracy at 5 inference steps while Jev holds 81–89%. *Steal: their audit protocol — consistency re-ask + calibration-vs-labels before trusting any gate.*

## The open ecosystem exploded (all post-Jev, mostly Qwen3.5 fine-tunes)

- **Together tev1** (`github.com/togethercomputer/tev1`, MIT code). Qwen3.5-4B + LoRA, 37.8K examples, trained for **$17**. Picks from 2–24 lettered options; returns logprobs but repo warns "logprobs are preferences, not calibrated confidence." Reported 880/1000 decisions, 300/300 policy — dev-set counts, not held-out accuracy (repo admits this). *Steal: the $17 recipe — task-specific gate clones are nearly free; our stack should treat gates as cattle, not pets.*
- **Laya** by Convai Innovations (HuggingFace `convaiinnovations/laya`, Apache-2.0). 421M-param non-autoregressive decision model, runs in-browser via ONNX. *Steal: 421M is small enough to gate at the edge/CLI with zero network.*
- **OpenDecider** (HF blog 2026-09-27, Apache-2.0, `pip install opendecider`). Family: nano (~400M Ettin encoder, 16 ms, ECE 0.092) through large-td (80B-A3B, ECE 0.083). Distilled from Qwen3-235B + DeepSeek V4.1 Flash with **teacher temperature scaling on held-out labels before averaging** — the calibration-transfer trick. nano beats Laya on typed-decisions (0.796 vs 0.766) and Jev on confident-half automation (94–95% vs 88%); Jev still wins Laya's own battery (0.774). <$30 training cost, Jev-API-compatible server, Ollama GGUF builds. *Steal: OpenDecider-nano as a drop-in local fallback with a compatible API to Jev — vendor-proofing.*
- **Kev** (0.8B/4B/9B, Qwen3.5-based, "closest architectural replica" per DataCamp); also decider-2b, this-that-model-1.0, Nimble-9B, GroundingJev (visual), decisionbridge. Curated lists: `yibie/awesome-jev`, `v-modal/awesome-jev-tools`. *Steal: this-that-model-1.0 — a trivially cheap first stage for cascades (see below).*

## What the benchmarks actually say

**arXiv:2610.02267** (2026-10-01, "Fast Models, Slow Evidence") — the paper that matters most for us: paired eval of **Laya vs Jev across 11 agent decision points** (7,283 cases + 6,640 robustness variants: routing, tool selection, injection guard, RAG relevance, groundedness, PII, safety). Results:
- Jev wins 9/11 (overall 76.7% vs 63.1%); biggest gap on large label sets (81% vs 35% — Laya's shared 192-token option budget truncates options).
- **Both are at chance on model routing** (RouterBench): Laya AUROC 0.48; Jev answers "cheap" 99.5% of the time. Zero-shot routing is unsolved.
- Laya order-sensitivity: reversing options flips 30% of decisions; 26% of its ≥0.9-confidence answers are wrong. **Per-domain Platt scaling fixed calibration: RAG ECE 0.22 → 0.027.**
- Safety: Laya passes 81.5% of harmful requests, PII AUROC 0.41 (inverted). Don't use it as sole safety gate.
- **Gate economics: RAG gating saved ~nothing (base rates); a safety pre-screen saved 4.3% net once its own cost was counted, not the 23.9% originally claimed.** The authors self-audited and corrected four such deployment-claim errors. *Steal: their self-audit discipline — always net out the gate's own cost against base rate of the event it gates.*

**arXiv:2610.00346** (decision gates vs trained classifiers vs LLMs). Jev: 0.732 workflows / 0.913 intents, 138 ms, $0.0079/1k decisions. Kev-9B statistically indistinguishable from Jev (0.712). Qwen3.6-27B with option-loglikelihood readout matches Jev (0.733/0.912). **With ~labeled traffic, boring classifiers win: fine-tuned BERT 0.940 and embeddings+LR 0.948 vs Jev 0.913, with ECE 0.017.** Decision models win only zero-label / changing-label-set regimes. Jev's out-of-scope rejection is weak (accepts 31% of OOS at a 5%-tuned threshold). **A cascade (this-that-model-1.0 → Jev) matched Jev accuracy at 0.43x cost.** *Steal: three-tier rule — labeled? fine-tune a classifier. Unlabeled but stable? decision model. And cascade the gate itself.*

**arXiv:2609.37647** — Jev strong zero-shot on sentiment/short scoped decisions. **arXiv:2610.03935** ("General Decision Models") — decision models excel when decisions are resolvable from provided evidence, weaken otherwise.

## Routing (the pre-Jev lineage)

- **RouteLLM** (LMSYS/ICLR 2025): 95% of GPT-4 quality at 26–48% of cost (paper: >2x savings; one 2026 roundup claims 85% on MT Bench — unverified). Still the open-source routing reference.
- **FrugalGPT** (2023): cascade headline 98% cost reduction. Successor: Dekoninck et al., ICML 2025 — unified routing+cascading theory; cascade-routing hybrid dominates when quality/cost estimators are accurate.
- **vllm-project/semantic-router** (~6.2k stars, trending Oct 2026): programmable policy-driven decision layer over model pools; ships its own open routing models ("Decision" 1.0–3.0, "Vela"); semantic caching and hallucination-detection endpoints. *Steal: policy-as-config for gates, and their Workload-Router-Pool separation.*
- **aurelio-labs/semantic-router**: the established embedding-based route matcher; redhat-et/semantic_router routes OpenAI-compatible requests by semantics.
- Managed routers: **Not-Diamond** claims 30%+ savings / 5%+ accuracy gains (vendor numbers, unverified); Martian publishes little.
- Community: a Reddit pilot reports Jev-based routing saved 33.2% vs premium, but a fixed mid-priced model was comparable — **could not verify (Reddit content not fetchable); directionally matches the sys1-eval "routing is at chance" finding.**

## Small-model judges

Production guidance converged on: 30–50-example expert-labeled calibration sets before trusting a judge (Comet, Feb 2026), continuous human-correction alignment loops (LangChain, Mar 2026), and explicit bias mitigation (arXiv:2510.12462, 6 judge models). I found **no published production calibration numbers for small judges specifically** — the sys1-eval paper's Laya findings (26% of high-confidence answers wrong; confidence field inconsistent) are the best hard data. *Steal: treat our gate's confidence as a feature requiring Platt/isotonic recalibration per decision type, not as ground truth.*

## Idle gating of autonomous agents

Closest work: veelenga's "Autonomous agents that sleep" (Jun 2026) — event-driven sleeping instead of poll loops; agent.ceo on self-pacing loops; OpenAI Codex issue #32221 requesting async wake/sleep for long-running agents; heartbeat/work-queue patterns. **All are event-schedulers or timers. Nobody publishes a confidence-calibrated System-1 classifier as the wake gate, and nobody publishes idle-gating savings numbers.** The sys1-eval base-rate warning (gates save little on rare events) is the only economics data point, and it's about RAG gating, not idle gating.

## What the field agrees on

1. Typed decisions + calibrated probabilities is a real new primitive (Jev, OpenAI Decisions API, OpenDecider, tev1 all converged in 6 weeks).
2. Calibration is the differentiator — Jev ECE 0.03–0.04 vs Luna 0.32; RLCD and temperature-scaled distillation are the two known ways to get it.
3. Decision models are good at: tool/intent selection, injection screening, groundedness, short scoped judgments. Bad at: zero-shot model routing (chance-level, both leaders), RAG skipping under realistic base rates, PII, sole-safety duty.
4. With labels, a fine-tuned classifier still beats them (0.94 vs 0.91) with better calibration (ECE 0.017).
5. Cascading gates compounds: this-that → Jev hit equal accuracy at 0.43x cost.
6. Gate cost must be netted against event base rates — several published savings claims shrank under audit.

## What nobody is doing yet (the opportunity list)

1. **System-1 idle gating with published economics.** No one ships "small model decides whether to wake the big model" for autonomous agents — sleep patterns are timer/event-based. The base-rate math that killed RAG-gating savings has never been run for idle gating, where base rates arguably favor it (most wakeups are noise).
2. **Decision-log versioning / git-native gates.** Waehner mentions "log decisions, retrain" in passing; nobody treats gate decision logs as versioned, reviewable artifacts tied to model checkpoints — natural territory for a git-native agent.
3. **Self-auditing gates as a product feature.** The self-audit methodology exists only in one paper. No gate ships with automatic "net savings after my own cost" accounting or drift alarms on decision distributions.
4. **Per-domain recalibration tooling.** Platt scaling took Laya's RAG ECE from 0.22 to 0.027 — nobody productizes post-hoc recalibration per decision point.
5. **Monitoring drift in the gate itself.** Everything benchmarks Day-1 accuracy; nobody publishes how decision-model calibration decays over weeks of traffic — the exact question our drift-probe work answers.
6. **System-1 → System-1 cascade routing** beyond one data point, and small-judge production calibration numbers generally.

## Verification flags

- Reddit-derived numbers (33.2% routing savings; r/hermesagent Jev latency post) — content not fetchable, unverified.
- Vendor claims: Not-Diamond (30%/5%), RouteLLM 85%-MT-Bench and "42% enterprise savings" roundups, OmniRoute prominence — single-source, unverified.
- tev1's 880/1000 is a dev set by the repo's own admission.
- GitHub star counts from single page fetches (vllm semantic-router ~6.2k).
- No public model named "winnow" found in the ecosystem — appears to be our internal naming (our local pair "laya/winnow" maps to Convai's Laya + internal).
- arXiv:2610.02267 and 2610.00346 figures come from fetched HTML preprints; cite the versions read (v1).

## Primary sources

TypeSafe blog (typesafe.ai/blog/introducing-system-one-models-and-jev); anth.us/articles/openai-decisions-api-preview; arxiv.org/abs/2610.02267, 2610.00346, 2609.37647, 2610.03935; github.com/togethercomputer/tev1; github.com/vllm-project/semantic-router; huggingface.co/blog/manjunathshiva/opendecider-beats-laya-and-jev; huggingface.co/convaiinnovations/laya; kai-waehner.de; veelenga.github.io/autonomous-agents-that-sleep; thenewstack.io/openai-decision-api-luna; community.openai.com Decisions API beta thread.
