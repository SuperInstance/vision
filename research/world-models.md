# World Models for a Git-Native Agent — Research Brief
2026-10-16 · Scope: one timeboxed experiment, laptop-only (RTX 4050 6 GB, shared), nightly "diff proposer" batch job. Kill rule already decided: 10 PRs opened, 0 merged → lane closes.

## 1. What exists today that fits (or doesn't fit) a 6 GB GPU

**DiffusionGemma is real — verified.** Google DeepMind released `google/diffusiongemma-26B-A4B-it` (June 2026, Apache-2.0): 25.2B total / ~3.8B active MoE, encoder-decoder discrete diffusion, ~52 GB BF16; ~37 community quants exist, but even 4-bit (~13 GB) exceeds 6 GB. Real, but not laptop-local. Nearest real things that fit: **Apple DiffuCoder-7B** (open-weight masked-diffusion code model, 130B code tokens) — Q4 ≈ 4–5 GB is borderline on a shared 4050, and it is not served by Ollama (HF/own runtime; 6 GB runtime unverified). LLaDA-8B-class dLLMs hit the same size wall. Mercury-class dLLMs are closed API.

**JEPA.** Official code: `facebookresearch/jepa`. I-JEPA is ViT-H (~632M; paper used 16×A100); V-JEPA 2 ships 300M and 1B checkpoints (pretraining cost: thousands of GPU-days). Inference fits 6 GB trivially — but these are image/video encoders. For repo state you would train your own tiny JEPA-style predictor (<10M params, PyTorch, CPU-class). Memory is not the blocker; data is.

**Repo-dynamics prior art: brand-new and big-lab-scale.** "Software World Models: From Consequence Prediction to Code Evolution" (arXiv 2610.04940, Oct 2026) learns change-conditioned dynamics of code; DeepMind's Code World Model (32B) predicts execution consequences in representation space — JEPA-flavored but far beyond 6 GB. Practical "bot opens maintenance PR" prior art is cron-driven (Dependabot/Renovate) or rule-triggered (Repairnator: failing-build detector → patch → PR, 2017–19 [from knowledge — verify]). No credible published "world model over repo metrics" at single-repo scale. That is both our gap and a warning.

## 2. "Nightly diff proposer," by feasibility

**(a) Drift detection — do first.** Nightly feature vector from git: dependency lag, test count/pass rate, doc coverage, CI duration, issue-backlog shape, churn. With 116 commits, classical methods (z-scores, EWMA, isolation forest, Holt/ETS forecast error) will beat any learned model. Output: anomaly score + attribution (which metrics diverged).

**(b) Signal-gated generation — run in parallel.** Anomaly arms qwen2.5-coder:7b (Ollama) to draft exactly one maintenance PR with evidence in the body. The appetite gate made concrete: the predictor decides *when* generation is worth GPU time; the LLM decides *what*.

**(c) True JEPA-style latent dynamics — stretch goal only.** A tiny predictor fits 6 GB easily, but 116 time points is ~100× too few; you would need to bootstrap granularity (per-file, per-CI-run rows → ~10³–10⁴ samples) and regularize hard. Attempt in week 2, and only if (a) shows signal. The right architecture at this scale matches the "graphs and quilts" intuition: a quilt of small per-metric forecasters wired into a graph — not one monolithic world model.

## 3. Experiment design (2 weeks, nightly)

- **Data**: backfill the 116-commit history into ~20 nightly metrics on day 1; keep snapshotting. Say it plainly: 116 points is tiny; all results are directional, not statistical.
- **Model ladder**: L0 fixed thresholds (Dependabot-parity baseline) → L1 classical time-series/anomaly → L2 tiny learned forecaster (MLP over metric windows; the 4050 is optional — CPU suffices). A diffusion/JEPA component enters only as an encoder for L2, if at all.
- **Trigger**: forecast error > k·σ on ≥2 metrics → arm the proposer. Cap: 1 PR/night.
- **Output**: one concrete action with evidence, e.g. "bump dep X (stale 214 days)", "add test for module Y (coverage 62→41% since abc123)".
- **Kill/success**: keep the decided rule (10 opened, 0 merged → close). Also log merge rate and a cron-baseline arm: the live question is whether predictive gating beats plain cron. If merges are all Dependabot-class actions, the world-model layer added nothing — say so and stop.

## 4. What a predictive model adds that System-1 cannot

System-1 classifiers score the present: is this prompt worth the loop, is this diff acceptable. A world model scores *trajectories*: where the repo is heading and whether an action bends the curve back toward the healthy manifold. It buys lead time (flag drift before thresholds trip), attribution (which metric diverged first and when), and a target state to propose diffs toward. In ReAct terms: System-1 is reflex; a small world model is the beginnings of a critic over time — cheap planning instead of expensive reaction.

---
*Unverified at publication: DiffuCoder-7B runtime on 6 GB; Repairnator specifics; any L2 benefit at n≈130. All other claims checked 2026-10-16 (Hugging Face model card; arXiv 2610.04940; facebookresearch/jepa).*
