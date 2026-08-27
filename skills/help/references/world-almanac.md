# The world almanac

Deciders do not only weigh options. They work against facts that are true regardless of what they choose:
*the budget is $10,000*, *our team knows Java*, *legal requires seven-year retention*, *the wall is
load-bearing*.

These are not decisions. They are the boundaries of the possible — the signposts that make some options
sensible and others impossible. Memolok keeps them in the **world almanac**: the collection of facts a
ledger reasons from, admitted once and referenced by the decisions that rely on them.

## Why they are not written into the decisions

Embedding a fact inside a decision record creates three problems at once.

The budget changes from $10,000 to $5,000, and the record cannot be edited — so it now contains a false
statement with no way to fix it honestly. Worse, the budget change was never your decision in the first
place; it is a change in the world that you have to navigate, which makes a decision record a strange
place for it to live.

And if ten decisions all leaned on the old figure, embedding gives you no way to find them.

Referencing solves all three. The decision records that it relied on the budget as it stood; the almanac
records what the budget was, and when that changed.

## A fact can also start the work

Being reasoned *from* is not the only thing a fact does. A new regulation, or a budget that just halved,
is often the reason somebody sits down to decide anything at all — and Memolok records that too: the
analysis names the admitted fact as the input it took up. The alternative would be retyping the fact as a
raw complaint so that something could point at it, which would leave the ledger holding the same thing
twice and no trace that either led to the other.

## Corrections, not edits

Facts are admitted, never rewritten. When one turns out to be wrong — you discover the budget was actually
$8,000 all along — you admit a **new** fact that corrects the old one. The original stays, because
decisions were made on it and the record of what people believed is the point.

This produces a specific and useful signal. A decision that relied on a fact which was **wrong when it was
admitted** is sitting on a **polluted premise**. It was not a bad choice — it was a rational choice on the
information available — but its foundation has moved, and somebody should look at it.

Note the precision: a polluted premise means the fact was wrong at the time. The world merely moving on is
ordinary change, and a decision made correctly under conditions that no longer hold is not polluted. It
may still deserve revisiting, which is **decision decay** — the slow erosion of decision quality by a
shifting world and nobody's attention.

> **Not built yet.** Admitting facts and recording corrections works today. Memolok does not yet scan the
> ledger to tell you which decisions relied on a fact that was later corrected, or which have decayed. The
> links are recorded faithfully, so the answer is recoverable by reading — but nothing surfaces it for you.

## Why it outlives the people

The almanac is also the team's catalogue of known knowns, and it matters most when somebody leaves. Their
decisions live in the ledger; their **worldview** lives in the almanac — the constraints they treated as
real, the assumptions they carried between projects, the facts they admitted and later corrected.

Without it, that walks out of the door with them, and one person quietly remains the only one who knows
why things are the way they are.

---

When they want to admit or review a fact: **`manage-almanac`**.
