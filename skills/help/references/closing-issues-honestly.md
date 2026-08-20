# Closing things honestly

Two organizational pathologies account for a remarkable amount of wasted senior time.

**The phantom decision** — an issue nobody ever actually resolved, which everyone glides over as though it
had been. It is in nobody's queue, it has no owner, and it surfaces as confusion months later when two
teams turn out to have assumed opposite answers.

**The zombie decision** — an issue that was discussed and turned down, but nobody remembers why, so it
comes back every planning cycle to be argued again from scratch.

Both are failures of closure, not failures of decision-making. Closing an issue honestly is not
administrative overhead; it is how you stop the bleeding.

## The honest ways matters come to rest

A raised matter can come to rest in several ways, and they are genuinely different:

| How it came to rest | The story it tells |
| --- | --- |
| **Fixed** | We decided to act, and later evidence confirms it worked |
| **Refused** | We agree the problem is real, but we are choosing not to act — here is who said so and why |
| **Externally blocked** | We know what to do, but cannot do it yet; a constraint is in the way |
| **No decision warranted** | We investigated, but nothing here needed deciding |
| **Overtaken by events** | The world removed the problem before we got to it |

A matter with none of these is simply still open — which is exactly what the ledger says about it,
without dressing it up as anything more settled.

The distinctions carry weight. **Refused** and **externally blocked** both mean nothing is happening, but
the first is an exercise of will and the second is a constraint — and a year later, that is the difference
between "revisit if the cost changes" and "revisit when the regulation lifts". Both are commitments backed
by real reasoning and an accountable decider.

**Overtaken by events** is the one people feel guilty about, unnecessarily. Sometimes the world simply
removes the problem: the customer sells the property before anyone looks at the cold living room. No
investigation theatre is required to satisfy a methodology. A world event closes it cleanly. If the
subject returns later, it returns as something new, not as a reopening.

**Fixed** is stronger than it looks — it is not "we shipped the fix", it is "we shipped the fix and the
world confirms it worked". The decision commits to act; confirmation registers only when evidence arrives.

## These are readings, not labels

Nothing writes one of these five words onto a matter. There is no status field to set, and no menu to
pick from. Each reading is just what a particular shape of record looks like when you read it back: which
analyses took the matter up, whether they concluded, and which decision or world event closed it.

That is deliberate, and it costs something worth naming. You cannot ask the ledger "what is the status of
this matter" and get one word back, because for a matter worked alongside three others, no single word
would be true of it. What you get instead is the shape — who looked at it, when, what they concluded, and
what has closed it since — which carries more than a label could, including *which* decision closed it and
*when*.

The practical consequence: two people reading the same matter may describe it slightly differently, and
neither is wrong. What they cannot do is disagree about the underlying facts, because those are all
recorded.

> **Not built yet.** Only two of these are reachable through this plugin today: analysis that concludes no
> decision is warranted, and analysis that produces a decision. Recording that a matter was refused,
> externally blocked, or overtaken by events is modelled but has no tool yet. Until it is, record the
> decision that closes the topic — the reasoning is the part that stops the zombie — and expect the matter
> itself to keep showing up as one nobody has picked up.

---

When they want to record the decision that closes a topic: **`record-decision`**, then
**`commit-decision`**.
