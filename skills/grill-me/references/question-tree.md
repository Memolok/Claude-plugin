# The question tree, phase by phase

## Phase 1 — Entry

One question, to sort what the user actually brought.

> **Question:** Is this something that's gone wrong that you want to fix, or do you already know what
> you're trying to achieve?
>
> **Recommended answer:** Sounds like the second — you opened with a target rather than a symptom.
>
> **Why:** It decides whether we start from the raw matter or straight from the objective, and the
> two get recorded differently.

Skip this entirely when the conversation already makes it obvious. The five-way routing lives in
`record-decision`; the two that matter here are bait and head Claim.

If it is bait, capture the user's words **now**, verbatim, in the draft. They are the raw signal, and
they will not survive three turns of paraphrase.

## Phase 2 — Head

Sharpen the Need into something a future observation could settle, without naming the mechanism.

> **Question:** What would have to be true for this to count as solved?
>
> **Recommended answer:** Something like *"the library must survive a single drive failure without
> data loss"* — a property, not a particular setup.
>
> **Why:** That's the head of the record, and it needs to be checkable later without pre-committing to
> how we get there.

**The recommended answer here is a framing, never a mechanism.** If you catch yourself recommending
*"use RAID1"*, that is phase 3 content leaking into phase 2.

Watch for the user handing you a mechanism. *"I need to set up RAID"* is a Verdict looking for a Need —
ask what it would protect against and let the real Need surface.

**This gate is provisional.** If phase 4 reveals the Need absorbed something only discovered during
deliberation, walk it back. See the `need-vs-verdict-drift.md` reference in `record-decision`.

## Phase 3 — Belly, options

**Draft first.** Read back what they already mentioned rather than asking them to generate a list.

> Sounds like you've already got two on the table: a single drive with scheduled backups, and a RAID1
> mirror. Anything else you looked at and set aside?

That is a confirmation with an opening, not a demand for options.

If exactly one path has ever been in play, one light check is fair:

> Was doing nothing ever on the table, or was it clear from the start that something had to change?

Once. If the answer is "it was always going to change", record that and move on. Do not run a
brainstorm to fatten the record.

## Phase 4 — Belly, arguments

Get their reasoning about the options, not yours.

> **Question:** What made you lean away from the RAID mirror?
>
> **Recommended answer:** I'd guess it doesn't cover the failure you're actually worried about — the
> flat you live in, not the drive.
>
> **Why:** That reasoning is the belly of the record; it's what makes the choice legible later.

Attach each argument to the option it concerns. Prefer their words, even when yours would be tidier.

## Phase 5 — Waist

The Verdict and the chosen option, or the reason for not proceeding.

> **Question:** So — single drive plus scheduled off-device backup?
>
> **Recommended answer:** Yes, on your own reasoning: it covers fire and theft, which the mirror
> doesn't.
>
> **Why:** This is the moment of commitment the record exists to capture.

Offer **Rejected** as readily as an Accept. A recorded "we will not" is a real decision — and not only
when deliberation ran out of options. If the answer to the Claim is *no* while the user gets on with
something else, that is still a Rejection, and it is a second record rather than a caveat inside the
first. The `memolok-method` skill carries the full set of situations.

## Phase 6 — Tail

Expected outcomes — the bets this decision makes.

> **Question:** If this works out, what should be true in six months? And what's it going to cost you?
>
> **Recommended answer:** No data loss from a single drive failure, and roughly ten minutes a month
> of backup babysitting.
>
> **Why:** These are what the record gets measured against later — the difference between a decision
> you can learn from and one you can only remember.

Push gently for something checkable. *"It'll be better"* is not a bet.

Cover gains, costs, risks, and dependencies — but as prose, one question, not a four-part form.

## Phase 7 — Alongside

What this decision deliberately does not settle.

> **Question:** Anything we've touched on that you're consciously not deciding right now?
>
> **Recommended answer:** The backup destination and schedule — we circled it twice without settling
> it, and it doesn't block the storage choice.
>
> **Why:** Recording it as an open question keeps this decision tight instead of letting it sprawl.

Frame these as **scope protection**, never as a to-do list. If the user starts trying to clear them,
say plainly that they travel with the record and do not block anything.

## Phase 8 — Persist

Preview, confirm, persist at **Deliberating**. Then straight into the next branch.

## Worked openers

**Blank page:**

> Right — let's start with what's actually bothering you about the current setup. What made this come
> up now?

**User arrives with a solution:**

> Before we get to whether Postgres is the right call, what would switching fix? I want to get the
> problem on the record before the answer.

**Picking up parked bait:**

> There's a matter sitting on the ledger from a while back — *"the CSV export mangles unicode"*.
> Is that what we're working on, or is this something else?

**Continuing after a seal:**

> That's MDR-7 sealed. You left the backup destination open on it — let's take that one now. What are
> you actually backing up to?

Note the last one: no menu, no asking whether to continue. Straight into the next fish.

## When the user pushes back

**"Just record what I said."** Fine — that is the point of the tool. Drop to a preview and persist.
Grill-me is a service, not a toll.

**"I don't know."** Legitimate. Either it is a fact you should look up, or it is a genuine open
question — record it as one and move on.

**"Does this matter?"** Answer honestly. If a region is thin because the decision genuinely does not
have that content, record it thin. A short honest record beats a padded one.
