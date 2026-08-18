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
plus `analysisId` (`null` when no analysis produced it).

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

Returns `{ id, mdlGuid, description, takenUpBy }`. Error: `Matter not found.`

`takenUpBy` is one entry per analysis that took this matter up — `{ referenceId, analysisId, created,
concludedAt, late, producesDecision }` — and `[]` when nobody has. `producesDecision` holds
`{ mdrHandle, mdrNumber }` per record **that analysis** minted; those records belong to the reasoning,
not to this matter, and nothing says any of them answers it. No rationale here — that is
`get_analysis`.

### `list_matters`

| Param | Type | Required |
| --- | --- | --- |
| `mdlGuid` | string | yes |
| `untaken` | bool | no |

Returns `{ matters: [...] }` in registration order, rows shaped like `get_matter`.

`list_matters(untaken: true)` is the unprocessed-bait inbox — every matter no analysis references.
`untaken: false` gives the complement. There is no status filter, because a matter has no status.

### `get_analysis`

| Param | Type | Required |
| --- | --- | --- |
| `mdlGuid` | string | yes |
| `analysisId` | string | yes |

Returns `{ id, mdlGuid, references, producesDecision, analysisRationale, performedBy, concludedAt }`.
Error: `Analysis not found.`

Each entry in `references` is `{ referenceId, motivatedBy, created, late }`. `late` true means the
input was attached after `concludedAt`, so the rationale does not account for it; **`null` means
unanswerable** — a backfilled reference with no date, or an analysis with no conclusion — and never
"not late".

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

Returns `{ id, mdlGuid, description, takenUpBy: [] }`. Record the raiser's words **verbatim** —
do not sharpen here.

### `create_analysis`

| Param | Type | Required |
| --- | --- | --- |
| `mdlGuid` | string | yes |
| `motivatedBy` | string[] (matter ids) | yes |
| `analysisRationale` | `{ markdown, lang? }` | yes |
| `producesDecision` | bool | no (default `true`) |
| `claimDescription` | `{ markdown, lang? }` | Path A only |

`motivatedBy` is a list: pass **every** matter this reasoning took up. Empty raises. The analysis
concludes in this call, stamping `concludedAt` with the same instant it dates the references, so
they read as on time.

Returns `{ analysis, mdr? }`. Path A mints the record at `New`; Path B omits `mdr` entirely (absent,
not null).

### `attach_analysis_reference`

| Param | Type | Required |
| --- | --- | --- |
| `mdlGuid` | string | yes |
| `analysisId` | string | yes |
| `motivatedBy` | string (matter id) | yes |

Takes up a matter the analysis did not originally reference. **Allowed after the analysis
concluded** — that is what it is for. The reference is dated now, so it reads as `late: true` and the
sealed rationale is untouched. Returns the reference.

At most one reference per (matter, analysis) pair; a second raises rather than replacing the first,
because two attachment times make lateness unanswerable.

### `retract_analysis_reference`

| Param | Type | Required |
| --- | --- | --- |
| `mdlGuid` | string | yes |
| `referenceId` | string | yes |

Withdraws a reference attached in error. Ungated, including against a committed record: an analysis
must describe reasoning that occurred. Returns `{ retracted }`.

### `reopen_analysis`

| Param | Type | Required |
| --- | --- | --- |
| `mdlGuid` | string | yes |
| `analysisId` | string | yes |

Clears `concludedAt`. Refused once any produced record carries `decidedAt` — uncommit that record
first. Use it when the reasoning genuinely was not finished; for scope that arrived afterwards, a
late reference is the honest record, not a reopen.

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

Expert path only — there is no matter parameter. Returns the record with a minted `mdrHandle`;
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

## Scratchpad tools

Disposable working notes. See the **`manage-notes`** skill for the journeys.

**Identifiers are different here, on purpose.** A `scratchpadId` is `sp_` followed by 24 hex
characters — `sp_6a761688013eff0dc9e8dee1` — not the bare id every other entity uses. The prefix is
what lets every reference field refuse one *by name*: pass a `scratchpadId` to `hasContext`,
`correctsFact`, `motivatedBy`, `tests.outcomeId` or `evidence` and the call fails telling you it is a
scratchpad, rather than reading as a mistyped World Fact id. Do not treat the two as interchangeable
in either direction.

### `create_scratchpad`

| Param | Type | Required |
| --- | --- | --- |
| `mdlGuid` | string | yes |
| `description` | `{ markdown, lang? }` | yes |

Flat prose, like `register_matter` — **not** the nested `{ description: { markdown } }` shape that
`ledgerIntent` and `verdict` take. The body is the entry; there is no second level.

Returns `{ scratchpadId, mdlGuid, description, createdAt, createdBy, modifiedAt, contributors }`.

### `get_scratchpad`

`mdlGuid` + `scratchpadId`. The only tool that returns a full body.

### `replace_scratchpad`

`mdlGuid`, `scratchpadId`, `description`. **Full replacement, not an append.** There is no partial
update: read with `get_scratchpad`, compose the whole new body, send that. Adds the caller to
`contributors` and advances `modifiedAt`.

### `delete_scratchpad`

`mdlGuid` + `scratchpadId`. **The only delete tool in Memolok.** Immediate and final — no recovery
window. Returns `{ scratchpadId, mdlGuid, deleted: true }`.

