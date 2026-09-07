---
name: revise-decision
description: >-
  Change a decision that is already sealed on the ledger — amend part of it, supersede it with
  a successor, uncommit and re-admit a retractable record, or settle a question an earlier record
  left open. Use when the user regrets a recent commitment, says a sealed record is now wrong or out
  of date, says part of it no longer holds, wants to withdraw or replace a prior decision, or asks
  how to fix something they already accepted.
argument-hint: "<record> <what changes>"
---

# /memolok:revise-decision — Change a sealed decision

Sealed records cannot be edited in place. This skill picks the honest route: amend part of it,
supersede with a successor, or uncommit and re-admit.

## Usage

```
/memolok:revise-decision <which record> <what needs to change>
```

Examples:

- `/memolok:revise-decision MDR-7 — the latency target in the verdict is wrong`
- `/memolok:revise-decision we're replacing the caching decision with a read-replica approach`
- `/memolok:revise-decision undo that, I accepted it by mistake`
- `/memolok:revise-decision MDR-3 settles the open question on MDR-7`

## Step 0 — Load the method

Load the **`memolok-method`** skill. The Decision Transaction Principle is the whole subject of this
skill.

## Non-negotiables

- Never invent `mdlGuid`, `mdrHandle`, or `mdrNumber` — the server mints them
- All prose is `{ "markdown": "...", "lang": "en" }`
- **Read `retractable` before proposing anything** — it decides the route
- Never tell the user a committed record can be patched in place
- Say "captured" only after the write succeeded
- Cite `MDR-{n}` for admitted records, `MDRh{handle}` for staged ones; never a bare handle (Rule F)
- **Never** propose a revision because a record no longer matches the ledger's stated purpose

## Workflow

### 1. Read the record

```
get_MDR(mdlGuid, mdrHandle)
```

### 2. Route on how much is wrong

**Ask this first.** `retractable` decides only whether one of the routes is still open; it does not
decide which route is right.

| How much of the record is wrong | Route |
| --- | --- |
| Staged — never sealed | Just patch it; use **`record-decision`** |
| Nothing; it should never have said that | **Uncommit** — step 3, needs `retractable: true` |
| Part of it; the rest still governs | **Amend** — step 4 |
| All of it; the decision is withdrawn | **Supersede** — step 5 |

Then read `retractable`, which is a read and not a guess. `null` staged, `true` uncommittable,
`false` anchored. Proposing an uncommit on an anchored record wastes the user's time and implies the
ledger is more malleable than it is. Amend and supersede work at either value.

### 3. Uncommit and re-admit

Requires `admin` or `owner` on the ledger, and status `Accepted` or `Rejected`.

Best for a genuine mistake caught early — a wrong figure in the Verdict, an outcome that was never
meant to be there — where nothing has come to depend on the record yet.

1. **Say what it costs.** Uncommit is governance-visible: the commit and the uncommit both stay on the
   audit plane permanently, and re-admission assigns a **new** ledger number. The old number does not
   come back.
2. `uncommit_MDR(mdlGuid, mdrHandle, reason)` — give a real reason; it is part of the record.
3. `update_MDR` while staged. Everything is patchable again.
4. `transition_MDR_status` to re-admit, with the ceremony from **`commit-decision`**. This is a fresh
   t₀, not a resumption of the old one.

Detail and payloads: `references/uncommit-and-readmit.md`.

### 4. Amend

For a record that still stands, where part of it has changed — and retiring the whole thing to say so
would withdraw a decision nobody disputes.

**Amendment is opt-out; supersession is opt-in.** Everything in the amended record stays valid except
what your record explicitly changes or removes. Everything in a *superseded* record stops being valid
and only the successor remains. Either way, whatever your record adds is valid, as in any record.

Two things follow, and the second is the one that gets missed:

- **Any partial change qualifies** — tightening a commitment, widening it, adding one that was
  missed, dropping one that no longer applies, or saying what a clause always meant. There is no
  separate instrument for clarifying; it is the same edge.
- **Write the deltas only.** Because everything else carries forward, restating the original does not
  amend it — it duplicates it. An amending record that reads like a rewrite of the original is doing
  supersession's job with the wrong instrument.

The original stays **Accepted** and keeps governing. A new record states the change and names what it
changes.

1. Mint a staged successor through **`record-decision`**. Its Need is the specific question — *what
   should this part say now* — not a restatement of the original decision.
2. Patch `amends: [7]` on the successor while it is still staged.
3. Commit it. At admission the target gains `amendedBy` pointing back, so a reader who opens the
   original sees that something later qualified it.

**Say what it costs before writing it.** The amendment Anchors the record it names, permanently: the
target can never be Uncommitted again by anyone. Amending sounds lighter than superseding and spends
that option just as finally. If the user might still want to take the original back, amend later.

