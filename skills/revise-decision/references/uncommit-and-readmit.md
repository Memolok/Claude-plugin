# Uncommit and re-admit

## Preconditions

| Check | Requirement |
| --- | --- |
| Role | `admin` or `owner` on the ledger |
| Status | `Accepted` or `Rejected` |
| `retractable` | Must be `true` |
| User intent | Explicit correction request, in their own words |

## Step 0 — Verify eligibility

```
get_MDR(mdlGuid, mdrHandle)
```

Confirm `retractable` is exactly `true`. Never call `uncommit_MDR` speculatively.

| `retractable` | Meaning |
| --- | --- |
| `null` | Staged already — nothing to uncommit |
| `true` | Eligible |
| `false` | Anchored, or already Superseded — supersede instead |

A record is anchored when another record's `supersedes`, `amends`, `dependsOn`, `enables`, or
`conflictsWith` cites its number, when one of its open questions has been settled, or when any observed
outcome was realized from it — and, fourth, when someone has **declared** that something outside the
ledger cites it. That declaration is `anchor_MDR`; the ledger cannot observe an external citation, so
it is asserted rather than detected, and there is no way to withdraw one.

## Step 1 — Uncommit

```json
{
  "mdlGuid": "<mdlGuid>",
  "mdrHandle": 1,
  "reason": "Verdict cited the wrong latency target; no other record references this yet."
}
```

The record drops back to a staged status, and `mdrNumber`, `decidedAt`, and `retractable` all clear.
The response comes back with `retractable: null` and `mdrNumber: null`.

`reason` is optional in the schema and mandatory in practice — the audit plane keeps both the commit
and the uncommit permanently, and a blank reason makes that history useless.

## Step 2 — Edit while staged

```json
{
  "mdlGuid": "<mdlGuid>",
  "mdrHandle": 1,
  "patch": {
    "verdict": {
      "description": {
        "markdown": "We will add a Redis read-through cache for hot read paths, targeting P99 under 200ms.",
        "lang": "en"
      }
    }
  }
}
```

Every tier-1 field is patchable again, as are the graph edges.

## Step 3 — Re-admit

```json
{ "mdlGuid": "<mdlGuid>", "mdrHandle": 1, "status": "Accepted" }
```

Run the ceremony from **`commit-decision`** first. This is a fresh t₀ with a fresh `decidedAt`, not a
resumption of the original commitment.

A **new** number is assigned from the live high-water mark. Uncommitting the most recent record and
re-admitting it usually returns the same number; uncommitting an older one leaves its original number
permanently unused.

## Telling the user what it costs

Before step 1, three things:

> Uncommitting **MDR-7** takes it back to editable so we can fix the Verdict. Two things worth knowing:
> the ledger keeps both the original commitment and the uncommit permanently — this is visible
> governance history, not an undo — and when we re-admit it, it gets a **new** number. MDR-7 as a
> reference goes away.
>
> Go ahead?

Then wait.

## Errors

| Message | Cause |
| --- | --- |
| `Only Accepted or Rejected Memolok Decision Records may be Uncommitted.` | Record is staged already |
| `This Memolok Decision Record is Anchored and cannot be Uncommitted. Use amend or supersede instead.` | Something depends on it |
| A permission error | Caller is a `member`, not `admin` or `owner` |
| `Cannot change the {label} on a ledger-resident Memolok Decision Record ({status}).` | Patch attempted before the uncommit landed |

## Anti-patterns

**Patching first and uncommitting when it fails.** Read `retractable`, then act. The failed patch tells
the user the tool is fighting them.

**Uncommitting to improve how a decision looks.** The ledger's worth is its honesty. A decision that
turned out badly is evidence, not an error to be tidied — supersede it and let both records stand.

**Uncommitting an anchored record because `retractable` was not read.** The error message is clear, but
by then the user has been offered something that was never available.

**Treating re-admission as resuming the original commitment.** It is a new t₀ with a new timestamp and
a new number. Anything that cited the old number now cites nothing — or worse, comes to cite whichever
record takes that number next. **This is what `anchor_MDR` exists to prevent**: a record declared as
cited outside the ledger refuses the uncommit outright, so the citation can never go wrong. Declaring
it is the writer's job at the moment of citing, not something anyone can add afterwards.
