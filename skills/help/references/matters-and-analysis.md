# Where decisions come from

Every decision record starts from a sharpened need, expressed in the language of the business. That
creates an obvious problem: the person who feels the pain almost never speaks that language.

The user-submitted bug does not say *"authentication requests must complete in under fifty
milliseconds."* It says *"I can't log in!"*, or at best *"login works, but it takes forever."* A team lead
notices a new regulatory requirement. A developer spots a chance to drop a legacy library. Someone in
sales relays a complaint they only half understood.

These are honest expressions of a situation that might need a decision, in the words of whoever raised
them. Memolok keeps them as **matters**, separate from the need any eventual decision is built on.

## Why they are kept separate

A matter carries no truth value and no emotional charge. It is not a claim about the world and not a
problem statement — it is raw material, the thing someone raised, preserved verbatim.

Anchoring a decision directly on that raw wording is the mistake the separation exists to prevent. *"Login
is slow"* is not falsifiable, cannot be assessed later, and quietly smuggles a diagnosis into the framing:
maybe login is fine and the network is not, maybe the user's expectation is the thing that changed.

Between the two sits **analysis**: the expert act of turning raw input into something a decision can be
built on. It is recorded in its own right, preserving who did it, what they concluded, and what came out
of it. That provenance is what lets a future reader walk backwards from a sharpened need to the messy
thing somebody actually said.

Analysis can also honestly conclude that no decision is warranted. That is a real outcome, not a failure.

One analysis can take up several matters at once, and produce several decisions — or none. Three people
reporting the same fault is one act of reasoning, not three; and two unrelated reports worked together
routinely surface a problem neither person mentioned. Memolok never asks which decision answers which
report, because in that situation there is often no honest answer. What it records is what the reasoning
actually took up.

## Not everything starts as a matter

An expert who already understands the problem can go straight to a sharpened need without any raw input in
front of them. This is the ordinary path for anyone recording their own work, and there is nothing
second-class about it.

The one thing that cannot be done is going back. An analysis names the decisions it produces at the
moment it produces them, so a decision recorded directly cannot later be attached to the raw report
that turned out to have prompted it. Register the matter afterwards and analyze it, and what you get
is a second decision — not provenance for the first.

## Who does what

Three seats, and in a small team one person may occupy all of them:

- **anyone** raises a matter — stakeholder, customer, engineer, manager
- **an expert** sharpens it into a need
- **an expert** commits the decision

Recording who did which is attribution, not permission. It documents the chain of agency; it does not
control who may write to the ledger. Access is governed separately, by membership.

## The loop closes

Observations from an earlier decision are among the best sources of new matters. Something you committed
to did not hold, or something nobody predicted turned up — that becomes raw input for the next decision,
which is where the ledger stops being an archive and starts being a working instrument.

---

When they want to park one without deciding anything about it: **`save-matter`**. When they want to
take one up and decide: **`record-decision`**.
