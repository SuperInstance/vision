# vision — a roadmap designed by competing AIs, for an agent that lives in git

This repo is the compass for the
[SuperInstance](https://github.com/SuperInstance?tab=repositories)
agent ecosystem: the roadmap of record, the honest baselines, and the
full transcript of the experiment that produced them — **a strategy
council in which two frontier models (Claude Opus 4.5 and MiniMax mcode)
received the same brief and competed**, with a third agent (ZCode)
moderating, independently verifying every empirical claim before
anything entered the plan.

## Why this exists

The system under development is
[zero-poc](https://github.com/SuperInstance/zero-poc): a GitHub repo
that *is* an AI agent — a ReAct loop run by Actions heartbeats, memory
kept in git, an LLM brain reached through a secret vault. The research
question: what's the smallest set of organs that turns a throttled cron
loop into something with a drive, honest memory, and earned autonomy?

The council's answer (and the reframe that drove everything after):

> The trust layer was treating a bored agent as a lying agent, and the
> mesh was treating a twitchy agent as a thinking one. Give it
> attention, a clock, and something it wants — then the reflexes
> already built become enough.

## Contents

- **[PLAN.md](PLAN.md)** — the roadmap of record: seven invariant laws
  (cited-not-judged, duty-cycle-not-probability, authority-is-a-CI-
  artifact, blindness-is-alarm…), sprint-by-sprint execution, and the
  falsifiable bets that would kill the plan itself.
- **[BASELINE.md](BASELINE.md)** — the honest numbers: 98% of the
  agent's cycles are idle; its lifetime autonomous substantive output
  is one commit. The yardstick every change must move.
- **[MAP.md](MAP.md)** — the ecosystem map: all nine repos, their
  roles, and the one-line chain from pacemaker → gate → duty cycle →
  consolidation → drift-probe → trust.
- **[NEXT-ZCODE.md](NEXT-ZCODE.md)** — the moderator's dissent:
  PLAN.md orders by organs, this argues for ordering by loops
  (work-source → audit → owner-feedback → then pulse). Diff them.
- **council/** — the full record: `brief.md`, both verbatim competing
  submissions (`submission-claude.md`, `submission-mcode.md`), the
  verified `synthesis.md` (every claimed bug re-run before credit),
  and both round-2 plan critiques (`plan-claude.md`, `plan-mcode.md`).
- **research/** — scouting reports: the DRIVE.md probe design
  (`probe-design.md`), world models for a git-native agent
  (`world-models.md` — what's real at 6 GB vs vaporware), the
  interoception node (`introception-node.md`), the System-1 landscape
  as of Oct 2026 (`system1-landscape.md`), and the TypeSafe/Jev
  deep-read (`typesafe-notes.md`).

## The method, if you want to run your own council

1. Write a brief that states the constraints honestly and demands four
   things: the blind spot, the abstraction, the dream, sprint surgery.
2. Give the SAME brief to two strong, *different* models. Tell them
   they are competing.
3. Moderate with verification: re-run every empirical claim yourself
   before crediting it. Expect one submission to be strategic and the
   other forensic — you need both.
4. Send the synthesis back for a second round; models catch errors in
   adjudication they don't catch in their own work (in this council,
   one caught a bug in its own round-1 formula during round 2).
5. Write laws, not aspirations. Ship sprints with exit criteria and
   pre-registered kill switches.

## License

MIT — see [LICENSE](LICENSE).
