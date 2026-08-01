# Withdrawing a decision

The instinct, when a decision no longer applies, is to mark it "deprecated" — a word borrowed from
software versioning, where it means a feature still exists but is discouraged.

For a decision this is a category error. A decision is not a feature. It is a commitment of agency made at
a specific moment, and commitments do not fade.

## Nothing stops being true on its own

If a decision is no longer being followed, it did not decay or become obsolete in a vacuum. Somebody, at
some moment, concluded that the earlier commitment should no longer govern what the organization does.

That act of withdrawal is itself a decision. It has a moment, a rationale, and someone accountable for it,
and it deserves a record exactly as much as the original did.

So withdrawal is done by **supersession**:

- **The original stays.** It remains in the ledger at its original moment of commitment, preserving what
  was decided and why. Its status changes to reflect that it has been superseded.
- **A new decision governs.** The successor is committed in its own right and explicitly names what it
  supersedes.
- **Retirement is still a decision.** Even when the new direction is simply to stop doing something with
  nothing replacing it, that is a decision, and it needs the options considered and the reasoning stated.
  You do not retire old decisions by editing their status; you supersede them with an explicit decision to
  supersede them with nothing.
- **One decision may supersede several.** Overriding three earlier commitments at once, or retiring a
  cluster of them together, is normal.

The result is that the causal chain never breaks. You never find a decision that just stopped being true.
You find the moment it was withdrawn, the reasoning, and who was accountable. "We stopped doing that"
turns from an organizational shrug into an auditable event.

## Stale constraints

There is a failure mode on the other side. After a decision has been superseded, or after a fact it
depended on turns out to have been wrong, later reasoning can still treat the dead anchor as live —
someone is still designing around a constraint that no longer exists. This is a **stale constraint**, and
it is worth naming because it is invisible from inside the conversation where it happens.

> **Not built yet.** Memolok does not scan the ledger for stale constraints. Superseding a decision records
> the withdrawal accurately, but noticing that a later decision still leans on the retired one is
> currently a human job.
