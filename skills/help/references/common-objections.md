# Fair objections

Answer these honestly, including the parts where the objection lands. A defensive answer convinces nobody
who has actually run a documentation initiative into the ground.

## "Isn't this just ADRs?"

Same lineage, and said without embarrassment. The practice of writing short records for architecture
decisions was popularized by Michael Nygard, developed by a number of people after him, and it worked —
that is why it spread.

If lightweight decision records are already working for your team, say so. Nobody needs replacing a habit
that is doing its job.

What Memolok adds is what the practice tends to lose at scale:

- **It is not about software.** The same shape holds for legal, medical, procurement, or a renovation.
- **Commitment is a distinct moment**, not a status field someone updates later. What sealed, sealed.
- **You commit to what you expect**, so the decision can be assessed later instead of merely remembered.
- **Deferrals are explicit**, so what a decision did not settle is recorded rather than assumed.
- **Withdrawal is a decision**, not a status change — nothing quietly becomes obsolete.

The honest summary: ADRs are a document practice, this is a ledger. If your records already answer *what
did we expect, and did it happen*, the difference is mostly rigor.

## "This is bureaucracy"

Partly true, and worth naming rather than dodging. Recording a decision properly costs something at the
moment of commitment. There is a real discipline here and it is not free.

Two things bound the cost. Memolok is opinionated about **recording** and neutral about **deciding** — it
adds no approvals, no sign-off chain, no meeting. Informal work skips the formal proposal step entirely
and goes from deliberation straight to commitment.

And the cost lands once, at commitment, while the saving lands every time somebody would otherwise have
re-litigated the question, reconstructed the reasoning from memory, or repeated a mistake whose lesson
walked out of the building.

If it is genuinely producing overhead without producing answers, record less: the decisions that are
expensive to reverse.

## "Our decisions are too small for this"

Size is not the test. **Irreversibility** and the **cost of re-deciding** are.

A one-line configuration change you will never revisit needs no record. A choice of naming convention you
will live with for three years, and which everyone will eventually forget the reason for, is exactly the
kind of thing this catches — small, cheap to make, expensive to be confused about later.

## "We'll write it up afterwards"

Everyone says this, and the reasoning is precisely what decays. Afterwards, the result is obvious and the
alternatives you rejected are gone: you no longer remember what almost won, or which constraint killed it.
What gets written is a justification of what happened, not a record of what was decided.

The counterpoint is real, though, and worth offering: a decision made last year is still worth recording
if whoever made it can still recover the reasoning. Late is far better than never. Just do not plan for
late.
