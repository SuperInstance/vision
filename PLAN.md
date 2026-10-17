# THE PLAN — roadmap of record
Adjudicated by ZCode from Council Round 1 + Round 2 (claude opus 4.5 +
mcode, 2026-10-16). Supersedes the prior three-sprint roadmap. Source
documents: `council/`. Where the advisors disagreed, the decision and its
owner are stated.

## The dream (one paragraph)

A **circadian organism whose activation is gated by calibrated
uncertainty and driven by an endogenous setpoint — earning graduated
trust until Casey stops supervising it because it earned that.** It wakes
every 15 minutes but thinks rarely; it looks even when nothing happened
(forced probes); it declines work it has learned nobody wants; every
memory it writes is cited to a commit or stamped unverified; every
action it takes moves a computable trust score that determines what it
is allowed to do next without asking. One mind, two substrates (neural
on the laptop, linear in the runner — the distillation seam is a
correctness boundary). One body, eventually: a sensing node that knows
whether the room is occupied before it decides to ping anyone.

## The laws (invariants, not aspirations)

1. **The distillation constraint.** A question may only be called
   System-1 if a nine-weight logistic regression can hold it. When the
   laptop model and `gate.json` disagree, rewrite the question — don't
   add capacity.
2. **Cited, not judged.** Every memory write cites a commit SHA or is
   stamped `[unverified, date]`. The probabilistic judge is a secondary
   signal; the artifact is primary.
3. **Duty cycle, not probability.** The drive is `forced_probe =
   hours_since_substantive_work > 6`, latched, reset only by real work.
   Bounded by construction: ≥1 run/6h, ≤4 forced runs/day. (mcode r2
   §1.1 — the original floor formula could never cross threshold.)
4. **Count actions, not wakeups.** The headline metric is substantive
   actions/day and `PRs merged ÷ opened`. Trust compounds on observed
   correct action; skips are invisible and compound nothing.
5. **Authority is a CI artifact.** `gate.py --selftest` runs on every
   push. Red CI withholds `GATE_AUTHORITY`; the gate fail-opens. The
   repo's CI tests the repo's judgment.
6. **Blindness is alarm.** An issue-fetch failure is a deterministic
   `RUN (blind)` veto — never a feature weight, never indistinguishable
   from zero issues.
7. **No self-feedback.** The gate never consumes its own prior verdict.
   (The latch, Finding C, is deleted, not re-weighted.)

## Sprint 0 — Repair the senses (Week 1, day by day)

Days 1–2 ship as one change set (both live in the digest/gate path).

- **D1 — blindness is alarm:** deterministic veto in `gate.py`: digest
  contains `fetch failed`/`unknown` → `RUN (blind)`. Failing-first
  regression row committed.
- **D2 — delete the latch:** drop the `last cycle outcome` line from the
  gate digest entirely; delete `outcome_idle`. Regression row: *1 visible
  open issue + prior gated_idle → RUN* (currently p=0.42 SKIP — fails
  today).
- **D3 — stop writing nothing:** extend the no-op rule to cycles that
  ran and found nothing. 20 idle cycles → zero MEMORY.md commits.
- **D4 — the duty cycle:** `hours_since_work` in `.agent-state.json`,
  monotonic across probes; forced probe latched. Simulated 24h: ≥4
  forced runs, ≤8 total LLM calls.
- **D5 — one memory file:** `CONTEXT.md` (rewritten, never appended;
  `## Facts` table: claim | sha | ts | confidence). MEMORY.md stays as
  an append-only event log the LLM no longer reads. One format — a
  second is a second thing that can be wrong at 3am.
- **D6 — cited-not-judged + purge:** SHA predicate on all memory writes;
  purge the 18 idle lines; close issue #1 (72 comments, ~half self-talk)
  with a pointer to git history.
- **D7 — measure honestly:** re-run extraction on repaired history.
  Report **substantive commits/day with no gate at all** — the number
  that decides whether the gate is worth 98% of anything. Wire
  `--selftest` into Actions (law 5).

**Sprint 0 exit:** all seven regression rows green; honest idle rate and
actions/day written into this repo; CI-withdrawable authority live.

