# Lifecycle and well-formedness gates

## Status enum

`New`, `Deliberating`, `Proposed`, `Accepted`, `Rejected`, `Superseded`

`Superseded` is never a transition target. A record reaches it when another record admits carrying
`supersedes`. `supersedes` may only target `Accepted` residents, once each; `Rejected` and
already-`Superseded` targets are refused, and a `Rejected` record may not carry `supersedes` at all.

## Transition matrix

| From | Allowed to |
| --- | --- |
| *(create)* | New, Deliberating, Proposed, Accepted, Rejected |
| New | Deliberating, Proposed |
| Deliberating | Proposed, Accepted, Rejected |
| Proposed | Accepted, Rejected |
| Accepted | — |
| Rejected | — |
| Superseded | — |

`Superseded` is not a legal creation status.

## Status aliases

`transition_MDR_status` resolves three aliases:

| Input | Resolves to |
| --- | --- |
| `Decided`, `Settled`, `Committed` | `Accepted` |

**`create_MDR` does not resolve aliases.** It converts `status` directly and raises a raw
`ValueError` on anything else, which bypasses the friendly error mapping. Always send canonical
values to `create_MDR`.

## Well-formedness

Evaluated on `create_MDR` (against the requested creation status), on `update_MDR` (against the
merged document at its current status), and on `transition_MDR_status` (against the target).

| Target status | Requirements |
| --- | --- |
| All statuses | `hasNeed.description.markdown` non-empty |
| Proposed, Accepted | ≥1 `alternatives`; `chosenAlternative` matches an `alternatives[].id`; non-empty `verdict.description.markdown` |
| Accepted | ≥1 `expectedOutcomes`, each with `manifests.description.markdown`; `decidedAt` (server-stamped) |
| Rejected | Verdict present; `decidedAt`; `supersedes` **and** `amends` empty |

Gate messages:

```
A Memolok Decision Record must have a head Claim before this transition.
At least one alternative is required before proposing a Memolok Decision Record.
A chosen alternative must reference one of the record's alternatives.
A Verdict description is required before proposing a Memolok Decision Record.
At least one expected outcome is required before accepting a Memolok Decision Record.
Each expected outcome must include description prose before accepting a Memolok Decision Record.
A Verdict is required before rejecting a Memolok Decision Record.
A Rejected Memolok Decision Record must not carry supersedes (every supersedes target must become Superseded at admission).
A Rejected Memolok Decision Record must not carry amends (nothing it says is in force, so it changes nothing in the record it names).
```

The practical ladder: **Deliberating** needs only a head Claim, so a fish can live there while the
body fills in. **Proposed** forces alternatives, a chosen one, and a Verdict. **Accepted** adds
expected outcomes.

## t₀ behaviour

On transition to `Accepted` or `Rejected` the server sets `decidedAt`, assigns `mdrNumber` from the
live high-water mark, publishes graph reciprocals, appends an audit event, and computes
`retractable`. Tier-1 patches are refused from then on.

## Tier-1 fields (sealed on ledger residents)

`hasNeed`, `hasContext`, `alternatives`, `deliberationFacts`, `expectedOutcomes`, `openQuestions`,
`chosenAlternative`, `verdict`, `decidedAt`, `status`

## Patchable

| Scope | Fields |
| --- | --- |
| Staged only | All tier-1 fish fields, plus `amends`, `supersedes`, `dependsOn`, `conflictsWith` and `settlesOpenQuestion` |
| Any status | `authoredBy`, `decidedBy`, `consulted`, `informed` |
| Never | `status` (use the transition tool), `openQuestions[].settledIn`, `mdrHandle`, `mdrNumber`, `amendedBy`, `supersededBy` |

`hasContext` is `update_MDR`-only — it is not a `create_MDR` parameter — and takes an ordered list of
World Fact or prior Observed Outcome ids.

## Post-admission correction

Two questions, in this order. **How much of the record is wrong** picks the instrument; `retractable`
only says whether one of them is still available.

| What is wrong | Instrument |
| --- | --- |
| The record should never have said that | **Uncommit** — needs `retractable: true` |
| Part of it; the rest still governs | **Amend** — a successor carrying `amends`; the original stays **Accepted** |
| The whole thing; it is withdrawn | **Supersede** — a successor carrying `supersedes`; the original goes **Superseded** |

