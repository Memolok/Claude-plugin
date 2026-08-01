# MCP tools catalog

Every tool except `ping` and `get_guidance` requires authentication. All ledger tools are scoped by
`mdlGuid`.

## Identity keys

| Field | When | Role |
| --- | --- | --- |
| `mdrHandle` | Every mint | Sole addressing key for record tools, including after admission |
| `mdrNumber` | Admission only | Ledger citation; `null` while staged |
| `retractable` | Computed at read | `null` staged; `true` uncommit-eligible; `false` anchored |

Never invent either value. Never address a record tool by `mdrNumber` or by a raw database id.
**Read `retractable` before suggesting an uncommit.**

All prose parameters use `{ markdown, lang? }` — see `prose-and-raci.md` for which are wrapped in a
`description` key.

## Read tools

### `ping` / `get_guidance`

No parameters. `ping` returns `"pong"`; `get_guidance` returns the server's instruction text.

### `whoami`

No parameters. Returns `{ userId, givenName, familyName, mbox }`.

### `get_MDLs`

No parameters. Returns `{ mdls: [{ mdlGuid, title, role }] }`.

### `get_MDR`

| Param | Type | Required |
| --- | --- | --- |
| `mdlGuid` | string | yes |
| `mdrHandle` | int | yes |

Returns the full record — fish body, `mdrNumber`, `retractable`, and any graph edges.

### `list_MDRs`

| Param | Type | Required |
| --- | --- | --- |
| `mdlGuid` | string | yes |
| `status` | string | no |

Returns `{ mdrs: [{ mdrHandle, mdlGuid, mdrNumber, status, retractable, headClaimMarkdown }] }`.

**The `status` filter is not validated.** A typo or a non-status value returns an empty list rather
than an error, which reads as "no records". Send exact status names.

### `get_matter`

| Param | Type | Required |
| --- | --- | --- |
| `mdlGuid` | string | yes |
| `matterId` | string | yes |

Returns `{ id, mdlGuid, status, description }`. Error: `Matter not found.`

### `list_matters`

| Param | Type | Required |
| --- | --- | --- |
| `mdlGuid` | string | yes |
| `status` | string | no |

Returns `{ matters: [...] }` in registration order. Unlike `list_MDRs`, an unknown status raises:

```
Unknown matter status {x}. Use one of: ...
```

`list_matters(status="MatterReceived")` is the unprocessed-bait inbox — every matter parked
but never analyzed.

### `get_world_fact` / `list_world_facts`

`get_world_fact` takes `mdlGuid` + `worldFactId`; `list_world_facts` takes `mdlGuid` alone. Return
`WorldFact` payloads carrying `worldFactId`, `manifests`, and optional `correctsFact`.

### `get_observed_outcome` / `list_observed_outcomes`

`get_observed_outcome` takes `mdlGuid` + `observedOutcomeId`. `list_observed_outcomes` takes
`mdlGuid` and an optional `mdrHandle` filter.

### `get_MDR_learning_delta`

| Param | Type | Required |
| --- | --- | --- |
| `mdlGuid` | string | yes |
| `mdrHandle` | int | yes |

Returns each expected outcome with the wakes that test it, plus unmatched Emergent and Deducible
wakes. Coexisting assessments are all returned — there is no "current result" field. Ledger residents
only.

## Write tools

### `create_MDL`

`title` (string). Caller becomes `owner`.

### `register_matter`

| Param | Type | Required |
| --- | --- | --- |
| `mdlGuid` | string | yes |
| `description` | `{ markdown, lang? }` | yes |

Returns the matter with `status: MatterReceived`. Record the raiser's words **verbatim** —
do not sharpen here.

### `create_analysis`

| Param | Type | Required |
| --- | --- | --- |
| `mdlGuid` | string | yes |
| `analyzes` | string (matter id) | yes |
| `analysisRationale` | `{ markdown, lang? }` | yes |
| `producesDecision` | bool | no (default `true`) |
| `claimDescription` | `{ markdown, lang? }` | Path A only |

Returns `{ analysis, mdr? }`. Path A mints the record at `New` with `promptedBy` set.

### `create_MDR`

