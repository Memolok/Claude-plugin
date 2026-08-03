# MCP tools catalog

Every tool except `ping` and `get_guidance` requires authentication. All ledger tools are scoped by
`mdlGuid`.

## Identity keys

| Field | When | Role |
| --- | --- | --- |
| `mdrHandle` | Every mint | Addressing key for record tools, including after admission |
| `mdrNumber` | Admission only | Ledger citation; `null` while staged. Accepted by `get_MDR` alone |
| `retractable` | Computed at read | `null` staged; `true` uncommit-eligible; `false` anchored |

Never invent either value. Never address a record tool by a raw database id.

**`mdrHandle` is the standard path whenever you have one** — it is what every record tool takes. `get_MDR` also accepts `mdrNumber`, for the one journey
where someone cites "MDR-7" and you hold no handle: read it directly rather than scanning
`list_MDRs`. Two things bound that exception:

- **It is a read.** A number is not a durable address until anchoring — an uncommit releases it and the next
  admission takes it. A read that lands on the wrong record announces itself, because the response
  states both identifiers; a write would not, so no write tool accepts a number.
- **The response carries `mdrHandle`.** Once you have read the record, use its handle for everything
  else in that session — patches, transitions, wakes, an uncommit.

**Read `retractable` before suggesting an uncommit.**

All prose parameters use `{ markdown, lang? }` — see `prose-and-raci.md` for which are wrapped in a
`description` key.

## Read tools

### `ping` / `get_guidance`

No parameters. `ping` returns `"pong"`; `get_guidance` returns the server's instruction text.

### `whoami`

No parameters. Returns `{ userId, givenName, familyName, mbox }`.

### `get_MDLs`

No parameters. Returns `{ mdls: [{ mdlGuid, title, role }] }`. A chooser across ledgers — it does not
carry the Ledger Intent.

### `get_MDL`

| Param | Type | Required |
| --- | --- | --- |
| `mdlGuid` | string | yes |

Returns `{ mdlGuid, title, role, ledgerIntent }`. `ledgerIntent` is `null` when the ledger has never
stated a purpose — that is normal, not an error.

**One ledger's own metadata, not its contents.** For what is inside, use `list_MDRs`, `list_matters`,
`list_world_facts`, `list_observed_outcomes`.

**Not `get_MDR`.** One letter apart, and completely different: `get_MDL` takes only `mdlGuid` and
returns ledger metadata; `get_MDR` takes `mdlGuid` + a record key and returns a decision record. Reading
one while meaning the other produces a confident answer to the wrong question.

### `get_MDR`

| Param | Type | Required |
| --- | --- | --- |
| `mdlGuid` | string | yes |
| `mdrHandle` | int | exactly one of the two |
| `mdrNumber` | int | exactly one of the two |

Returns the full record — fish body, `mdrHandle`, `mdrNumber`, `retractable`, and any graph edges,
plus `promptedBy` and `analysisId` (both `null` on the expert path).

Pass the handle when you have one. `mdrNumber` is here for the case where a person cites a number
and you would otherwise scan `list_MDRs` to find its handle — one call instead of a ledger-wide read.
**Exactly one of them MUST be passed.** A number can have been released by an uncommit and taken by a
later record, so check the record you get back is the one meant; the handle it returns is what you
use for every other tool.

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

Returns `{ id, mdlGuid, status, description, analysisId, producesDecision }`.
Error: `Matter not found.`

`analysisId` is `null` until analyzed. `producesDecision` holds `{ mdrHandle, mdrNumber }` per record
minted, `[]` when dismissed. No rationale here — that is `get_analysis`.

### `list_matters`

| Param | Type | Required |
| --- | --- | --- |
| `mdlGuid` | string | yes |
| `status` | string | no |

Returns `{ matters: [...] }` in registration order, rows shaped like `get_matter`. Unlike
`list_MDRs`, an unknown status raises:

```
Unknown matter status {x}. Use one of: ...
```

`list_matters(status="MatterReceived")` is the unprocessed-bait inbox — every matter parked
but never analyzed.

### `get_analysis`

| Param | Type | Required |
| --- | --- | --- |
| `mdlGuid` | string | yes |
| `analysisId` | string | yes |

Returns `{ id, mdlGuid, analyzes, producesDecision, analysisRationale, performedBy }`.
Error: `Analysis not found.`

Point read; there is no `list_analyses`. Reach it by `analysisId` from `get_matter` or `get_MDR`.

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

| Param | Type | Required |
| --- | --- | --- |
| `title` | string | yes |
| `ledgerIntent` | `{ description: { markdown, lang? } }` | no |

Caller becomes `owner`. Returns the same shape as `get_MDL`.

Pass `ledgerIntent` when the user has agreed a purpose — one call, not create-then-set. Note the nested
shape: `{ "description": { "markdown": "…" } }`, **not** a flat `{ markdown }`.

### `set_ledger_intent`

| Param | Type | Required |
| --- | --- | --- |
| `mdlGuid` | string | yes |
| `ledgerIntent` | `{ description: { markdown, lang? } }` | yes |

**Full replacement, not an append.** There is no history and no version. Read the current statement
with `get_MDL` first, then send the complete new one.

Requires `member` or above. Returns the same shape as `get_MDL`.

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

Returns `{ analysis, mdr? }`. Path A mints the record at `New` with `promptedBy` set; Path B omits
`mdr` entirely (absent, not null).

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