Both ends must be **Accepted**, for the same reason. Only an Accepted record has content still in
force for the opt-out default to carry forward. A **Rejected** record can neither amend nor be
amended — nothing it says is in force, so there are no deltas to apply and nothing to apply them to —
and a **Superseded** target has already had its whole content retired.

### 5. Supersede

The honest route whenever the world moved on rather than the record being wrong, or when the decision
is withdrawn outright.

The original stays exactly as it is — it was true at its own t₀, and that history is the point. A new
record carries the new decision and names the old one.

1. Mint a staged successor through **`record-decision`**, with its own Need, alternatives, and Verdict.
   A supersession is a real decision and needs real deliberation — including when the intent is simply
   to retire something with no replacement.
2. Patch `supersedes: [7]` on the successor while it is still staged.
3. Commit it. At admission the target flips to **Superseded** and reciprocals publish.

`supersedes` may only target **Accepted** residents, and a `Rejected` record cannot carry one — the
same requirement as amendment, for the same reason: only an Accepted record has content to retire.
Once each, too, and that falls out of the rule rather than sitting beside it: superseding flips the
target to **Superseded**, so a second attempt finds nothing left.

### 6. Settle an open question

Different from every route above. The earlier record is not changing — a question it deliberately left
open is being answered by a later decision.

| Holder's state | Route |
| --- | --- |
| Admitted (`mdrNumber` set) | The resolver patches `settlesOpenQuestion: [{ hostMdrNumber, openQuestionId }]` while staged, then commits |
| Still staged | Update the **holder** directly — remove or fold the question, cite the resolving MDR in prose |

The deciding factor is the holder's status, not which record came first. A staged holder has no frozen
question and no number to target.

Settling a question **anchors** the holder, so it can no longer be uncommitted.

Both patterns: `references/open-question-settlement.md`.

## Which route is honest

The mechanics follow from `retractable`, but when both are available the question is *what actually
happened*:

| What happened | Route |
| --- | --- |
| We recorded it wrong | Uncommit and re-admit — the record never should have said that |
| We were right then, and the world changed | Supersede — both records are true at their own t₀ |
| We were right then, and we now know we were wrong | Supersede — the original is honest evidence |
| The decision stands and shipped; something it *promised* was wrong | **Amend.** The Verdict is sound and the implementation is live; one commitment is not |

That fourth row is the one that gets forced into the wrong route. A record whose Verdict is sound and
whose implementation is live does not need revising because one expected outcome was badly worded;
uncommitting it would be revision of a decision nobody disputes, and superseding it would retire
something still in force. What is actually being decided is whether to honour the commitment — a new
question, at a new t₀ — and `amends` is what says the new record answers it without retiring the old.

**One variant still has no link back.** If that new record admits as **Rejected** — declining the
promise rather than replacing it — it can carry neither `amends` nor `supersedes`, because nothing it
says is in force to change or retire anything with. The pair is then connected only by the successor
citing the wake in `hasContext` and naming what it declines in prose. Write that deliberately;
nothing else will.

Uncommitting to make a past decision look better is ledger fraud. The value of a record is its honesty,
not its correctness: a well-reasoned decision that failed teaches more than one retrofitted to look
prescient.

If the user wants to erase an embarrassing decision rather than correct a recording error, say plainly
that superseding is the route, and that the original standing is what makes the ledger worth keeping.

## What is never possible

- Editing a tier-1 field on a ledger resident. `retractable: true` does not mean patchable.
- Deleting anything. There are no delete tools.
- Uncommitting an anchored record — amend or supersede instead.
- Amending or superseding anything but an `Accepted` record, or carrying either on a Rejection.
- Superseding the same record twice.
- Un-anchoring anything. Amending a record spends its Uncommit path for good.
- Dissolving a `conflictsWith` pair. It Anchors both records, including the one that declared it.
- Recovering the old number after re-admission.
- Backdating anything. `decidedAt` is server-stamped.

## Tips

- Anchoring can also be **declared**, by `anchor_MDR`, when something outside the ledger cites the
  record. It is permanent and unverifiable, so a record can be un-uncommittable for a reason the
  graph does not show — read `retractable`, never infer it from the edges. **The refusal names the
  kind**, so `Anchored (project)` or `Anchored (other)` tells you it was declared rather than derived.
- Recording a wake usually anchors its source record, so uncommit *before* registering outcomes if
  revision is still on the table.
- `uncommit_MDR` needs admin or owner. A `member` gets a permission error and needs the ledger owner.
- Re-admission takes the next number from the live high-water mark. Uncommitting the most recent record
  and re-admitting it often returns the same number; uncommitting an older one does not.
- A supersession chain is readable history. Do not collapse or tidy it.

## References

| File | Load when |
| --- | --- |
| `references/uncommit-and-readmit.md` | Before proposing an uncommit — it is governance-visible and costs the ledger number permanently, which the user should hear before agreeing |
| `references/open-question-settlement.md` | Before authoring a settlement — the route turns on the holder's status, and sealing two related records in the wrong order destroys the edge |