### `list_scratchpads`

| Param | Type | Required |
| --- | --- | --- |
| `mdlGuid` | string | yes |
| `limit` | int | no (default 25, clamped to 100) |
| `offset` | int | no (default 0) |

Returns `{ scratchpads: [...], total, limit, offset }`, most recently touched first. Rows are
previews — `excerpt`, `truncated`, `length` — and **never carry a body**.

### `search_scratchpads`

| Param | Type | Required |
| --- | --- | --- |
| `mdlGuid` | string | yes |
| `query` | string | yes |
| `limit` | int | no (default 25) |

Returns `{ scratchpads: [...], total, limit, matchMode }`. Rows add `matchExcerpt` — a window around
the hit — and `score` when the text pass ranked them.

`matchMode` is `text` (stemmed, relevance-ranked) or `regex` (the text pass found nothing, so a
substring pass answered). An empty result means no note matched; it is not a cue to list everything.

**Attribution.** `createdBy` is `{ userId, name? }`; `contributors` is a list of the same. The
contributor set is cumulative and never shrinks, so it does **not** say who edited most recently —
pair it with `modifiedAt`. Any of the four attribution facts may be absent, which is normal.

## Feedback tools

Feedback goes to **Memolok**, not to the user's ledger. No `mdlGuid` tenancy, no Claim, nothing
citable. Journey and the standard a report is held to: the **`send-feedback`** skill.

### `submit_feedback`

| Param | Type | Required |
| --- | --- | --- |
| `reports` | array of report objects | yes |

**A batch always** — a single report is a batch of one, and one call means one preview-and-confirm.
Up to 25 per call. Validation is all-or-nothing: a malformed third report leaves the first two
unwritten, and the error names the offender by index (`reports[2].evidence[0]: …`).

Each item in `reports`:

| Field | Type | Required |
| --- | --- | --- |
| `title` | string | yes |
| `kind` | `bug` \| `suggestion` | yes |
| `report` | `{ markdown, lang? }` | yes |
| `userVerbatim` | `{ markdown, lang? }` | no |
| `mdlGuid` | string | no |
| `artifacts` | array of `{ kind, name, version? }` | no |
| `evidence` | array of evidence items | no |
| `expectation` | `{ markdown, lang? }` | no |

`artifacts[].kind` is one of `skill`, `mcp_tool`, `mcp_server`, `plugin`, `model`. Artifacts name what
was **in play**, not what is at fault — a skill contradicting a tool is two artifacts.

An **evidence item** is either a call record or a citation, and must carry a `response` or an
`excerpt`:

| Field | Type | For |
| --- | --- | --- |
| `occurredAt` | ISO-8601 string | call record |
| `toolName` | string | call record |
| `request` / `response` | `{ markdown, lang? }` | call record |
| `requestId` | string | call record (`X-Memolok-Request-Id`) |
| `source` / `excerpt` | string / `{ markdown, lang? }` | citation |

There is no `outcome` field and no report-class field. A call that returned 200 and did the wrong
thing is the case worth catching, and a success/error flag would file it under "success".

**Never send** `submittedBy`, `submittedAt`, `serverVersion` or `modelVersion` — the server records
those and refuses a caller that sets them. `artifacts[].version` is yours to supply for what you can
actually read (the plugin, a skill); the server stamps its own build.

Returns `{ reports: [{ feedbackId, title, kind, submittedAt }, …] }` in submission order — a digest,
not the bodies. Keep the ids.

### `get_feedback`

| Param | Type | Required |
| --- | --- | --- |
| `feedbackId` | string (`fb_<24 hex>`) | yes |

Your own reports only. A stranger's id returns *not found* rather than a permission error, and so
does a report deleted during triage.

### `update_feedback`

| Param | Type | Required |
| --- | --- | --- |
| `feedbackId` | string (`fb_<24 hex>`) | yes |
| `patch` | object | yes |

Owner-only, no time limit. Patchable: `title`, `kind`, `report`, `userVerbatim`, `mdlGuid`,
`artifacts`, `evidence`, `expectation`. **Arrays replace, they do not merge** — send the full list.

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
| `An analysis must take up at least one input; motivatedBy is empty.` | Empty `motivatedBy` |
| `That input is already referenced by this analysis.` | Duplicate attach — the existing reference stands |
| `{field} is a scratchpad id.` | A `sp_…` value passed to a reference field. Nothing may cite a note — admit a World Fact instead |
| `{field} must be a scratchpad id of the form 'sp_<24 hex characters>'.` | A bare id passed where a `scratchpadId` was expected |
| `Scratchpad not found.` | Missing note, wrong ledger, or already deleted |
| `A scratchpad body may be at most 65536 bytes;…` | Paste too large — split it, or keep a pointer to the source |
| `claimDescription is required when analysis produces a Memolok Decision Record.` | Path A without a claim |
| `A Memolok Decision Record cannot cite its own Observed Outcome in hasContext...` | DTP violation |
| `{field} must be a feedback report id of the form 'fb_<24 hex characters>'.` | A ledger id passed to a feedback tool — different kinds of address |
| `Feedback report not found.` | Not yours, wrong id, or deleted during triage |
| `reports[n] may not set serverVersion, …` | The server records the build; a caller cannot claim one |
| `reports[n].evidence[m] must carry either a response … or an excerpt` | An evidence item that asserts nothing |
