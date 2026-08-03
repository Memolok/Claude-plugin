---
name: revise-decision
description: >-
  Change a decision that is already sealed on the ledger — uncommit and re-admit a retractable
  record, supersede it with a successor, or settle a question an earlier record left open. Use when
  the user regrets a recent commitment, says a sealed record is now wrong or out of date, wants to
  withdraw or replace a prior decision, or asks how to fix something they already accepted.
argument-hint: "<record> <what changes>"
---

# /memolok:revise-decision — Change a sealed decision

Sealed records cannot be edited in place. This skill picks the honest route: uncommit and re-admit,
or supersede with a successor.

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
- Cite `mdrNumber` for admitted records; never volunteer `mdrHandle` (Rule F)
- **Never** propose a revision because a record no longer matches the ledger's stated purpose

## Workflow

### 1. Read the record

```
get_MDR(mdlGuid, mdrHandle)
```

### 2. Route on `retractable`

| Value | State | Route |
| --- | --- | --- |
| `null` | Staged — never sealed | Just patch it; use **`record-decision`** |
| `true` | Committed, nothing depends on it yet | **Uncommit** — step 3 |
| `false` | Anchored, or already Superseded | **Supersede** — step 4 |

This is a read, not a guess. Proposing an uncommit on an anchored record wastes the user's time and
implies the ledger is more malleable than it is.

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

### 4. Supersede

The route for an anchored record, and the honest route whenever the world moved on rather than the
record being wrong.

The original stays exactly as it is — it was true at its own t₀, and that history is the point. A new
record carries the new decision and names the old one.

1. Mint a staged successor through **`record-decision`**, with its own Need, alternatives, and Verdict.
   A supersession is a real decision and needs real deliberation — including when the intent is simply
   to retire something with no replacement.
2. Patch `supersedes: [7]` on the successor while it is still staged.
3. Commit it. At admission the target flips to **Superseded** and reciprocals publish.

`supersedes` may only target **Accepted** residents, once each. A `Rejected` record cannot carry
`supersedes`.

### 5. Settle an open question

Different from both routes above. The earlier record is not changing — a question it deliberately left
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

Uncommitting to make a past decision look better is ledger fraud. The value of a record is its honesty,
not its correctness: a well-reasoned decision that failed teaches more than one retrofitted to look
prescient.

If the user wants to erase an embarrassing decision rather than correct a recording error, say plainly
that superseding is the route, and that the original standing is what makes the ledger worth keeping.

## What is never possible

- Editing a tier-1 field on a ledger resident. `retractable: true` does not mean patchable.
- Deleting anything. There are no delete tools.
- Uncommitting an anchored record — mint a successor instead.
- Superseding a `Rejected` or already-`Superseded` record.
- Recovering the old number after re-admission.
- Backdating anything. `decidedAt` is server-stamped.

## Tips

- Recording a wake usually anchors its source record, so uncommit *before* registering outcomes if
  revision is still on the table.
- `uncommit_MDR` needs admin or owner. A `member` gets a permission error and needs the ledger owner.
- Re-admission takes the next number from the live high-water mark. Uncommitting the most recent record
  and re-admitting it often returns the same number; uncommitting an older one does not.
- A supersession chain is readable history. Do not collapse or tidy it.

## References

| File | Load when |
| --- | --- |
| `references/uncommit-and-readmit.md` | Running the uncommit route |
| `references/open-question-settlement.md` | Deciding between `settlesOpenQuestion` and a direct edit |