## Sprint 1 — Shadow, stakes, predicates (Week 2)

- Minigate shadow-wired (log verdicts; skip nothing) — but **authority
  comes from D7 + selftest, not shadow logs** (an idle stream cannot
  measure false-idle; mcode r2).
- Stake/commitment tags on intake: `commitment: hard|soft|watch|reject`
  × `blast_radius: low|med|high` — logged from the first signal, and
  **differentially routing** by Sprint 2 (claude r2: logging is step one,
  routing is the payoff; also honor explicit rejections — the NOT-doing
  list).
- Injection screen on issue bodies pre-context (annotate `[UNTRUSTED]`,
  never drop, never a wake-gate).
- `DECISION_MODEL` switched off laya for any server-side gate use.
- Evidence predicate primary; cycle-judge kept as secondary annotation
  on `[unverified]` claims.

## Sprint 2 — Pacemaker, drive, trust (Weeks 3–4)

- Cloudflare Worker pacemaker (same trust domain as the vault): commits
  `last_tick`; time becomes observable from inside the repo.
- Circadian cron: work 08–23 weekdays; **02:17 consolidation job** —
  rebuild CONTEXT.md from git, re-rank, prune the board, write the day's
  digest. No thinking, just sleep.
- Cadence to 15 minutes — gated on Sprint 0's honest number still
  justifying it.
- **TRUST.md** (claude r2 final move): append-only ledger
  `ts | decision | stake | outcome | autonomy_delta`; rolling
  `autonomy_score` ∈ [0,1]; `autonomy_ceiling` per stake
  `{low:0.2, med:0.5, high:0.8}`. Score ≥ ceiling → act unasked; below →
  surface and wait. Auditable: Casey can read exactly why the agent did
  or didn't act. Trust accounting without real work is theater — this
  ships only if Sprint 1 produced real actions.
- **DRIVE.md** (mcode r2 final move): one row per probe:
  `ts | probe# | expected | observed | action|declined | declined_because`.
  The declines are a training set; boredom becomes computed ("declined
  same class 3× running → stop proposing it").

## Sprint 3 — Expectations, memory, fleet pattern (Weeks 5–8)

- **The predictive organ** (claude r1+r2, built from mcode's probes):
  expectations over signals — "issue #3 should have a reply by now;
  silence is signal." Trained on DRIVE.md's expected/observed rows. This
  is what the probes are FOR: six months ≈ 720 rows, the first honest
  corpus for "should this repo be doing X?"
- **Temporal queries** over the Facts table (what did I believe about X
  on date Y) — orthogonal to SHA-trust; ships after CONTEXT.md is real.
- **Vault schema as a versioned, CI-tested repo** (demoted from Sprint 1
  per mcode r2 §1.5 — the gh fallback already mitigates the 404).
- **Fleet pattern:** one spine, N proposers, PRs as proposals. Git is
  already the consensus. No mesh, no gossip, no operator.

## R&D lane (timeboxed, kill criteria written before starting)

- **World-model nightly proposer:** one experiment, two weeks, laptop
  GPU, appetite-gated, opens PRs. Kill: 10 PRs, 0 merges → closed.
  (Scout report: `research/world-models.md`.)
- **Interoceptive node:** ONE sensing Arduino (closest real hardware to
  the "UNO Q" — scout report `research/introception-node.md`), no
  actuators until `autonomy_score` exists and judge accuracy >99% over 8
  weeks (claude's defer wins over mcode's repurpose: the stricter gate
  governs). Second unit stays in the drawer until the first proves out.

## The bets (falsifiable, 30-day clock)

1. Fixing the memory window reduces measured idleness more than shipping
   the gate does. (Substantive commits/day, post-Sprint-0, decides.)
2. The duty cycle produces self-initiated work. (~0 substantive actions
   /day for a month with the probe active falsifies.)
3. Citing a SHA beats judging a claim. (Unsupported-claim rate, predicate
   vs judge, decides.)

If all three fail, the reflex-first roadmap was right — ship the mesh
and reopen the old plan. The bets are the plan's own kill switch.
