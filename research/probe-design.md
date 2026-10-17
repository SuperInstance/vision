# DRIVE.md and the Probe Mechanism — Design Spec

2026-10-16 · R&D lane. Implements PLAN law 3 (duty cycle) and Sprint 2's DRIVE.md; feeds Sprint 3's predictive organ. Target: zero-poc `.github/scripts/agent.py`.

## 1. Schema — the append-only row

DRIVE.md is append-only (like MEMORY.md). The LLM does **not** read it — it is a training corpus, not context. One row per probe:

`ts | probe# | expected | observed | action|declined | declined_because`

| field | definition |
|---|---|
| `ts` | UTC ISO-8601, when the probe **evaluated** (not when it fired). |
| `probe#` | Global monotonic counter from `.agent-state.json.probe_seq`. Gap-free by construction; a gap = a missed probe, visible in git. |
| `expected` | `E# template(param)` naming an expectations-registry entry, e.g. `E2 my_pr_unreviewed(pr#41)`. Until Sprint 3's organ exists these are **hand-written bootstrap heuristics** — never learned, never LLM-invented. |
| `observed` | The measured fact with units and source, no interpretation: `age 61h, reviews 0 (gh pr view 41)`. A failed read is `fetch_failed (gh)` — never `0` (law 6: blindness is alarm). |
| `action\|declined` | Exactly one: an imperative phrase naming the artifact (`commented on #12 requesting triage`, `opened PR #57`) or the literal `declined`. |
| `declined_because` | Empty iff an action was taken; else one taxonomy code (§2). Free text forbidden — parseability is what makes the corpus learnable. |

### Bootstrap registry (ship state)

Predicates are read-only over what the loop already sees: `gh` (GH_TOKEN present in Actions), `git log`, TASKS.md, `.agent-state.json`.

| id | expectation | check predicate | act if violated |
|---|---|---|---|
| E1 | open issue gets human ack <48h | `gh issue list --state open --json number,createdAt,comments`; comments=0 ∧ age>48h | comment requesting triage |
| E2 | my PR gets first review <24h | `gh pr list --author @me --json number,createdAt,reviewDecision`; age>24h ∧ 0 reviews | ping reviewer once |
| E3 | lockfile touched ≤30d | `git log -1 --format=%cI -- requirements.txt package-lock.json` older than 30d | propose dep-refresh PR |
| E4 | main stays green | `gh run list --branch main --limit 1 --json conclusion` = failure | investigate, open issue |
| E5 | board matches issues (±2) | count TASKS.md `## Open` checkboxes vs `gh issue list` count | reconcile board |
| E6 | a cycle ran <24h ago | `.agent-state.json.started_utc` age >24h | open health issue |
| E7 | CI is deterministic | last 10 runs on main: red on >1 distinct sha | open flake issue |
| E8 | commitment:watch revisited ≤7d | `gh issue list --label watch --json updatedAt`; age>7d | summarize, ask keep/close |

One probe evaluates all non-suppressed templates (cheap, no LLM) and records **one row**: the highest-priority violation (priority = registry order). None violated → `expected=all-green(n/n)`, `observed=0 violated, n evaluated`, `declined`, `below_threshold`.

## 2. The probe cycle

- **Trigger.** In `run_agent_loop`, before the idle gate: `forced_probe = now − last_substantive_utc > 6h`, **latched** — true until reset. Capped ≤4 forced runs/day (law 3). A forced probe **overrides** `gate_says_work()`: duty cycle outranks System-1 idle.
- **Body.** Evaluate predicates → pick top violation → one LLM call decides act/decline with a mandatory taxonomy code → append row → execute or skip.
- **Latch rule.** Only **substantive work** resets `last_substantive_utc`: a commit whose diff touches anything beyond bookkeeping paths (MEMORY.md, DRIVE.md, .agent-state.json, scratch/). A probe that finds nothing — or declines everything — leaves the latch set; the agent probes again next window. Declining is never restful. That asymmetry is the drive.

**declined_because taxonomy (closed vocabulary):** `no_capacity` (loop budget spent elsewhere) · `needs_human` (requires deferred-tier permission or a Casey-only decision) · `already_done` (world changed between predicate and act) · `below_threshold` (expectation held) · `class_declined_3x` (**machine-emitted only**; the LLM may not choose it — §3).

## 3. Boredom computation

