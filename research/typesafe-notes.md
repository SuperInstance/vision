# TypeSafe / Jev — research notes + experiment design (2026-10-17)
Status: docs deep-read COMPLETE; API adapter + full bench WRITTEN and
ready (`gate-replay/jev_bench.py`); **execution BLOCKED — the key at
/mnt/c/Users/casey/type.txt is rejected by the API** (`401 Cannot
authenticate`, Bearer form confirmed correct; X-API-Key/Token forms
aren't even recognized). Need a fresh key from console.typesafe.ai.
The moment a valid key exists: `TYPESAFE_API_KEY=... python3 jev_bench.py`.

## The API, distilled

- `POST https://api.typesafe.ai/v1/systemone` — `{model, state,
  questions}` → `{model, answers{...}, usage{input_tokens}}`. State is
  ingested once; ALL questions evaluated in parallel.
- Models: `jev-1.14.0` (Oct 14 2026) and `jev-1.13.0`; aliases
  `jev-preview` / `jev-latest`. $0.042/Mtok INPUT ONLY (output free).
  64k state / 32k per question; a header extends to 262k.
- Primitives: **noul** (yes/no → 0–1 probability), **choice** (≤255
  options → choice + full `probabilities` + `confidence`), **score**
  (2–10 ordered levels → probability-weighted score + distance-aware
  confidence), box (image coordinates).
- **Confidence is a FORMULA, not a model estimate** (choice:
  `(p_max − 1/n)/(1 − 1/n)`; score: distance-weighted spread). No
  calibration guarantees claimed; full probabilities always returned so
  alternatives (p_max, p1/p2 ratio) are computable client-side.
- Rate limits per-org (429/529 → exponential backoff; SDKs auto-retry).
  Errors: 400 rule violations, 401 key, 422 validation, 413 >32MiB.

## The three traps when porting our bench

1. **Confidence scale ≠ Ollaya's.** TypeSafe 2-option confidence =
   `2·p_max − 1`: our GATE threshold "conf ≥ 0.6" means p_max 0.8 there.
   jev_bench gates on p(option) directly to stay comparable.
2. **`criteria` shapes differ per primitive** (choice: map; score:
   ordered array). Our cycle-judge questions port as: task_completed →
   choice; claim_supported → **score** (supported < partial < overstated
   is ordinal — distance-aware confidence is a strictly better fit than
   our 3-way choice).
3. **English-primary**: fine for us; noted for any CJK Telegram input.

## Synergy findings with our stack (the exciting part)

1. **Speculative fan-out collapses our whole System-1 layer into ONE
   call per heartbeat.** Gate + stake + injection-screen + triage asked
   together against the same digest — extra questions are ~free (docs:
   "usually has little effect on response time"). This replaces the
   gate-call-then-maybe-judge-call chain; experiment 4 in the bench
   verifies it against single-question runs.
2. **The agent skill exists for coding agents** (`claude plugin
   marketplace add typesafe-ai/skills`) — installable for our whole
   team so mcode/claude/zcode natively reach for Jev when building.
   Their best-practice line mirrors our council: agents "aren't great
   at writing questions" — edit collaboratively, keep thresholds in
   constants.
3. **Cost model works for us**: ~300-token gate digest ≈ $0.013 per
   1,000 gate calls. At 96 beats/day that's ~$0.45/month for the cloud
   gate — the DECISION_URL-in-Actions problem dies if Jev passes the
   bench (zero infra, unlike Ollaya).
4. **Their three-tier confidence pattern is our TRUST routing**: high →
   act; medium → confirm; low → human. Same shape claude proposed; Jev
   gives the calibrated-ish number to drive it.
5. **Jev is not a chat model** — no agent-brain swap; it's the gate,
   judge, and router layer only. Matches our split exactly.

## Experiment matrix (jev_bench.py, ~1–2¢ total cost)

1. GATE: 79 rows (corpus + adversarial) × 2 models × 2 framings
   (choice / noul) — idle-skip, false-idle, latency, cost/1k.
2. JUDGE: the weasel claim (task-framed; choice + ordinal score
   variants — local best was winnow 0.99 task-framed vs 0.57
   claim-framed) + the 10-piece injection set + controls.
3. Comparison vs local stack: winnow:e4b (98% idle-skip @0 false-idle),
   laya (6%), minigate (13% false-idle on adversarial).
4. FAN-OUT: gate+stake+untrusted in one call; verify answers match
   singles and latency barely moves.

## What this unlocks if the bench passes

- DECISION_URL in Actions → point zero-poc's existing v0.4 gate at Jev
  (agent.py's decide() needs only a path/auth tweak) — the cloud gate
  becomes real with zero infra.
- The daily digest's triage, the evening lane's appetite gate, and the
  injection screen all ride the same single fan-out call.
- The fleet gets one shared decision service (the vault proxies it with
  the existing OIDC allowlist pattern).

## EXECUTED (2026-10-17, key working — the `apikey_` prefix was required)

Full results in gate-replay/JEV-RESULTS.md. Headlines: gate 95% idle-skip
@ 10.5% false-idle on the combined set, 0.25s, $0.05/month at 96 beats/day
— cloud gate solved. Weasel claim caught at p=1.00 (best measured).
Injection 7/8 vs laya's 0/6. Fan-out: whole System-1 layer in ONE 0.2s
call. The ordinal claim question PASSED (conf 0.00) — the ledger's
0.5-means-pass principle validated on first live data. Blind-veto stays
in front of any model gate (Jev read past an llm_error retry signal).
Next: thin adapter for DECISION_URL (vault-side), teacher-labels →
Platt-scale laya, question rewrite for claim_supported.
