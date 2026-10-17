# BASELINE — the honest numbers (Sprint 0 D7, 2026-10-17)
Measured from zero-poc git history, 2026-10-04 → 2026-10-17 (12 days),
no gate active, owner-maintenance commits included and attributed.

| metric | value |
|---|---|
| total commits | 129 |
| heartbeat cycles | 57 |
| cycle idle rate (non-substantive) | **98%** |
| substantive commits (any author) | 19 |
| substantive commits by the AGENT itself | **1** (e250e8f5, 2026-10-06) |
| autonomous substantive actions/day | **~0.08** |
| days with ≥1 substantive commit | 4/12 (33%) — all owner-maintenance days |

## Reading

1. **The 98% idle rate is real** — the gate's cost case stands (skip
   almost everything, buy cadence). But the headline metric per law 4
   is actions/day, and it is ~zero. Cadence without a setpoint would
   make the agent *idle 96 times a day instead of 4* (mcode, round 2).
2. **All 13 recent substantive commits are owner maintenance** (the
   v0.3/v0.4/Sprint-0 pushes). The agent's entire autonomous
   contribution since birth is one docs fix. This is the number the
   duty cycle, probes, and TRUST.md exist to move.
3. **The bets now have their yardstick:** bet #1 (memory repair
   reduces idleness) and bet #2 (duty cycle produces self-initiated
   work) are measurable as: autonomous substantive commits/week.
   Baseline: 0.5/week. Falsification thresholds per PLAN.md.

## What was repaired before this measurement

Sprint 0 (2026-10-16/17): gate self-feedback deleted (the latch),
blindness veto, no-op completions write nothing, duty-cycle forced
probe (6h), MEMORY.md purged (9 idle lines), CONTEXT.md working memory,
[unverified] stamping on SHA-less memory claims, issue #1 closed.
Going forward, D3 means idle cycles leave no trace in MEMORY.md — the
corpus can regenerate cleanly once ~2 weeks of repaired cycles
accumulate.
