# Deciding not to

A ledger records positions, and a position against something is still a position. **Rejected** is one
of the two things a decision can become at t₀ — not the outcome you fall back on when deliberation
fails to find a path.

Hold it as available from the moment you start shaping work, not as a status you pick at the end. It
is frequently the **first of two records** rather than a consolation prize for the absence of one.

## The symptom that says you have two records

> You are about to write a record whose Verdict spends a paragraph explaining what you are **not**
> doing, inside a record nominally about what you **are** doing.

That is two decisions wearing one record. The declined part has its own head Claim and belongs in its
own **Rejected** record; what remains is a clean **Accepted** one. Both are real, both get numbers,
and the pair is more use to a later reader than the single record that tried to carry both.

The sharper form of the same question:

**Am I declining an option, or declining a Claim?** An option is rejected *by* a record — it sits in
`alternatives`, unchosen, with the reasoning against it in the belly. A Claim is rejected *as* a
record.

## When a Rejection is the right shape

Four situations, not one. The first is the familiar case; the other three are the ones that get
missed.

**Deliberation found no viable path.** Every alternative was explored and none survives. The record
carries the alternatives and the reasoning that ruled each out.

**A Claim is declined while different work proceeds.** Nothing is stuck — the team simply will not do
what was asked, and is doing something else instead. A viable path existing elsewhere is not evidence
against a Rejection; it is irrelevant to it. This is the most commonly missed shape, because the
presence of forward motion makes the whole situation *feel* like an acceptance.

**A sealed commitment turns out to be wrong.** A record promised something, the wake shows it did not
hold, and the honest successor declines the promise rather than replacing it. Common when the
*wording* of a commitment was the error while the decision it sat on was sound and shipped.

**A proposal from elsewhere will not be taken up.** Someone raised it in good faith; the answer is no.
Recording it is what stops the same proposal returning every quarter with no institutional memory of
why it was declined last time.

## Reasoning carries the whole value

*"Rejected"* on its own is close to worthless. What a future reader needs is what was considered, what
ruled it out, and **what would have to be different** for the answer to change. Without that last
part, the next person cannot tell whether the conditions still hold, so they re-run the entire debate
— which is precisely what the record existed to prevent.

## Shaping the record

The gates, the transition matrix and the `supersedes` constraints are in `lifecycle-and-gates.md`.
What follows is judgement, not validation — the tools will not stop you.

**Do not set `chosenAlternative`.** Nothing requires one on a Rejection, and setting it reads as an
acceptance to every later reader. Alternatives may be present and unchosen; that is the record showing
its work, and it is worth including when real options were weighed.

**A Rejection is a leaf in the decision graph.** It cannot carry `supersedes`, it cannot carry
`settlesOpenQuestion`, and nothing can supersede it. The only edge that works is `hasContext` —
premises pointing *in*. So when a Rejection needs to be found from the record it relates to, prose and
context are the only carriers, and they have to do that work deliberately: say in the Verdict what
this declines and why, because no structure will say it for you.

**Give it a tail when it has one.** Expected outcomes and open questions are not required, but a
rejection with consequences worth being wrong about should commit to them — *we expect nobody to raise
this again this year*, *we expect the workaround to hold*. It accumulates wakes like any other
resident, and a rejection that turns out to have been the wrong call is exactly as informative as an
acceptance that did.

## What a Rejection is not

It is not a failed draft, an abandoned attempt, or a record that something did not happen. It was
never in force, so nothing follows from it operationally — but it is a full commitment, permanently,
as of the moment it was made.

The alternative to sealing one is leaving the question in limbo, where it returns in three months to a
different room and occasionally gets the opposite answer for no better reason than who is present.
Worse is a weak acceptance taken to close the topic, which leaves a commitment in force that nobody
believes in.
