# MCP boundaries

**Rule:** if no tool exists for something below, do not simulate it with local files, prose-only
records, or invented calls. Tell the user it is not available yet.

## Available

| Capability | Tools |
| --- | --- |
| Auth and health | `ping`, `get_guidance`, `whoami` |
| Ledger discovery and creation | `get_MDLs`, `get_MDL`, `create_MDL` |
| Ledger intent | `set_ledger_intent` (read it back with `get_MDL`) |
| Matter intake | `register_matter` |
| Matter reads | `get_matter` (one, whole), `discover_matters` (the ledger's matters as prose, each with its summary, subjects and who took it up) |
| Analysis, Path A and B | `create_analysis` |
| Analysis reads | `get_analysis` (point read) |
| Analysis inputs (matter, world fact or observed outcome), after conclusion | `attach_analysis_reference`, `retract_analysis_reference` |
| Reopen a concluded analysis | `reopen_analysis` (refused once a produced record is committed) |
| Expert mint | `create_MDR` |
| Record reads | `get_MDR` (one, whole), `discover_MDRs` (the ledger's records as prose, each with its summary, status, handle and decision date) |
| Staged patch, including `hasContext` | `update_MDR` |
| Lifecycle transition | `transition_MDR_status` |
| Uncommit a retractable record | `uncommit_MDR` (admin/owner) |
| Declare an external citation | `anchor_MDR` (member; permanent, and it makes an Uncommit impossible) |
| World almanac | `admit_world_fact`, `get_world_fact` (one, whole), `discover_world_facts` (the almanac as prose, each fact with its summary and what it corrects) |
| Wake | `record_observed_outcome`, `get_observed_outcome` (one, whole), `discover_observed_outcomes` (a ledger's wake as prose, each with its summary and recorded result) |
| Learning delta, one record | `get_MDR_learning_delta` |
| Working notes | `create_scratchpad`, `get_scratchpad`, `replace_scratchpad`, `delete_scratchpad`, `list_scratchpads` |
| Feedback to Memonos, the team behind Memolok | `submit_feedback`, `get_feedback`, `update_feedback` — see the `send-feedback` skill |

### Graph edges, staged records only

`update_MDR` may patch `amends`, `supersedes`, `dependsOn`, `conflictsWith` and
`settlesOpenQuestion`. Targets must be admitted `mdrNumber`s. On a resident the field is refused —
including `[]`, since presence is what is refused. Reciprocals (`amendedBy`, `supersededBy`) publish
at admission and are never writable.

`supersedes` may only target `Accepted` residents, once each; admission flips them to `Superseded`.
`amends` requires an `Accepted` record at each end too, for the same reason: amendment is opt-out
over the anchor's surviving content, and a `Rejected` or `Superseded` record has none.

`dependsOn` and `conflictsWith` carry no status gate, including against a `Rejected` record. They
relate the records, not their contents, and a decline is a real decision even though what it declined
is not in force — so a later decision can rest on it, or collide with it.

**Every one of these Anchors its target at admission**, so amending a record ends its uncommit path.
`conflictsWith` is symmetric and Anchors both endpoints, the author's own record included.

### Almanac context, staged records only

`update_MDR` may set `hasContext` to an ordered list of World Fact or prior Observed Outcome ids from
the same ledger. The list freezes at admission. A record may not cite its own wake.

### Retractable

| Value | Meaning |
| --- | --- |
| `null` | Staged — patch freely; uncommit not applicable |
| `true` | Committed and not anchored — **still not patchable**; uncommit first |
| `false` | Anchored — amend one clause, or supersede the whole record |

This is operational metadata computed at read time. It is never exported.

## Not available — do not invent

### Closing a matter as refused, blocked, or overtaken

Analysis reaches two endings: it produces records, or it produces none. Recording that a matter was
**refused** (valid, and we commit to not acting), **externally blocked**, or **overtaken by events**
is modelled with no tool behind it.

Until they land, record refusal or blockage rationale in the Verdict prose at t₀, and do not
substitute a Path B dismissal — "no decision warranted" says the investigation found nothing to
decide, which is weaker and different from refusing a problem you agree is real.

**Do not invent fields for any of these**, but they are absent in two different ways, which matters if
you go looking for them:

| Field | Why it is unreachable |
| --- | --- |
| `resolvesMatter` | Exists in storage; **forbidden by name** on the MCP surface |
| `rendersMoot` | Exists in storage; not patchable and not a mint parameter |
| `declinesMatter` | **Not in the data model at all** |
| `blocksResolutionOf` | **Not in the data model at all** |

Re-analysis, by contrast, **is** available: open a second analysis over the same input, or attach it
to an existing one. Nothing limits an input to one analysis.

### Feedback reports are not ledger entities

`submit_feedback` sends the feedback to Memonos, not to the user's own ledger. A report has no `mdlGuid` tenancy,
carries no Claim, cannot be cited, and never becomes a Matter. Mental model: write-only – there is
**no list and no search**: a report is reachable only by an id its submitter still holds, and it may be
deleted server-side during triage, so `get_feedback` legitimately returns not-found for something that existed.

Do not simulate the missing enumeration by keeping a local index of what you sent.

### Portfolio intelligence

| Capability |
| --- |
| Outcome-evaluation reminders |
| Polluted-premise and decision-decay queries |
| Stale-constraint review |
| Impact analysis |
| Conflict and dependency surfacing during deliberation |
| Open-question settlement registry queries |
| Automatic supersession on a `Violated` result |

`get_MDR_learning_delta` covers one record at a time. For anything portfolio-wide, read with
`discover_MDRs` and reason in conversation — do not claim the ledger surfaced it.

### Ledger administration

| Capability |
| --- |
| Managing membership |
| Decision-record-type taxonomy |
| Concern hierarchy |
| Setting `typeId` or `concernIds` |

`create_MDL` is open to any authenticated user, who becomes owner. Everything else about membership is
provisioned by a Memolok administrator.

### Project bridges

Linking records to repository files, and freezing cited artifacts at t₀, are not on the tool surface.
`anchor_MDR` is not that link: it records that an external citation exists and what kind of place it
lives in, never where, so nothing resolves from a record back to a file.

### Deletion

**`delete_scratchpad` is the only delete tool, for the only deletable entity.** Everything else is
immutable on admission (`Matter`, `WorldFact` substance) or sealed at t₀ (`DecisionRecord`), and
there is no delete for any of them — a matter registered by mistake is left unreferenced or closed
through Path B, a wrong
fact gets a correcting successor, a regretted record is uncommitted or superseded.

Do not generalise from the scratchpad tool. It exists only because a note is the one thing nothing
else can depend on.

### Scratchpad references

Nothing may reference a scratchpad, permanently and by design. Passing a `sp_…` id to `hasContext`,
`correctsFact`, `motivatedBy`, `tests.outcomeId` or `evidence` is refused by name — as is an `fb_…`
feedback report id, which is not a ledger entity at all.

There is also no link the other way — no "promoted from", no provenance edge. Content mined out of a
note is authored fresh with no trail back, and the user should be told that once.

## Naming

| Use | Not |
| --- | --- |
| `mdrHandle` | a database id as an address |
| `mdrNumber` (`null` while staged) | `adrNumber` |
| `matterId` | matter "number" |
| `worldFactId`, `observedOutcomeId` | invented ledger numbers |
| `scratchpadId` (`sp_` + 26) | a bare id, or any other entity's prefix — every kind carries its own, so a category error is refused by name |
| `mdlGuid` | `adlGuid` |

Tools take `mdrHandle`; as a convenience, `get_MDR` also accepts `mdrNumber` so a record someone cites by number can be read directly. Use the handle whenever you hold one.

In conversation, cite `mdrNumber` as `MDR-{n}` once admitted; while staged, name the record
`MDRh{handle}` or paraphrase its head Claim. Never a bare handle.
