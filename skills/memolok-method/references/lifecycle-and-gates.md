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
| Rejected | Verdict present; `decidedAt`; `supersedes` empty |

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
| Staged only | All tier-1 fish fields, plus `supersedes` and `settlesOpenQuestion` |
| Any status | `authoredBy`, `decidedBy`, `consulted`, `informed` |
| Never | `status` (use the transition tool), `openQuestions[].settledIn`, `mdrHandle`, `mdrNumber`, `supersededBy` |

`hasContext` is `update_MDR`-only — it is not a `create_MDR` parameter — and takes an ordered list of
World Fact or prior Observed Outcome ids.

## Post-admission correction

| `retractable` | Action |
| --- | --- |
| `null` | Staged — just patch it |
| `true` | Uncommit (admin/owner), edit while staged, re-admit under a fresh number |
| `false` | Anchored — mint a successor carrying `supersedes` |

A record becomes Anchored when another record's `amends`/`supersedes`/`dependsOn`/`enables`/
`conflictsWith` cites its number, when one of its open questions has been settled, or when any
Observed Outcome was realized from it. Recording a wake therefore usually Anchors the source.

## Graph edges on staged records

`supersedes` and `settlesOpenQuestion` target admitted `mdrNumber`s. For `settlesOpenQuestion`, the
target is the **older open-question holder**, not the closing record. If the holder is still staged it
has no number to target — update the holder in place instead. `supersededBy` and
`openQuestions[].settledIn` are read-only.

## Matter shape, not matter status

A matter has no status field. There is no chain to walk and no value to set — the read is the
topology:

| What you see | What it means |
| --- | --- |
| No reference points at it | Nobody has picked it up (`list_matters(untaken: true)`) |
| Reference → analysis, `producesDecision: []` | Investigated, nothing warranted |
| Reference → analysis with produced records | Decision work happened |
| Reference `created` later than the analysis's `concludedAt` | Attached after the reasoning closed; the rationale does not account for it |

Neither direction is bounded: an analysis may take up many matters, and a matter may be taken up by
many analyses. Re-analysis does **not** need a new matter — open a second analysis over the same one.

An analysis concludes in the call that creates it. `reopen_analysis` clears that conclusion, and is
refused once any record it produced carries `decidedAt`.

## Posture summary

| Posture | Path |
| --- | --- |
| Informal (default) | Deliberating → t₀ |
| Formal | Deliberating → Proposed → t₀ |

A fully populated fish — verdict, outcomes, open questions — may sit at `Deliberating` indefinitely.
Only the head Claim is required at that status.
