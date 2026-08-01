# Ceremony variants

## Accepting

> **Ready to seal this on the ledger.**
>
> You're committing to **a Redis read-through cache on the hot read paths**, on the grounds that it
> addresses P99 latency directly where horizontal scaling would not.
>
> The moment I commit, the server stamps the decision time and the record seals — Verdict,
> alternatives, expected outcomes and open questions become immutable ledger evidence. Your expected
> outcome, *P99 under 200ms within 30 days*, becomes the bet this record gets measured against later.
>
> **Still open by design:** whether the write path needs separate treatment. That travels with the
> record.
>
> Shall I commit this as **Accepted**?

## Rejecting

Rejection deserves the same weight. Do not apologize for it or frame it as a failure.

> **Ready to close this out as Rejected.**
>
> You're committing that the team will **not** migrate the monolith to microservices under this need —
> the cost exceeds the benefit, the operational maturity isn't there, and the modular monolith is
> meeting its SLOs.
>
> This is a real ledger entry, not an abandoned draft: it gets a number, and it records who decided
> and why. Next time this comes up, the reasoning is already there.
>
> Shall I commit this as **Rejected**?

## One-shot mint

When the user committed before any record existed, surface the same implication *before* calling
`create_MDR` at `Accepted` or `Rejected`:

> Before I write this — minting it as **Accepted** seals it immediately. The Verdict and the expected
> outcomes become immutable the moment it lands, same as any commitment. Happy to go ahead?

Then mint, then verify with `get_MDR`.

## Terse mode

Some users find ceremony tiresome, particularly when sealing several records in a row. Compress, but
keep the three load-bearing parts: **what is being committed**, **that it seals**, and **an explicit
ask**.

> Sealing **MDR-8** — Redis cache on hot read paths. This freezes the body permanently. Go ahead?

What must never be dropped is the ask. Everything else is style.

## Marking the moment afterwards

One beat, using the number the record just received:

> **MDR-7 is Accepted** — sealed just now. That's on the ledger.

Occasionally a little more, when the decision was hard-won or long-running:

> **MDR-7 is Accepted.** That's the storage question closed after three sessions of going back and
> forth — and the reasoning is on the record now, not just the answer.

Then continue. Do not offer to end the session, and do not follow a seal with a menu of what to do
next unless the user asked for one.

## What ceremony is for

Not formalism. It exists because t₀ is the one irreversible moment in the whole model, and a user who
did not realize they were crossing it has been badly served.

Three things it must do:

1. **Say what is being committed**, in the user's own terms — not "MDR-7" but the actual choice.
2. **Say what seals**, in plain language — no acronyms, no field names.
3. **Ask, and wait.** An explicit yes, then the tool call.

## Anti-patterns

**Bundling ceremony into the recap:**

> *"Here's the draft — shall I persist it and accept it?"*

Two steps collapsed into one. The user cannot endorse accuracy and commit separately.

**Inferring commitment from your own prose:**

> *"This reads like a firm commitment — want me to mint it straight to Accepted?"*

The Verdict sounding decisive is a fact about your writing, not about their intent.

**Offering Proposed on informal work:**

> *"…or shall we hold it at Proposed for now?"*

There is no governance body here. The honest options are seal or leave it at Deliberating.

**Treating open questions as blockers:**

> *"Before we can Accept, we should close these open questions."*

They are scope boundaries. Name them in the ceremony, seal anyway.

**Ceremony with no ask:**

> *"Committing this now — the record seals at this point. MDR-7 is Accepted."*

Narrating the ceremony while doing it regardless. The pause is the entire point.