| `retractable` | What it means |
| --- | --- |
| `null` | Staged — just patch it |
| `true` | Uncommit available (admin/owner): edit while staged, re-admit under a fresh number |
| `false` | Anchored — amend or supersede |

Amend and supersede work at either value. Only Uncommit needs `true`, so read it to route *that*
decision and not the other one.

A record becomes Anchored four ways. Three the ledger computes for itself: another record's
`amends`/`supersedes`/`dependsOn`/`conflictsWith` cites its number, one of its open questions has
been settled, or an Observed Outcome was realized from it. Recording a wake therefore usually Anchors
the source.

**Amending permanently anchors what you amend.** `conflictsWith` is heavier still: it
is symmetric, so both records end up Anchored — *including the one that declares it* — and nothing
dissolves the pair.

The fourth is **declared**, by `anchor_MDR`, and says something outside the ledger cites the record —
the ledger cannot see that for itself. It is not inferred from anything you write; you declare it.
Form and precondition: the **`memolok-citations`** skill.

## Graph edges on staged records

`amends`, `supersedes`, `dependsOn` and `conflictsWith` take arrays of admitted `mdrNumber`s. Author
them while the record is staged; once it is a resident, the field is refused, and that includes
sending an empty `[]` — presence is what is refused, because clearing an edge after admission would
undo something the ledger has published.

| Edge | Says | Target's fate at your t₀ |
| --- | --- | --- |
| `amends` | that record stays valid *except* where this one changes or removes something | stays **Accepted**, Anchored |
| `supersedes` | none of that record stays valid; only this one remains | becomes **Superseded** |
| `dependsOn` | this decision operationally relies on that one | Anchored |
| `conflictsWith` | the two stand in tension | Anchored — **and so is this record** |

**`amends` and `supersedes` both need an `Accepted` record at each end**, because both act on content,
and only an Accepted record has content in force. Amendment is **opt-out** (everything in the
amended record stays valid but the deltas), while supersession is **opt-in** (retiring everything). A
`Rejected` record can carry neither, and can't be named by either. `supersedes` additionally targets
a record only once – a `Superseded` target has nothing left.

`dependsOn` and `conflictsWith` assert a relation between records rather than operating on their
contents, so they hold against any resident, including Rejected records. This may feel contradictory.
An Accepted record can `dependsOn` a Rejected record: if that record was accepted, this record could
not be accepted, so this record depends on that record's rejection. The other is even more strange,
but equally valid: an Accepted record can also `conflictsWith` a Rejected record: the decision we
just accepted conflicts with that rejection, e.g. accepting that our company opens a store in Tokyo
after we rejected accessing the Asian market.

For `settlesOpenQuestion`, the target is the **older open-question holder**, not the closing record.
If the holder is still staged it has no number to target — update the holder in place instead.
`amendedBy`, `supersededBy` and `openQuestions[].settledIn` are read-only; the server mints them at
admission.

## Matter shape, not matter status

A matter has no status field. There is no chain to walk and no value to set — the read is the
topology:

| What you see | What it means |
| --- | --- |
| No reference points at it | Nobody has picked it up (`discover_matters(untaken: true)`) |
| Reference → analysis, `producesDecision: []` | Investigated, nothing warranted |
| Reference → analysis with produced records | Decision work happened |
| Reference `created` later than the analysis's `concludedAt` | Attached after the reasoning closed; the rationale does not account for it |

Neither direction is bounded: an analysis may take up many inputs, and one input may be taken up by
many analyses. Inputs need not be matters: an admitted World Fact or Observed Outcome is taken up by
its own id, and the table above reads only for matters — almanac entries have no inbox and no
"unused" state. Re-analysis does **not** need a new matter — open a second analysis over the same
input.

An analysis concludes in the call that creates it. `reopen_analysis` clears that conclusion, and is
refused once any record it produced carries `decidedAt`.

## Posture summary

| Posture | Path |
| --- | --- |
| Informal (default) | Deliberating → t₀ |
| Formal | Deliberating → Proposed → t₀ |

A fully populated fish — verdict, outcomes, open questions — may sit at `Deliberating` indefinitely.
Only the head Claim is required at that status.
