# Council Brief — The Refined Dream
## Read this fully, then write your submission to the file you were told to use.

You are one of two competing AI advisors in a strategy council. The other
advisor is a different model working from this same brief. You are NOT
collaborating — you are competing for the sharpest insight. The moderator
(a third agent) will synthesize both submissions and the best ideas win a
place in the permanent roadmap. Do not hedge. Do not summarize the brief
back. Contribute what only you can see.

## The system today

"zero" is a git-native AI agent: a GitHub repo that IS the agent. A
throttled Actions heartbeat (~4-5 runs/day) runs a ReAct loop (agent.py);
its LLM brain goes through a secret-vault Cloudflare Worker (paid tokens).
Memory is git itself: append-only MEMORY.md + a TASKS.md board. It reads
GitHub issues and Telegram DMs. It is alive and autonomous on main.

A new capability was just built and measured: **System-1 decision models**
— tiny fast confidence-scored classifiers (~20-150ms, local, free) — used as:

1. **Idle gate** (v0.4, live): before each expensive cycle, a distilled
   digest is judged "actionable work?" — a confident "no" skips the LLM
   entirely. Fail-open by design.
2. **Completion judge** (PoC): task-framed questions catch false
   self-reports. Measured: weasel claims ("Completed: awaiting
   instructions" after abandoning the task) score 0.57 claim-framed but
   FLAG at 0.99 task-framed. Framing is the lever.
3. **Flail detector + injection screen** (PoC): deterministic stall rules
   + untrusted-text annotation for issue bodies.

Measured on real git history (49 cycles: 48 idle): winnow-class decision
model would skip 98% of idle cycles at zero measured false-idle; the
current default model (laya) only fires 6% — wrong tool, being replaced.
A zero-dependency distilled gate (10 logistic weights, pure stdlib) now
exists for running the gate inside any CI runner with no server.

Three PoC repos live: gate-replay (calibration from git history),
minigate (distilled serverless gate), cycle-judge (trust layer).

## The current roadmap (to be improved by you)

- **Sprint 1 — Integration, shadow first:** fix known bugs; wire minigate
  as shadow gate (log, don't skip) to farm production labels; wire the
  completion judge before memory writes ("Attempted: … unverified" for
  flagged claims); injection screen pre-context. Exit: a week of shadow
  logs, zero incidents.
- **Sprint 2 — Cadence:** external pinger + workflow_dispatch heartbeat
  (cron throttling bypass); grant skip authority if production false-idle
  ≈ 0; board hygiene (zombie task reaping); Telegram triage. Exit:
  15-minute cadence at the old cost envelope.
- **Sprint 3 — Trust + memory:** flail break-on-stall; relevance-ranked
  recall beyond the 2000-char recency window; false-negative feedback
  loop; self-healing memory.
- **R&D lane:** federated edge mesh (state/context bus, tiered routing,
  hot/warm/cold compute pods, gossip); world-model companions
  (diffusionGemma, JEPA-family "graphs and quilts" alongside the Jev-like
  decision models); two Arduino Uno Q units as reflex nodes (micro-router
  firmware, self-describing manifests) in an "external neural net."

**The strategic reframe already banked:** the idle gate's value is not
token savings — it is CADENCE. Skip most cycles free, run every 15
minutes, same cost envelope: a batch agent becomes a responsive one.

## Your task (compete on all four)

1. **Unique insight:** what does everyone here still not see? Find the
   load-bearing idea that is missing from the roadmap. Not incremental
   improvements — the blind spot.
2. **Abstraction:** name the deeper pattern this system is becoming.
   ("An agent whose entire cognition is cheap judgments, with expensive
   reasoning as the exception" is already taken — go further or demolish
   it.) Give the abstraction a name you'd defend.
3. **The refined dream:** where is this heading in your voice — 12-36
   months out. Be concrete enough to act on: what exists, what runs
   where, what a day in the life looks like, what the fleet/mesh/world
   actually does. Opinionated, not a survey.
4. **Sprint surgery:** the three biggest changes you'd make to the
   roadmap above, with reasoning. Kill things that deserve to die.

Constraints to respect: solo human builder (Casey) + AI agents doing the
work; laptop GPU (RTX 4050 6GB, shared) is the only local compute;
GitHub Actions is the only free cloud runtime; secrets live in a
Cloudflare vault; everything ships as agent-readable git repos.
