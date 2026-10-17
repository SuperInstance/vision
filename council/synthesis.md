# Council Synthesis — adjudicated by ZCode (moderator)
2026-10-16 · after independent verification of both submissions' claims

## Verification record (before anything is credited)

mcode's three forensic findings were independently re-run by the moderator:

| finding | claim | verdict |
|---|---|---|
| A | MEMORY.md dominated by idle chatter | **CONFIRMED** — 15 of 24 lines are awaiting-boilerplate (3010 chars). BUT the specific claim that Casey's 3 Telegram messages sit outside the 2000-char window is **FALSE at current HEAD** (all 3 are inside). The pathology is real; that detail is not. |
| B | `issues_unknown` feature never fires in production (string mismatch: prod says "fetch failed, count unknown", training said "open issues: unknown") | **CONFIRMED — production bug.** A total issue-fetch failure parses as `n_issues=0`. Blindness is encoded as idleness. |
| C | Idle-outcome latch: `outcome_idle` (−1.44) flips a digest with one visible open issue from p=0.75 RUN to p=0.42 SKIP | **CONFIRMED — design defect.** The gate feeds on its own last verdict; it has no exogenous change source. A single idle verdict blinds it to the human channel until a human writes a board task. |
| D | Cold start is the only production regime (ephemeral runners) | Sound; adopted as a written requirement. |

Claude's submission is strategic (no directly checkable code claims); its
reward below is for framing, not measurement.

## Category verdicts

**Unique insight — mcode, with claude's folded in.** mcode's "zero has
no attention, no clock, and no drive" is the load-bearing insight: all
three built System-1 instruments manage the pathology of having nothing
to do; the weasel claim was a *bored* agent, not a lying one. It is
falsifiable in 30 days (substantive-commit count) and buildable now
(tonic floor: `p(work) = max(p_exogenous, min(0.35, 0.03 × hours_since_work))`;
appetite ceiling from git's free labels: PR merged/closed/silent). Claude's
missing predictive model — "some silences are suspicious; an agent that
knows what should happen can detect meaningful absences" — is the same
insight at a higher level of organization: it is what the tonic layer
*becomes* once a clock exists. Both get seats; mcode's is the keystone.

**Abstraction — merged; mcode names the mechanism, claude names the relationship.**
- mcode: **the tonic layer** / "going endogenous" — every subsystem today
  is phasic (evoked); a drive signal is the missing organ. Plus the
  **distillation constraint ("second law"): a question may only be called
  System-1 if a nine-weight logistic regression can hold it** — neural on
  the 4050, linear in the runner, disagreement means the question is badly
  posed, not that capacity is needed.
- claude: **Calibrated Ambient Agency** — uncertainty itself as the
  routing key; the agent as peripheral nervous system that is always
  processing and rarely interrupting; **trust compounding** as the real
  second-order payoff of cadence ("the gate manufactures opportunities to
  demonstrate reliability").
The two compose into one sentence: **a circadian organism whose
activation is gated by calibrated uncertainty, earning graduated trust.**

**The refined dream — merged (they describe the same organism from outside and inside).**
Claude's day-in-the-life (one morning notification: handled/pending;
Telegram "fork it" → done by desk-time; an agent you stop supervising
because it earned it) is the user-facing story of mcode's architecture:
CONTEXT.md (rewritten, not appended) + evidence-bound memory + DRIVE.md
ledger + a pacemaker worker + circadian cron (work 08–23, consolidation
at 02:17 — "its most frequent behavior today is writing 'nothing
happened' into its own memory; a sleep cycle is the cheapest fix") + an
evening appetite-gated generative lane on the laptop GPU that proposes
PRs. Convergence marker: both independently repurposed the Arduino from
reflex-actuation to **sensing the room** (claude: context for urgency;
mcode: interoception — the one information class the repo structurally
cannot observe). Both killed the mesh: claude on trust grounds (physical
failure before a trust model exists), mcode on operator-elimination
grounds (git is already the consensus — one spine, N proposers, PRs as
proposals).

**Sprint surgery — mcode's scalpel, claude's addition, both kills honored.**
- Adopted from mcode: **Sprint 0 (repair the senses)** — clean memory,
  stop no-op writes, fix the two confirmed gate bugs, re-measure the
  honest idle rate BEFORE trusting the gate's 98% number ("measured on a
  poisoned digest"); **evidence predicates over probabilistic judging**
  (every memory write cites a SHA or is stamped `[unverified]` — "cited,
  not judged"; the judge's founding measurement is n=1); **gate authority
  via `gate.py --selftest` in CI** (red CI withdraws authority — you
  cannot measure false-idle on a stream with no busy in it); **vault
  schema as a versioned, CI-tested repo** ("the last unversioned organ");
  close issue #1 and purge the 18 idle lines.
- Adopted from claude: **stake/commitment tags logged from day one**
  (hard/soft/watch/reject × blast radius — cheap now, and the calibration
  dataset can't be retrofitted); **queryable temporal memory over vector
  search** (structured fact log with timestamps/confidence/sources —
  interpretable and debuggable); **graduated autonomy as a measurable
  stat** fed by the acceptance ledger.
- Honored kills: the mesh as written (replaced by one-spine-N-proposers);
  Arduino reflex actuation (deferred past trust milestones); world-model
  companions demoted to ONE timeboxed nightly experiment with a written
  kill criterion (10 PRs, 0 merges → closed).

## The bet we are now taking (mcode's falsifiable claims, adopted)

1. Fixing the memory window reduces measured idleness more than shipping
   the gate does. (30-day substantive-commit count decides.)
2. The tonic floor produces self-initiated work. (~0/day for a month with
   the floor active falsifies.)
3. Citing a SHA beats judging a claim. (Unsupported-claim rate decides.)

If all three fail: the reflex-first roadmap was right, ship the mesh.

## Sentence of record

The trust layer was treating a bored agent as a lying agent, and the mesh
was treating a twitchy agent as a thinking one. Give it attention, a
clock, and something it wants — then the reflexes already built become
enough, and cadence stops being the point: **trust compounding is.**
