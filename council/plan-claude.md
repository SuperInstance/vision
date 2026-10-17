# Plan — Claude Opus 4.5 (Round 2)

## 1. Where the Synthesis is Wrong

**A. Conflating tonic floor with predictive model.**

The synthesis says my "meaningful absences" insight is "what the tonic layer *becomes* once a clock exists." No. A tonic floor (`p(work) = max(p_exogenous, 0.03 × hours_since_work)`) makes the agent wake up more often. A predictive model notices when reality diverges from expectation — "issue #3 should have a reply by now; silence here is signal." These are different organs:

- Tonic floor: "I haven't worked in 12 hours, wake up and look"
- Predictive model: "I expected Casey to respond by 14:00; she didn't; that's unusual"

The tonic floor cures monotonic decay. The predictive model cures obliviousness to meaningful absence. Mcode's floor is correct and necessary. But it doesn't give the agent *expectations*, only *restlessness*.

**B. Misattributing the Arduino kill.**

The synthesis says both advisors "independently repurposed the Arduino from reflex-actuation to sensing." I said **kill the Arduino entirely** until trust milestones are met — not repurpose, defer. Mcode said repurpose. The synthesis credited us both for sensing; I explicitly said no physical periphery until self-audit passes 8 weeks at >99%.

**C. Stake tags adopted as logging, not behavior.**

The synthesis adopted "stake/commitment tags logged from day one" as a calibration dataset. That misses the behavioral point: a high-stakes decision (blast radius = production outage) should route differently than a low-stakes one (blast radius = stale comment). Logging is step one; differential routing is the payoff. The synthesis captured the logging, not the routing.

**D. "Queryable temporal memory" is not the same as "evidence-bound memory."**

The synthesis adopted both mcode's SHA-citing and my temporal queries as if they're the same reform. They're orthogonal:

- SHA-citing: every claim is grounded in a verifiable artifact
- Temporal queries: "what did I believe about X on date Y?"

Both are right. Both should ship. They solve different problems: SHA-citing solves trust, temporal queries solve context retrieval.

---

## 2. Sprint 0 + Sprint 1 Task List (First Week, Sequenced)

### Sprint 0: Repair the Senses

**Day 1-2: Stop the bleeding**

| # | Task | Done looks like |
|---|------|-----------------|
| 0.1 | Close issue #1, archive the 72-comment thread | Issue closed. No open issue named "Fleet health report" exists. |
| 0.2 | Purge MEMORY.md: delete all "Awaiting further instructions" lines | `grep -c "Awaiting" MEMORY.md` returns 0. File under 1500 chars. |
| 0.3 | Split MEMORY.md → CONTEXT.md (live facts, rewritten) + git log (history) | `CONTEXT.md` exists, <800 chars, contains only: current project state, active blockers, Casey's last instruction. `MEMORY.md` gone or archived. |
| 0.4 | Patch `agent.py`: no-op cycles write nothing to memory | Cycles that find no work commit no changes to `CONTEXT.md`. Verified by running gate-replay on a simulated idle cycle. |

**Day 3: Fix the gate bugs**

| # | Task | Done looks like |
|---|------|-----------------|
| 0.5 | Fix `issues_unknown` string mismatch (Finding B) | `minigate/train.py` and `agent.py` use identical string. Feature fires on production's "fetch failed, count unknown". |
| 0.6 | Invert `issues_unknown` weight: unknown = alarm, not idle | Retrained gate: `issues_unknown` weight is positive (>0.5). Replaying a fetch-failure digest outputs p(busy) > 0.6. |
| 0.7 | Remove `outcome_idle` feature entirely (Finding C) | Gate has no feature that consumes its own prior verdict. Decision is purely exogenous. |

**Day 4: Establish honest baseline**

