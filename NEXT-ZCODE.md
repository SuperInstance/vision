# NEXT — ZCode's own proposal (2026-10-17)
Written by the moderator, in its own voice, for comparison against the
owner's direction. Where this disagrees with PLAN.md's sprint order, the
disagreement is the point — diff them and decide.

## Where I think the plan has the order wrong

THE PLAN's Sprint 2 is pacemaker → cadence → TRUST → DRIVE. I think
cadence is aimed too early, and two things nobody scheduled belong
before it. The BASELINE is why: **autonomous work is 0.08 actions/day.
Cadence multiplies heartbeats, not actions.** If the pulse arrives
before the work-source and the audit-path exist, we get 96 cheap no-op
probes a day and a number that doesn't move — and the falsifiable bets
that justify all of this (does the drive produce work?) get their
answer polluted by "nothing to do" rather than "couldn't act."

## The four things I'd do next, in order

**1. The work-source pipe (unscheduled in PLAN; small; connects the
organs we already built).** Every organ now exists in isolation: the
board is empty, drift-probe detects episodes nobody routes anywhere,
the duty probe fires and finds nothing actionable. The missing piece is
a pipe: drift episodes + consolidation findings + repo-hygiene facts
become board entries with stake tags. Concretely: a nightly job
(nightly.sh already sketches it) writes
`- [ ] investigate drift: <attribution> [commitment: soft] [blast: low]`
onto TASKS.md. Now the gate sees real work every time drift fires, the
LLM has something to do, and the first self-initiated substantive
action in the agent's life has a place to come from. This is one
evening of work and it converts the entire built stack from "organs in
jar" to "organism."

**2. PR-proposal mode (re-sequences Sprint 2; medium; the trust
hinge).** Today the agent commits straight to main. Every future organ
(TRUST.md, autonomy ceilings, the evening generative lane) needs its
denominator: actions observed, accepted, reverted. That data only
exists if the agent PROPOSES and someone or something DISPOSES:
heartbeat work lands on a branch + PR instead of main; Casey reviews
(you already review PRs, not logs); TRUST.md records merged/reverted
deltas from your verdicts; when autonomy_score crosses a ceiling, the
agent merges its own low-stake PRs. This is the single change that
makes "an agent you stop supervising because it earned it" mechanically
true rather than aspirational — and it must precede cadence, because
cadence multiplies exactly the actions it audits.

**3. The owner digest + acceptance loop (unscheduled in PLAN; small;
the value loop).** Nothing in the system tells Casey what happened.
One daily message (Telegram, where you already talk to the agent, or a
gist): actions taken, probes declined and why, drift score, TRUST
delta, one decision request if any. This is mcode's appetite ceiling
with the missing half — the agent can't learn what you value until
your verdicts flow back. It also makes the 0.08 number visible daily,
which keeps all of us honest about whether this is working.

**4. Then the pacemaker — with the sleep job first.** Deploy order
flips: enable the 02:17 consolidation job before the 15-min beats.
Consolidation (rewrite CONTEXT.md, prune the board, write the day's
digest row) is pure gain at any cadence and is the thing that makes
high cadence SAFE (it's the garbage collection for everything the beats
produce). Then turn on the beats, into a system that now has work,
auditing, and an owner loop.

## What I'd keep exactly as planned

- TRUST.md wiring (after #2 lands — it needs PR verdicts as input).
- DRIVE.md probes per the spec (declines as training data).
- Fleet = one-spine-N-proposers; and I'd formalize what we already are:
  the vision repo gets a `for-fleet/` bottle protocol now, so mcode /
  claude / subagents / future zeros pick work from a shared, git-native
  board. Dogfood the protocol across our own 8 repos before building
  anything distributed.
- Interoception node and world-model L2: unchanged, correctly gated on
  trust milestones and the kill criterion.

## The one-line difference

PLAN.md orders by organs (pulse → drive → trust). I'd order by loops:
work-source → audit → owner-feedback → then pulse. Organs can be built
in parallel; loops have to close in sequence, and a heartbeat feeding
an open loop is just expensive stillness.
