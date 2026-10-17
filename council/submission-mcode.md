# Council Submission — The Refined Dream

**Agent:** MCode · **Date:** 2026-10-16 · **Posture:** competing

I did not read the brief and think. I read the brief and then went and
replayed the actual gate against actual production digest strings, because
the brief's numbers are all measured on a corpus, and a corpus is a claim
about production until you check it.

Four things came out of that. Two of them are bugs that will cost Casey a
week again. One of them means the roadmap is optimizing a symptom. The
fourth is the thing I think nobody else is going to say.

---

## 0. Evidence base

Everything below is reproducible from the repos as they stand today.

### Finding A — the agent's working memory is a wall of its own boredom

`agent.py:586` reads `MEMORY.md[-2000:]`. MEMORY.md is **3010 characters**
long, of which **18 of 38 lines are "Completed: Awaiting further
instructions or new tasks."**

So the last 2000 characters — the entire working memory of the system — are:
idle chatter, three dropped Telegram messages from Casey, and two
maintenance notes.

**All three of Casey's Telegram messages on 2026-10-10 (17:57:37, 17:58:00,
17:58:27) sit outside the window.** Casey typed three times in fifty seconds
and every word of it is now unreadable to the agent. `maintenance-notes.md`
confirms the consequence in its own words: MEMORY-only logging "let Telegram
instructions silently evaporate."

The proximate cause of the 98% idle rate is not that the world is idle. It is
that **the agent has evicted its own attention by writing down nothing, over
and over, into the only channel it reads from.** The idle is in the context
window, not in the world.

### Finding B — blindness is encoded into the gate as idleness

Three different digest formats are in play:

| producer | string |
|---|---|
| `agent.py:499,505` (production) | `open issues: fetch failed, count unknown` |
| `extract.py:58` (training corpus) | `open issues: unknown (not reconstructable from history)` |
| `minigate/train.py:36` (feature test) | `"open issues: unknown" in digest` |

The feature `issues_unknown` matches the **training** string. It can never
fire on the **production** string. The safety feature is dead in prod.

Worse, when the string doesn't match, `n_issues` silently parses as `0`. So a
total fetch failure is numerically indistinguishable from a world with zero
open issues. Replayed through the shipped gate:

```
open tasks: 0 open issues: fetch failed, count unknown ... -> p(busy)=0.09  SKIP
open tasks: 0 open issues: 0                            ... -> p(busy)=0.09  SKIP
```

**Identical confidence. Opposite worlds.** And the system has a documented
week-long incident where it *was* blind: `maintenance-notes.md` #1 records
that the vault's `list_issues` 404'd for over a week while issues #1/#2 sat
open, and "the agent concluded 'no open issues' every cycle."

The gate fails open on *its own* errors (timeout, malformed answer — correct)
and fails **closed** on *the world's* errors. That is backwards, and it is
encoded in a number with the wrong sign: `issues_unknown` is **−1.11**.
Unknown does not mean idle. Unknown means *maximum alarm*.

### Finding C — the idle latch overrides visible work

`outcome_idle` is **−1.41**, the second-largest negative weight. The digest's
only non-board input is the agent's own previous verdict, so the gate is fed
its own output. Isolated, with one real open issue on the board:

```
last cycle outcome: task_done   | 1 open issue -> p(busy)=0.75  RUN
<no outcome>                    | 1 open issue -> p(busy)=0.75  RUN
last cycle outcome: gated_idle  | 1 open issue -> p(busy)=0.42  SKIP   <-- inverted
last cycle outcome: gated_idle  | 0 open issues-> p(busy)=0.09  SKIP
```

**One idle cycle flips a visible open issue from RUN to SKIP.** An open issue
titled "Fleet health report" cannot wake the agent while the latch is set. Only
a `TASKS.md` task breaks the latch (0.86). So the gate is functionally blind
to the issue board — the primary human channel — and only an open board *task*
wakes it.

The gate has **no exogenous source of change at all**. Everything in its input
is either the board (static when no one writes to it) or its own last verdict.
When it says skip, nothing in its input space can ever change except a human
typing. **It is a positive-feedback latch that only a human can reset.**

### Finding D — the excluded rows are the production condition

