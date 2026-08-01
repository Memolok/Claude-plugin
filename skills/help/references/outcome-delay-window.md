# The gap between deciding and knowing

Between committing to a decision and being able to observe its effects, time passes. The decision has to
be implemented, and then the implementation has to run long enough to move something in the world.

That span is the **outcome delay window**. It is where you plan, build, ship, and execute — everything
that happens after the decision and before the evidence.

## Why the ledger says nothing about it

Memolok is deliberately silent about that window. Sprints, tickets, migrations, and rollout plans are
execution, and execution is not decision-recording. Plenty of tools already handle it.

The silence has a purpose beyond scope discipline. By staying out of the execution window, the decision
ledger remains a clean instrument for one specific question — *was the original bet right?* — undistorted
by the noise of how the work went. A decision can be correct and executed badly, or wrong and executed
beautifully. Keeping the two records separate is what lets you tell those apart later.

## The window has no standard size

Depending on the domain and the specific decision, it can be seconds or decades. A caching change reports
back the same afternoon; a decision about materials in a building shows up in twenty years of maintenance
costs. Memolok does not prescribe what a reasonable delay looks like for your field, because there is no
general answer.

But it is worth attending to, and not as an administrative detail — as **risk exposure**. A commitment
sealed last week with no observations yet is in a completely different position from a two-year-old
commitment whose expectations have never been checked. In the first case you are waiting on the world. In
the second you may have been running on a wrong forecast the entire time, and nobody has looked.

The portfolio question worth asking is: which of our current commitments have gone longest without anyone
checking whether they held? Long latency is not failure — some bets legitimately take years. Unmeasured
latency is unmanaged risk.

> **Not built yet.** Memolok cannot currently answer that question for you, and it cannot remind you when a
> decision is due for a look. Recording an observation is fully supported whenever you make one; noticing
> that one is overdue is on you. Until it is automated, put the check in whatever calendar or tracker you
> actually read — the decision that quietly never gets assessed is the one this is designed to catch.