| # | Task | Done looks like |
|---|------|-----------------|
| 0.8 | Re-run `extract.py` on cleaned CONTEXT.md + fixed features | New `corpus.json` generated from repaired memory. |
| 0.9 | Measure idle rate with NO gate (ground truth) | Report: "Honest idle rate without gate: X%." If X < 85%, flag that gate's value prop is weaker than assumed. |
| 0.10 | Add `hours_since_substantive_work` feature to digest | Feature computes correctly from git log. Verified by manual inspection of 5 sample digests. |

**Day 5: CI guardrails**

| # | Task | Done looks like |
|---|------|-----------------|
| 0.11 | Implement `gate.py --selftest` (regression suite) | `--selftest` runs 10+ labeled test cases. Exit 0 = all pass. Exit 1 = any fail. |
| 0.12 | Wire selftest into CI: red CI withdraws skip authority | `.github/workflows` runs `--selftest` on every push. Gate skip is disabled if CI is red. |
| 0.13 | Version vault schema in a repo, validate in CI | `vault-schema/` directory exists with OpenAPI or JSON schema. CI validates vault responses against schema. |

---

### Sprint 1: Shadow + Trust Layer

**Day 6-7: Shadow gate + evidence binding**

| # | Task | Done looks like |
|---|------|-----------------|
| 1.1 | Wire minigate as shadow (log decision, never skip) | Every cycle logs: `gate_decision=RUN|SKIP, actual_outcome=...`. No cycle is skipped by gate. |
| 1.2 | Implement SHA-citing rule for memory writes | Every line in `CONTEXT.md` ends with `[sha:abc1234]` or `[unverified, 2026-10-17]`. No bare claims. |
| 1.3 | Implement stake/commitment tags on task intake | New tasks in `TASKS.md` get two fields: `commitment: hard|soft|watch|reject`, `blast_radius: low|med|high`. Logged from first signal. |
| 1.4 | Log disagreements: System-1 tag vs. eventual human/LLM override | When a human changes a stake tag, log the delta. This builds the calibration set. |

**Day 7+ (ongoing through week 2):**

| # | Task | Done looks like |
|---|------|-----------------|
| 1.5 | Injection screen on issue bodies before context inclusion | Issue text is wrapped with `[UNTRUSTED: ...]` annotation in digest. Agent sees the annotation. |
| 1.6 | Add tonic floor to gate (mcode's formula) | `p(work) = max(p_exogenous, min(0.35, 0.03 × hours_since_work))`. At 12 hours idle, floor = 0.36. |
| 1.7 | Implement pacemaker worker (Cloudflare ping → `last_tick` commit) | Worker deployed. Every 15 min, `last_tick` file updates. `hours_since_work` computed from this + git log. |

**Exit criterion for Sprint 1:**
- 7 days of shadow logs collected
- SHA-citation rate on memory writes: 100%
- Gate selftest green on every commit
- Honest idle rate (post-fix) documented
- Tonic floor active, no production incidents

---

## 3. Final Move for the Dream

The synthesis merged our dreams but lost one thing: **trust as a computable scalar, not a vibe.**

Concrete proposal:

```
TRUST.md (append-only ledger, one line per decision)
---
ts | decision_type | stake | outcome | autonomy_delta
2026-10-17T14:00 | close_issue | low | accepted | +0.01
2026-10-17T15:30 | open_pr | high | merged | +0.05
2026-10-17T16:00 | edit_readme | med | reverted | -0.08
```

**autonomy_score** = rolling sum of `autonomy_delta` over last 30 days, clamped to [0, 1].

**autonomy_ceiling** = `{low: 0.2, med: 0.5, high: 0.8}` — the score threshold required to take that stake level autonomously.

When `autonomy_score >= autonomy_ceiling[stake]`, the agent acts without asking. When it's below, the agent surfaces the decision and waits.

This makes "trust compounding" concrete: every good decision raises the ceiling; every bad one lowers it. The agent literally earns the right to act. The number is auditable. Casey can read `TRUST.md` and see exactly why the agent did or didn't take an action.

**Ship this in Sprint 2, after tonic floor and shadow data prove the agent is doing real work.** Trust accounting without real work is theater.
