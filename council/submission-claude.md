# Council Submission — Claude Opus 4.5

## 1. Unique Insight: The Blind Spot

**The system has no theory of commitment gradients or stake awareness.**

Every actionable signal is treated identically. The idle gate asks "is there work?" but never "how much does this decision matter?" The completion judge catches lies but doesn't know if the task was critical or trivial. There's no concept of:

- **Hard commits** (definitely doing, failure is unacceptable)
- **Soft commits** (planning to do, can be deprioritized)
- **Watch list** (monitoring, not acting, but these inform judgment when signals arrive)
- **Explicit rejections** (things NOT being done, which is as important as things being done)

The TASKS.md board is flat. A task is either there or not. But real agency has *background hypotheses* — things the agent is tracking that don't consume cycles but shape interpretation of future signals. Right now, when zero gets an issue, it either acts or doesn't. It can't say "I'm watching this pattern across issues and will act when it crosses a threshold."

**The deeper blind spot:** The agent models the world (issues, DMs) but has no *predictive model* — no expectation of what *should* happen next. This is the difference between a reflex system and a cognitive one. A cognitive agent notices when reality diverges from expectation. Zero can only notice what's explicitly signaled to it.

This matters because: the idle gate's implicit assumption is that all non-actionable moments are equally uninteresting. They're not. Some silences are suspicious. Some are expected. An agent that knows what *should* be happening can detect meaningful absences, not just meaningful presences.

---

## 2. Abstraction: Name the Pattern

The brief says "cheap judgments with expensive reasoning as exception" is taken. Here's what's deeper:

**This is an Ambient Agent.**

Not an agent that "runs" — an agent that *exists* as a continuous presence with variable activation. The analogy: peripheral vision. Your periphery is always processing, but only pulls conscious attention when something moves. Zero is becoming that: always parsing, rarely thinking.

But "ambient" alone undersells the architecture. The tighter name:

**Confidence-Gated Presence**

The defining characteristic isn't cheap vs. expensive cognition — it's that *uncertainty is the trigger*. High confidence either direction (definitely work, definitely idle) routes to fast paths. Low confidence escalates. The System-1 layer isn't just a cost optimizer; it's a *calibrated uncertainty detector*.

This is different from tiered compute or hierarchical agents. Those route by *type* of request. This routes by *epistemic state*. The agent's meta-cognition ("how sure am I?") is itself the routing key.

If I had to pick one name to defend: **Calibrated Ambient Agency** — always present, variably active, with uncertainty as the dial.

---

## 3. The Refined Dream (12-36 months)

Here's where this is heading:

**Month 12:** Zero runs every 10 minutes (144 cycles/day). 97% are System-1 only. Memory has evolved from flat MEMORY.md to a **semantic commit log** — structured facts with timestamps, confidence, and source links, stored as markdown but indexed by cheap embeddings. The agent doesn't just remember — it can query "what did I believe about X six weeks ago and why did it change?"

The completion judge has generalized into a **self-audit layer**: before any memory write or output, a cheap model sanity-checks the claim against the task frame, the recent context, and known failure modes. The agent catches its own hallucinations before they hit git.

**Month 18:** Multiple zero-family instances exist. One watches the main project. One monitors a constellation of open-source dependencies (security advisories, breaking releases). One handles inbox triage. They share a **world-state bus** — not gossip for gossip's sake, but a minimal contract: "here's what I just learned that might matter to you." Cross-agent context without cross-agent interference.

The human-facing layer is transformed: Casey doesn't see "zero ran." She sees **compressed reports**: "3 signals today, 1 action taken (opened PR for dep bump), 2 recommendations pending (test flake pattern detected, auth error spike correlates with last deploy)." The agent has learned what she finds surprising vs. obvious.

**Month 30:** The agent has graduated autonomy. It earns trust on small actions (closing stale issues, updating deps) and surfaces big decisions. Trust is a *measurable stat*, not a binary. Self-audits that pass consistently raise the autonomy ceiling; self-audits that catch near-misses lower it.

The physical periphery exists now: an Arduino with a temp/light sensor feeds environmental signals. Not for action — for context. The agent knows Casey's home office is cold and dark (evening, winter), and adjusts its notification urgency accordingly. A second unit watches the mailbox; a package arrival is just another signal in the relevance stream.

**Day in the life:** Casey wakes. Phone shows one notification: "Handled overnight: dep security patch merged, CI green. Pending your decision: repo X went silent for 14 days, historical pattern suggests maintainer burnout — recommend checking in or forking." She replies "fork it" via Telegram. By the time she's at her desk, the agent has created the fork, opened an issue explaining the fork rationale, and drafted a message to the original maintainer. She reviews, edits one line, sends.

The agent isn't her assistant. It's her **peripheral nervous system** — always processing, rarely interrupting, taking reflex-level actions silently, escalating to conscious attention only what requires her judgment.

---

## 4. Sprint Surgery: Three Changes

### KILL: Arduino reflex nodes in the R&D lane (for now)

**Reasoning:** The core agent doesn't have graduated trust yet. Adding physical peripherals before the software trust model is solid creates a larger attack surface and a debugging nightmare. The failure mode "agent mistakenly took physical-world action" is much worse than "agent mistakenly wrote to git." Physical periphery should come *after* self-audit and autonomy gradients are proven.

**Move to:** Post-Sprint-3, gated on completion judge accuracy >99% over 8 weeks.

### ADD: Stake/commitment model in Sprint 1 (not Sprint 3)

**Reasoning:** This should be wired in shadow mode from day one. Every task should get a commitment-level tag (hard/soft/watch/reject) and a stake estimate (blast radius if wrong). Even if the agent doesn't act on these yet, logging them builds the dataset for future confidence calibration. You can't train a stake-aware model without stake-labeled data.

**Implementation:** Add two fields to TASKS.md entries. System-1 model classifies incoming signals on both dimensions. Log disagreements between System-1 and eventual human/LLM override.

### REFRAME: "Relevance-ranked recall" → "Queryable memory with temporal awareness"

**Reasoning:** "Relevance-ranked recall beyond 2000-char" is vague about what relevance means. The actual capability needed is: the agent can ask its own memory "what did I believe about X?" and "what changed about X since date Y?" This is different from similarity search. It's temporal + semantic.

**Concrete change:** Don't build embeddings-and-ANN-search (the generic approach). Build a structured fact log with explicit timestamps, confidence, and source citations. Retrieval is SQL-style queries over structured fields, not vector similarity. Cheaper, more interpretable, and the failure modes are debuggable.

---

## Final Remark

The real unlock isn't cadence. Cadence is the first-order effect. The second-order effect is **trust compounding**. An agent that runs 144 times per day and gets 143 of them right builds a track record. That track record becomes the license to act autonomously. The idle gate doesn't save tokens — it manufactures opportunities to demonstrate reliability.

The dream isn't a faster agent. It's an agent you stop supervising because it earned it.
