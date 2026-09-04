# Retractable and anchored

Committing a decision is not the same as freezing it forever, and the difference is the part people
usually get wrong in both directions.

## The two states after commitment

**Retractable** — the decision is committed, but nothing else in the world has leaned on it yet. No other
decision cites it, nobody has declared that something outside the ledger cites it, no evidence has been
attached to it, none of
its open questions have been settled. In that window, a privileged owner can withdraw the commitment: the
record returns to draft, its public number goes back into circulation, and both the original commitment
and its withdrawal stay permanently visible in the ledger's history.

**Anchored** — something now relies on it. From that point the decision's substance is sealed. Changing
course means a new decision that supersedes this one, not a rewrite of it.

Three of those dependencies the ledger sees for itself, because they happen inside it. The fourth it
cannot: a decision quoted in a design document, an email or a ticket is relied upon just as heavily,
and nothing about that reaches Memolok. So it is **declared** — whoever writes the citation says that
they have, and the record stops being withdrawable from then on. The declaration records only that a
citation exists and roughly what kind of place it lives in. It never records where, because a stored
location goes stale the moment a file moves and then reads as authoritative.

There is no undoing a declaration, which is the point of making one. Nothing verifies it either —
Memolok cannot see your files or your mail — so it is a commitment made in good faith, and it is
worth exactly as much as the discipline behind it.

## Why it is called a transaction

In law and accounting, a transaction is a single dated entry recording a commitment made at a specific
moment: at time *t*, we did this. Once later work depends on that line, it is never rewritten in place. If
a posting was premature and nothing has yet depended on it, the books can retract it — but the journal
still shows that it was posted and withdrawn.

Decisions are treated exactly that way. The commitment date is the posting date, the record is the entry,
and the ledger history is the journal that cannot be quietly tidied.

Engineers will recognize the family resemblance to a database commit, with one addition: a governed
withdrawal path that stays open only while nothing has taken a dependency.

## What this is not

It is not a versioning system, and retraction is not an undo button for embarrassment. Withdrawing a
commitment is itself a recorded act with a stated reason. The point of allowing it at all is narrow: a
posting made in error, before anyone relied on it, should not force the ceremony of a full superseding
decision — but it should still leave a trail.

Nothing here softens the rule in the other direction. An anchored record is never patched to look better,
and a retractable one is never silently rewritten. The choice is withdraw-on-the-record or supersede; it
is never edit-in-place.

Whether a specific record is still retractable is a fact the ledger computes rather than a judgment call.
Read it before promising anyone that something can be taken back.

---

When they want to withdraw or replace a sealed decision: **`revise-decision`**.