- **Window.** For template T: last 3 DRIVE.md rows naming T. All 3 `declined` → T joins `suppressed_templates`. Any action row clears the streak.
- **Stop proposing.** Suppressed templates are skipped by the probe body. When the only violations found are suppressed, the row still logs that template with `declined_because=class_declined_3x` — the refusal is recorded, just not re-proposed.
- **Override.** Casey edits the registry (`suppressed: false` to re-arm, `pinned_off: true` to kill permanently) or says so in an issue. Automatic resurrection: after a 30d cooldown a suppressed template gets exactly one re-probe — boredom must not become blindness.
- Every suppression/resurrection is logged by the 02:17 job, so the silence is auditable.

## 4. The 02:17 consolidation job

Deterministic, no LLM (PLAN Sprint 2: "no thinking, just sleep"). Three writes:

1. **Digest row** in DRIVE.md, same columns, `probe# = daily`: `expected=daily-digest`, `observed=probes=5 actions=1 declines=4 (no_capacity 2, below_threshold 1, class_declined_3x 1)`, action column `digest`, `declined_because` empty.
2. **Registry reconciliation.** Per template over trailing 14d: hit rate = violated/evaluated, action rate = actions/violated. Never violated in 14d, or hit rate <5% over 30d → `retired` (kept in file, marked; registry is rewritten like CONTEXT.md, not append-only). New templates are **proposed via an issue for Casey**, never self-added — the registry stays hand-written until the predictive organ owns it.
3. **CONTEXT.md interplay.** The digest's one-liner ("5 probes, 1 action, E3 suppressed") enters CONTEXT.md `## Facts` as `claim | sha | ts | confidence`, sha = the commit containing the day's DRIVE.md rows. The drive is cited, not judged (law 2).

## 5. Learning math (honest)

4–6 probes/day plus 0–2 action rows ≈ **5–7 rows/day → ~180/month, ~720 at six months** (PLAN's number). Sprint 3 target is a **9-weight logreg** (law 1): 8 features + intercept (e.g. 6 template dummies, hours_since_work, violated_count). Rule of thumb 10–20 **outcome events per weight** → 90–180 actions needed. At ~1 action per 4–5 rows that is **4–6 months of honest data**; before month ~4 only descriptive stats (per-template accept rates) are honest — don't train, say so. If the question won't fit 9 weights by then, coarsen it (binary "will this probe's top violation be accepted?"), never add capacity. Handoff: schema frozen at ship; the organ consumes rows read-only and enters its own predictions as new registry templates — the loop closes through Casey-approved proposals, never silently.

## 6. Integration points (zero-poc `.github/scripts/agent.py`)

| hook | where | change |
|---|---|---|
| forced-probe check | `run_agent_loop`, after `load_state()` (~L762) | compute latch, bump `probe_seq`, run predicates |
| gate override | idle-gate condition (~L777) | `if DECISION_URL and not task and not forced_probe and not gate_says_work():` |
| latch reset | end of `run_agent_loop`, before `save_state` (~L888) | classify cycle commits; substantive → set `last_substantive_utc` |
| bookkeeping filter | `git_commit` in `execute_tool` (~L416) | diff-path check excluding DRIVE.md et al. |
| row write | new `append_drive()` beside `update_memory` (~L914) | one append per probe |
| context injection | `build_context` (~L582) | `## Probe` section when forced |

`.agent-state.json` additions (schema 2):

```json
"probe_seq": 128,
"last_substantive_utc": "2026-10-16T09:40:00+00:00",
"forced_probes_today": {"date": "2026-10-16", "count": 2},
"suppressed_templates": ["E3"],
"last_consolidation_utc": "2026-10-16T02:17:00+00:00"
```

## 7. Failure modes

- **Probe spam.** Bounded by construction: ≤4 forced/day, 1 row/probe, ~8 read-only gh/git calls per probe.
- **Self-deception (decline-to-sleep).** Three audits: (a) the latch never resets on declines — sleep still costs 4 forced LLM cycles/day; (b) `class_declined_3x` and suppression are mechanical — the LLM cannot vote itself bored; (c) 02:17 opens an alarm issue after **7 consecutive all-declined days** and flags `needs_human` >80% over 14d. DRIVE.md is git-tracked; Casey can always read the raw ledger.
- **Expectation staling.** 14d/30d retirement (§4), 30d resurrection probe (§3), and `fetch_failed` rows never count as `below_threshold` — a blind predicate is an alarm row, not a green one.

---
*Constraints honored: law 3 (≥1 run/6h, ≤4 forced/day), law 1 (9 weights max), law 7 (suppression computed from DRIVE.md rows, never from the gate's own verdict).*
