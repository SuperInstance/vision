# MAP — the SuperInstance agent ecosystem (2026-10-17)

One live agent, one bench, seven satellites. Private unless noted.

| repo | role | status |
|---|---|---|
| **zero-poc** (PUBLIC) | The agent itself. ReAct loop (agent.py), vault-gated LLM brain, git-native memory, duty-cycle drive, idle gate, shadow observer. Sprint 0 + Sprint 1 shipped (58 tests). | live on Actions |
| **minigate** | The serverless idle gate: 9-weight logreg, zero deps; selftest grants/withdraws skip authority in CI (14 rows, currently GRANTED). Vendored into zero-poc for shadow use. | stable |
| **gate-replay** | Calibration from git history: corpus extraction (49 real cycles), replay harness, adversarial probe set. Winnow-class models: 98% idle-skip @ 0 false-idle. | stable |
| **cycle-judge** | Trust layer PoC: task-framed completion judge (weasel claims 0.57→0.99 by framing), flail detector (deterministic rules load-bearing), injection screen. EVAL.md documents laya-only limits. | stable |
| **vision** | This repo: the council record, PLAN.md (roadmap of record), BASELINE.md (the yardstick), research/ (probe-design, world-models, interoception scouts), MAP.md. | living doc |
| **trust-ledger** | Trust as a computable scalar: autonomy deltas, 30-day rolling score, ceilings per blast radius, anti-inflation caps, explain(). 20 tests. | PoC, ready to wire |
| **pacemaker** | The agent's clock: circadian dispatch worker (15-min beats 08–23 weekdays, 02:17 sleep job), KV-deduped, /tick reserved for the interoception node. 47 tests. | deploy-ready (user deploys) |
| **drift-probe** | The world-model lane's L1: classical drift detection over 15 git metrics (EWMA+z / Holt). Verified on real history; blind to steady slides by design. 15 tests. | PoC, nightly-ready |
| hermit-bench (local, not a repo) | The dry-run bench: hermetic stub gh, dead-port LLM canaries, scenario boards. Every agent.py change validates here before push. | local discipline |

**The chain, one line each:** the pacemaker gives a pulse → the gate
skips idle beats for free → the duty cycle forces a look every 6h →
consolidation keeps memory honest (CONTEXT.md, SHA-cited) → drift-probe
watches the trajectory → TRUST.md converts correct actions into
autonomy → probes (DRIVE.md) turn declines into training data → the
predictive organ eventually notices the absences.

Read the full reasoning: `council/` (two competing models + verified
synthesis), `PLAN.md` (laws + sprints), `BASELINE.md` (the honest
numbers), `NEXT-ZCODE.md` (the moderator's own sequencing proposal,
offered for comparison against the owner's direction).