| Param | Type | Required |
| --- | --- | --- |
| `mdlGuid` | string | yes |
| `claimDescription` | `{ markdown, lang? }` | yes |
| `status` | string | yes — canonical values only |
| `alternatives` | `[{ id?, label?, description, satisfies? }]` | no |
| `deliberationFacts` | `[{ onAlternative?, description }]` | no |
| `expectedOutcomes` | `[{ id?, description }]` | no |
| `openQuestions` | `[{ id?, description }]` | no |
| `verdict` | `{ description }` | no |
| `chosenAlternative` | string | no |
| `authoredBy`, `decidedBy` | string | no |
| `consulted`, `informed` | string[] | no |

Expert path only — there is no `promptedBy` parameter. Returns the record with a minted `mdrHandle`;
`mdrNumber` only if created at `Accepted` or `Rejected`.

### `update_MDR`

| Param | Type | Required |
| --- | --- | --- |
| `mdlGuid` | string | yes |
| `mdrHandle` | int | yes |
| `patch` | object | yes |

Patch keys: `hasNeed`, `hasContext`, `alternatives`, `deliberationFacts`, `expectedOutcomes`,
`openQuestions`, `verdict`, `chosenAlternative`, the four RACI fields, `supersedes`,
`settlesOpenQuestion`. Patches may be incremental — send only what changed.

### `transition_MDR_status`

`mdlGuid`, `mdrHandle`, `status`. Admission to `Accepted` or `Rejected` sets `decidedAt`, assigns
`mdrNumber`, and mints reciprocals.

### `uncommit_MDR`

`mdlGuid`, `mdrHandle`, optional `reason`. Requires `admin` or `owner`, status `Accepted` or
`Rejected`, and `retractable: true`. Demotes to staged, clearing `mdrNumber` and `decidedAt`.

### `admit_world_fact`

`mdlGuid`, `claimDescription`, optional `correctsFact`. Use `correctsFact` only for a fact that was
wrong when admitted — never for ordinary world drift.

### `record_observed_outcome`

| Param | Type | Required |
| --- | --- | --- |
| `mdlGuid` | string | yes |
| `mdrHandle` | int | yes — must be a ledger resident |
| `claimDescription` | `{ markdown, lang? }` | yes |
| `discoveryType` | `Expected` \| `Emergent` \| `Deducible` | yes |
| `tests` | `{ outcomeId }` or an id string | Required when `Expected` |
| `testResult` | `Satisfied` \| `Violated` \| `Inconclusive` | Required when `Expected` |
| `observedBy`, `correctsFact` | string | no |

There is no `observedAt` parameter — the server stamps the current time, so a wake cannot be
backdated. Recording one typically Anchors the source record.

## Common errors

| Message | Cause |
| --- | --- |
| `Memolok Decision Ledger not found.` | Bad `mdlGuid`, or a read by a non-member (deliberate — no existence leak) |
| `You are not a member of this Memolok Decision Ledger.` | Write without membership |
| `Memolok Decision Record not found.` | Missing record, or non-member read |
| `The {field} reference belongs to a different Memolok Decision Ledger.` | Cross-ledger reference |
| `Use transition_MDR_status to change Memolok Decision Record status.` | `status` in a patch |
| `Cannot change the {label} on a ledger-resident Memolok Decision Record ({status}).` | Tier-1 patch after t₀ |
| `Inter-record graph links may only be authored on staged Memolok Decision Records.` | Graph patch on a resident |
| `No updatable fields were provided.` | Empty or no-op patch |
| `Cannot transition a Memolok Decision Record from {from} to {to}.` | Illegal transition |
| `supersedes may only target Accepted residents (MDR-{n} is {status}).` | Bad supersession target |
| `This Memolok Decision Record is Anchored and cannot be Uncommitted.` | Uncommit on an anchored record |
| `Only Accepted or Rejected Memolok Decision Records may be Uncommitted.` | Wrong status for uncommit |
| `Observed Outcomes can only realizeFrom a ledger-resident Memolok Decision Record...` | Wake on a staged record |
| `discoveryType Expected requires tests referencing an expectedOutcome.` | Missing `tests` |
| `Analysis can only start from a Matter in MatterReceived status.` | Matter already analyzed |
| `claimDescription is required when analysis produces a Memolok Decision Record.` | Path A without a claim |
| `A Memolok Decision Record cannot cite its own Observed Outcome in hasContext...` | DTP violation |