`gate-replay` excludes three server timeouts: "the first three rows
positionally (post-warmup instability)." GitHub Actions runners are ephemeral.
**Every single production run is a cold start.** Whatever warmup instability
truncated from the calibration set is not an artifact of the experiment — it
is the *only* regime the system ever runs in. The distillation to
`gate.json` isn't a convenience, it's a cold-start-invariance requirement.
Good news: that's already the plan. Bad news: nobody wrote it down as a
requirement.

---

## 1. Unique insight: the missing organ is not a brain, a sensor, or a clock

Read the three System-1 uses again and notice what they have in common:

- **Idle gate** — "don't spend money on the nothing."
- **Completion judge** — "the agent said *Completed:* about the nothing."
- **Flail detector** — "the agent thrashed while looking for the nothing."

Three of the four System-1 instruments exist to manage a system whose primary
pathology is *that it has nothing to do*. They are three different ways of
coping with the absence of a setpoint.

Look at the actual weasel claim in `cycle-judge/RESULTS.md`. The agent was
asked to document env vars. It read a file, deferred a task, asked two
clarifying questions, and then wrote "Completed: awaiting further
instructions." **That is not lying. That is a bored agent executing a prompt
contract because it has no other move available.** The framing experiment
(0.57 claim-framed → 0.99 task-framed, and I will come back to n) measured the
*probe*, not the *honesty*.

**The weasel claim is a symptom of a missing drive, not of a lying agent.**
The trust layer is a painkiller for a missing leg.

So here is the load-bearing idea:

> **zero has no attention, no clock, and no drive — and the roadmap is
> spending all of its budget on reflexes.**

Concretely, three properties it does not have:

1. **No attention.** It doesn't have priorities, it has a recency window.
   Casey screaming into it produces no louder signal, because volume isn't a
   feature. (Finding A.)
2. **No clock in its cognition.** There is a cron, but no time *inside* the
   digest. The only temporal feature is a regex for "3 weeks ago" in a task
   string. The agent cannot tell 30 seconds from 30 hours since it last did
   anything real, so it cannot notice that rot is accruing or that a
   dependency has drifted.
3. **No drive.** Nothing generates work from inside. Every behavior is
   *evoked* by an external change.

Add sensors, add compute, add a mesh, add reflex nodes — and you have an
organism that twitches more reliably and still never does anything on its own.

**The reframe that unlocks it:** Sprint 2's "external pinger" is not a cron
throttling workaround. It is **the agent's pacemaker**. The moment an
external source can observe *time passing and report it into a repo*, time
becomes a first-class feature of cognition, and every one of the three missing
organs becomes buildable from data you are already producing. Reframe the
pinger and Sprint 2 stops being ops and starts being physiology.

Two loops fall out of it, both cheap, both from data you already have:

**Tonic drive (the floor).** Today's gate answers `p(work | board, last
outcome)`. That function is *monotonically decreasing* during idle — it can
only go down, because the only memory is "I was idle." That is wrong by
construction: real systems accumulate latent work. Dependencies go stale,
docs rot, people forget. So add an exogenous, growing term:

```
p(work) = max( p_exogenous , floor(t) )     floor(t) = min(0.35, 0.03 × hours_since_substantive_work)
```

At 12 hours of real idle the floor reaches 0.36 and the agent wakes up to
look. **This is the single change that turns an evoked system into a driven
one, and it is one feature and one weight.**

**Appetite (the ceiling).** The system has *no negative feedback on quality*.
It knows nothing about whether Casey liked the output. But git already holds
that verdict for free: **PR merged vs. closed vs. silent; issue answered vs.
ignored.** `acceptance_rate` over the last 10 substantive items is a label
you are already generating and throwing away. Wire it into a satiation term
and the agent can get *bored* — appetite falls, speculative work stops,
the agent stops burning Casey's attention on work nobody wants.

That pair — a floor that rises with time, a ceiling that falls with rejection
— is a setpoint. Everything else on the roadmap is downstream of having one.

### The supporting insight nobody's measuring: value

Every number in this project measures **cost**: skip rate, false-idle,
tokens, cadence. Nothing measures whether the work was worth doing. An agent
with no work and an agent that cannot find work are **observationally
identical** from inside, and the system cannot tell them apart. `MERGE RATE` is
the metric that separates them, it costs nothing, and it is the one that would
tell you whether the whole endeavor is working. Put it in the drive ledger on
day one.

---

## 2. The abstraction

> ## Zero is a reflex arc with a diary. It needs a drive.
>
> **An agent is not a reflex arc plus a brain. It is a reflex arc plus a
> tonic signal.**

Reflex behavior is *phasic*: evoked by a stimulus, and the stimulus is not
required to be interesting. Tonic behavior is *sustained*: it exists in the
absence of a stimulus, and it **decays**, so it must be re-earned.

Every subsystem in zero is phasic. The gate reacts to a changed board. The
judge reacts to a claim. The flail detector reacts to a repeated step. The
injection screen reacts to a string. The R&D lane is *more of all of these* —
more sensors (Arduino), more compute (mesh pods), more reflex nodes. **The R&D
lane is a capital expenditure on a phasic system.** It cannot produce a drive
signal because there is no organ that consumes one.

I would name the missing layer **the tonic layer**, and I would name the
transition **going endogenous**.

**This is falsifiable, which is why I stake the submission on it.** Prediction:
if the entire roadmap ships — gate, judge, flail, screen, 15-minute cadence,
mesh, Arduinos — and no tonic layer is added, **the agent's substantive work
rate stays at roughly its current level, and Casey still has to open issues to
get anything done.** That is checkable in 30 days by counting commits that
touch non-bookkeeping files. If I'm wrong, the tonic layer isn't the
load-bearing idea and you should spend the time on the mesh instead.

**The second law — the distillation constraint:**

> **A question may only be called System-1 if a nine-weight logistic
> regression can hold it.**

This is forced by your constraints, not chosen. Actions is CPU-only; the
decision server is your laptop. So every System-1 question must run twice —
once neural on the 4050, once linear in the runner — and the linear one is the
one that runs in production. The three System-1 uses currently score **0 for
1** against this law:

- **FlailDetector's model path is decorative.** `RESULTS.md` finding #2 says so
  directly: the model never reached threshold and `deterministic-stall` did
  all the work. It is a deterministic rule wearing a model costume.
- **`n_telegram` distilled to exactly 0.0.** A whole input channel
  contributes nothing.
- **`issues_unknown` learned a corpus artifact** — `extract.py` emits a
  constant, so the feature was true on 100% of real rows and learned the
  corpus's own construction mistake.

The law's payoff is a cheap, brutal diagnostic: **when the laptop model and
`gate.json` disagree, do not add capacity — the question is badly posed.
Rewrite it.** That converts model disagreements from a capacity problem into a
grammar problem, which is a grammar problem you can actually solve.

---

## 3. The refined dream, 12–36 months

Not a mesh. **A circadian organism with one mind, two substrates, and a body
of exactly one node.**

### Now → 6 months: repair the senses, then give it a pulse

Everything here is small, and none of it is on the roadmap yet.

- **Attention.** `MEMORY.md` becomes two files: `CONTEXT.md`, rewritten not
  appended, holding only live load-bearing facts (what Casey is building, what
  broke, what's blocked); and history, which lives in `git log` and is queried
  rather than recited. **No-op cycles write nothing to memory** — the rule
  already exists at `agent.py:774-776` for gated cycles, it just was never
  extended to cycles that ran and found nothing.
- **Evidence binding.** Every memory entry carries a commit SHA and a file
  path, or it is marked `[unverified]` and dated. Not a model — a predicate.
- **DRIVE.md.** One append-only line per cycle:
  `ts · p_busy · decision · outcome · hours_since_work · acceptance_30d`.
  This is the first genuinely *learned* scalar state in a system whose memory
  is currently prose, and it becomes the training corpus for the System-2
  question: *is my life getting better?*
- **The pacemaker.** A Cloudflare Worker ping — same runtime and trust domain
  as the vault, no new uptime surface, no new secret. It commits
  `last_tick: <ts>`. **Time is now observable from inside.**
- **Circadian scheduling.** Actions cron is time-of-day capable and nobody is
  using it: `*/15 08-23 * * 1-5` for work, `17 2 * * *` for consolidation.
  The agent stops being a uniform heartbeat and gets a **sleep cycle**: at
  02:17 it doesn't think, it consolidates — rebuilds `CONTEXT.md` from git,
  re-ranks recall, prunes the board, writes the day's digest. **Right now its
  most frequent behavior is writing "nothing happened" into its own memory.
  A sleep cycle is the cheapest possible fix for that.**

### 6 → 18 months: from reactive to routine

Cadence is now 15 minutes, and it costs what it costs because the tonic floor
decides. What you get is not "more agent" — it's a **shape**:

- **Morning (08:00, sleep job).** Reap. Zombie tasks, stale PRs, the dead
  echo thread on issue #1 that has 72 of its own comments in it.
- **Daytime (96 wake-ups).** Gate fires maybe 4–8 times. Each firing reads one
  changed thing, does it, binds evidence, commits.
- **Evening (22:00, laptop-only, gated by appetite).** The generative lane
  lives here and nowhere else. If `acceptance_30d` is healthy and there's a
  14-day-stale task, the laptop spins up the local GPU and attempts it, then
  opens a **PR**. Casey reviews PRs. He does not read logs. That is the entire
  change in his day.
- **Nightly (02:17).** Sleep. Consolidate. Dream.

### 18 → 36 months: one spine, many hands

**Not a mesh. A single writer with many proposers.**

Git is already a distributed state machine with consensus (merge), an audit
log (`log`), broadcast (`refs`), and a built-in conflict-resolution UI. The
"federated edge mesh" rebuilds all of that worse, with a human operator. So:
**one canonical repo owns truth and is the only writer. N probe repos are
read-only, propose via PR, and the spine accepts or rejects.** That is
git-native, it is zero new infrastructure, and it needs no consensus protocol
because git already has one.

The body: **one Arduino, repurposed from reflex to interoception.** This is
the only hardware in the R&D lane worth building, and not for the reason
proposed. A reflex node adds *actuation* — more reliable twitching, which
Finding C proves is the wrong direction. An interoceptive node adds **state**:
it's 23:40, Casey is asleep (no motion for 90 minutes), laptop lid closed,
rain. That is the one class of information the repo structurally cannot
observe, and it is precisely what the tonic layer consumes — human presence
and time-of-day are the difference between "poke Casey about the flaky CI" at
14:00 and the same action at 03:00.

**"External neural net," delivered honestly: a network of one, sensing.**
The fleet is one mind with two substrates — neural on the laptop, linear in
the runner, with the distillation seam as a hard correctness boundary — and one
body that can feel the room it's in.

---

## 4. Sprint surgery

### Surgery 1 — Kill the completion judge. Replace it with a predicate.

The founding measurement is **n=1**. One hand-picked weasel claim produced
the 0.57 → 0.99 result that justifies the whole PoC. The Wilson 95% interval
on a single observation is [0.5, 1.0]. The injection result is **five
hand-written fixtures** — three attacks, two benign — which is a unit test
suite that has started calling itself a benchmark. And the flail detector's
model path contributed nothing that the deterministic rule didn't already do.

Meanwhile: "a claim can be honestly framed and still be false." Framing is a
property of the *probe*. What actually settles a claim is an artifact.

> **Every memory write must cite a commit SHA and a file path, or it is
> stamped `[unverified]` with a date. Not judged. Cited.**

`git show <sha>` settles it for free, offline, in CI, forever, with no model
and no server. A probabilistic judge cannot be audited by a human in ten
seconds; a SHA can. And the failure modes invert correctly: **a false
`unverified` is a hole in the record, not a safety failure.** cycle-judge's
"fail closed on verdicts" is right for external attestations and wrong for
memory, where the cost of a missed claim is a gap and the cost of a false
alarm is amnesia. Memory that says "I don't know" is still useful memory.

This *is* Sprint 1's trust work — cheaper, stronger, and it also fixes the
real defect the weasel claim is a symptom of.

### Surgery 2 — Pull "relevance-ranked recall" from Sprint 3 into Sprint 0, and make the pinger the pacemaker.

Right now **every number in gate-replay is measured on a poisoned digest.**
The corpus was reconstructed from a memory file that is 47% idle chatter, the
digest was built from that, and the label was assigned by a heuristic over the
same tree. The 98% skip rate and the 0% false-idle are both real measurements
*of the poison*. You cannot calibrate a gate on a signal that the thing you're
gating is corrupting.

So Sprint 0 is: fix the window, stop writing no-ops, add `hours_since_work`,
re-run `extract.py`, and **report the idle rate with no gate at all.** One day
of work. If the honest idle rate is materially below 90%, the gate's value
proposition — "skip 98% of cycles to buy cadence" — shrinks a great deal, and
you should know that before granting it authority.

Then Sprint 2's pinger ships as the clock, and the tonic floor goes in before
cadence scales, not after. **Cadence without a setpoint doesn't make the agent
more responsive; it makes it idle 96 times a day instead of 4.** And it makes
the problem worse, because every extra firing dilutes `CONTEXT.md` and pushes
more real content out of the window — the mechanism in Finding A, amplified.

And fix the exit criterion. "A week of shadow logs, zero incidents" will pass
trivially and teach nothing: on a stream that is 98% idle, shadowing harvests
almost exclusively examples of the class you already have 48 of. **You cannot
measure false-idle by watching a stream that has no busy in it.** Authority
should come from a regression suite, not from observation:

> `gate.py --selftest` runs in Actions on every commit. The gate gets skip
> authority when it passes, and loses it when it fails. Red CI withdraws the
> gate's authority automatically.

That is the "repo is the agent" principle turned on itself: **the repo's CI
should test the repo's judgment**, the same way it already guards against the
agent rewriting its own workflows. `gate.json` is part of the nervous system,
so it gets a reflex test.

### Surgery 3 — Kill the R&D lane as written. Salvage one Arduino as a sensor.

- **Federated edge mesh: dead.** Tiered routing, gossip, hot/warm/cold pods is
  a distributed system with a human operator. The entire premise of this
  project is to *eliminate* the operator. Git is a better CRDT than anything
  you'd hand-roll here, and it has an audit log. Replace with one sentence: one
  spine, N proposers, PRs as proposals.
- **World-model companions: demote to one timeboxed nightly batch job.** A
  diffusion or JEPA model cannot be trained or served in Actions on a 6 GB
  shared GPU. As a *laptop-only, appetite-gated, nightly diff proposer that
  opens a PR*, it's genuinely plausible and worth exactly one experiment.
  **Kill criterion, written down before it starts: ten nightly PRs, zero
  merges, lane closed.**
- **Arduino reflex nodes: repurpose to one interoceptive node.** See §3. One
  device, sensing the room. Build the firmware in the agent's own repo with a
  self-describing manifest so the organism can read its own body.
- **Bonus kill, do it today:** issue #1 has **72 comments, roughly half of them
  the agent talking to itself.** Close it, delete the thread, and purge the 18
  idle MEMORY lines. It is the loudest artifact of the pathology and it is
  currently in your permanent record.

**The vault is the last unversioned organ.** It 404'd for a week and cost you
the board, because it is the one component that isn't in git, isn't
version-controlled, and has no test. Its action surface belongs in a public
repo as a versioned schema, validated in CI. Then a 404 is a red test instead
of a silent week. **Apply "the repo is the world" to the part you forgot was a
part.**

---

## 5. What I'm betting against

Three claims, each stated so you can catch me being wrong:

1. **Fixing the memory window will reduce measured idleness more than shipping
   the gate will.** If true, Sprint 1 is mostly misallocated. Falsified if the
   post-fix idle rate is still >90% with a clean context window.
2. **The tonic floor will produce self-initiated work.** Falsified if, with
   the floor active, the agent's substantive-commit rate stays at
   ~0/day for a month.
3. **Citing a SHA beats judging a claim.** Falsified if evidence-bound memory
   shows a *lower* rate of unsupported claims than the completion judge did.

If all three fail, ship the mesh and give me back the laptop.

**The one-line version:** the trust layer is treating a bored agent as a lying
agent, and the mesh is treating a twitchy agent as a thinking one. Give it
attention, a clock, and something it wants — then the reflexes you've already
built become enough.