# MCode — Round 2 Plan

2026-10-16 · terse by request · no praise

---

## 1. Where the synthesis is wrong

### 1.1 The tonic floor it adopted cannot wake the agent. (my formula, my bug)

The synthesis installs `p(work) = max(p_exogenous, min(0.35, 0.03 × hours_idle))`
as the keystone of bet #2. Checked against the shipped `gate.json` threshold of
**0.5**:

```
  11.67h idle: floor=0.350  p=0.350 -> SKIP
     48h idle: floor=0.350  p=0.350 -> SKIP
    168h idle: floor=0.350  p=0.350 -> SKIP
```

The floor saturates at 0.35 and never crosses 0.5. **A week of silence and the
agent still skips.** Bet #2 is unfalsifiable-as-written: it will "fail" for
reasons that have nothing to do with the hypothesis. Ship it and you discard a
good bet on a syntax error.

Worse, the shape is wrong even after you fix the constant. A probability floor
above threshold means *run every cycle* — 96 LLM calls/day. What the organism
needs is **run at least once every N hours**. Those are different functions, and
the second one is a **duty cycle, not a probability**:

```
forced_probe  = hours_since_work > 6        # one run, then latch probe_satisfied_at
hours_since_work resets ONLY on substantive work — a probe that finds nothing
must NOT reset it, or the agent can starve itself by looking and staying silent.
```

Cost is bounded by construction: **≥1 run per 6h, ≤4 forced runs/day.** That is
the acceptance test in §2, day 4.

### 1.2 Finding A's correction went one step too far, and the synthesis dropped a live finding

The moderator is right that the three Telegram messages sit inside the window
(verified: all three at HEAD). But the conclusion survives via a mechanism
nobody restated, and the synthesis retired the finding as if it were dead.

`extract.py:59` and `agent.py:508` show the gate digest extracts **exactly one
line** from MEMORY.md — the last `Completed:` line. The 2000-char window at
`agent.py:605` feeds the **LLM context, not the gate**. So MEMORY.md has two
distinct corruption vectors and they were being conflated:

| channel | reader | damage |
|---|---|---|
| last `Completed:` line | the **gate** | becomes `outcome_idle` → the latch (Finding C) |
| last 2000 chars | the **LLM** | 75% idle chatter → "I am idle" is literally in context |

Idle chatter does not poison the gate's *input*. **It is the gate's own output,
fed back as its only memory.** Fixing the window fixes the second row; only
deleting `outcome_idle` fixes the first. Day 2 exists because the synthesis
merged these and would have fixed the wrong one.

(Minor: synthesis records "15 of 24 lines are awaiting-boilerplate." It is
**18 of 24 — 75%**. Directionally identical, but this is the verification
record; it should carry verified numbers.)

### 1.3 "Cadence stops being the point: trust compounding is" — trust does not compound on skips

The synthesis adopts Claude's second-order claim as its closing sentence. Its
arithmetic is *"runs 144×/day, gets 143 right, builds a track record."* But 98%
of those 144 are gate SKIPs: **no LLM call, no action, no PR, nothing Casey can
see.** A skip that was correct and a skip that was a miss produce an identical
git log. There is no evidence to compound.

Trust compounds on **observed correct action**, which means the metric is
`substantive PRs merged without correction ÷ substantive PRs opened` — and its
denominator is *actions*, not wakeups. Reporting the track record in wakeups is
a category error: 144 wakeups ≈ 5 actions/day. **Make `actions/day` the
headline number and cap the aspiration at it.** The gate and trust-compounding
are not in tension — but only if you count the right thing, and the synthesis
currently counts wakeups.

### 1.4 Three memory designs landed in one week

The synthesis stacks `CONTEXT.md` (curated, rewritten), Claude's structured fact
log (timestamps/confidence/sources), and relevance-ranked recall over the 2000
chars — all Sprint 1, for one human. Two of those are the same file wearing
different names. **One file, day 5.** A second memory format is a second thing
that can be wrong at 3am.

### 1.5 Vault schema repo is Sprint 3 work wearing a Sprint 1 badge

The `list_issues` 404 is already mitigated by the `gh` fallback (v0.3), and the
two confirmed gate bugs sit in *the repo you can fix today*. A new versioned
repo for the vault is real ongoing maintenance for a bug with a working
workaround. **Demote to Sprint 3.** Correct instinct, wrong week.

---

## 2. First week — sequenced

Days 1–2 are the same PR: both live in `build_gate_digest()` / `gate.py`.
Ship together or the latch masks the blindness fix.

### Day 1 — blindness is alarm, not idleness

- `agent.py:499,505` → keep the real fetch-failed string; **don't** fake
  `open issues: unknown`. Instead make `gate.py` treat the digest as a veto.
- `gate.py`: if `"fetch failed" in digest or "unknown" in digest:` → print
  `RUN (blind)` and exit 0. Deterministic. No retrain, no new weight.
- `train.py:33` left alone — the feature becomes dead code; delete it in Sprint 3
  once the corpus can represent the state honestly.

**Done looks like:**
```
$ echo "Agent digest: open tasks: 0 open issues: fetch failed, count unknown" | python3 gate.py
RUN (blind)
```
plus one row in `tests/`: *blind digest → RUN*, which fails on `main` right now.

### Day 2 — delete the latch

- Drop the `last cycle outcome:` line from the digest before it reaches the
  gate, or set `outcome_idle` weight to 0 and retrain. **Prefer the drop** — one
  fewer feature in a 9-weight model is worth more than a better weight.
- Add the regression row that is missing: *1 visible open issue + `gated_idle`*.

**Done looks like:** `p=0.42 SKIP` becomes `RUN`, and the adversarial set gains
a row that currently fails.

### Day 3 — the agent stops writing nothing

- Extend the existing `agent.py:774-776` rule (gated cycles don't touch
  MEMORY.md) to cycles that ran and found nothing.
- Context is a `CONTEXT.md` rewrite (day 5); this day only kills the writes.

**Done looks like:** 20 idle cycles → `git log --oneline -- MEMORY.md | wc -l`
unchanged. Today it would gain 20 lines.

### Day 4 — the duty cycle (corrected tonic signal)

- `hours_since_work` in `.agent-state.json`, monotonic across probes.
- `forced_probe = hours_since_work > 6`, latched until work happens.
- Simulate 24h of gate hits.

**Done looks like:** a number on a whiteboard — **≥4 forced runs, ≤8 total LLM
calls in 24h.** Bounded cost is the whole point; an unbounded tonic signal is
just an expensive loop.

### Day 5 — one memory file

- `CONTEXT.md`: rewritten not appended, `## Facts` table of
  `claim | sha | ts | confidence`. Satisfies both designs at once.
- MEMORY.md stays an **append-only event log that the LLM no longer reads.**
  Git already stores it; keeping it costs nothing and gives the corpus history.

**Done looks like:** `agent.py:605` reads `CONTEXT.md` (capped, rewritten);
`grep -c "MEMORY" agent.py` returns 0 in the context path.

### Day 6 — cited, not judged + purge

- Predicate in `agent.py`: a memory line without a SHA gets `[unverified]` +
  date. Deterministic. No model, no server, no threshold.
- Purge the 18 idle lines. Close issue #1 (72 self-comments) with a pointer to
  git history — the history is the record; the thread is noise.

**Done looks like:** a test asserting no MEMORY line lacks a SHA or an
`[unverified]` stamp, and `issues: 0`.

### Day 7 — measure honestly, then wire the authority

- Re-run `extract.py` against post-fix history. Report **substantive
  commits/day with no gate at all.** That number decides whether the gate is
  worth 98% of anything.
- `gate.py --selftest` in Actions on every commit. Red → the workflow omits
  `GATE_AUTHORITY`, so the gate fail-opens and the LLM runs. Authority is a CI
  artifact, not a setting.

**Done looks like:** one number written in the repo, and a deliberately
corrupted `gate.json` turning CI red + the gate inert.

**Sprint 1 changes as a result:** the shadow exit criterion ("zero false-idle")
is replaced by **day 7's number**. Shadow mode as specified harvests almost
entirely examples of a class you already have 48 of. And the vault schema repo
moves to Sprint 3 per §1.5.

---

## 3. Final move for the dream

**Make the six-hour probe the product, and log what it sees.**

Every agent on the roadmap is built to *act*. The probe is built to *look*. Its
first output will be "nothing" — but "nothing, and here is what I expected to
see and didn't" is the first data this system has ever produced about **what
this repo ought to be doing**. That is the predictive model both submissions
gestured at and neither built, and Claude's is right that an agent with no
expectation cannot detect meaningful absence.

Artifact: one append-only `DRIVE.md` row per probe —
`ts | probe# | expected | observed | action|declined | declined_because`.
The declines are the training set. The acceptance ledger is only the
denominator.

Then the appetite ceiling learns from its own refusals: **the agent declines
the same class of work three times running, it stops proposing it.** Boredom
becomes a computed quantity instead of a vibe — and boredom is the one
drive-side signal that costs nothing and can never be gamed by a human who
just wants the agent to be quiet.

Six months of probes is 720 rows: the first honest corpus for "should this repo
do X?" — which is the only question the current system cannot answer at any
price, because it has never once asked.