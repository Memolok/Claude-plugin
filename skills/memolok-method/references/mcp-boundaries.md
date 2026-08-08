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
| Matter reads | `get_matter`, `list_matters` |
| Analysis, Path A and B | `create_analysis` |
| Analysis reads | `get_analysis` (point read) |
| Expert mint | `create_MDR` |
| Record reads | `get_MDR`, `list_MDRs` |
| Staged patch, including `hasContext` | `update_MDR` |
| Lifecycle transition | `transition_MDR_status` |
| Uncommit a retractable record | `uncommit_MDR` (admin/owner) |
| World almanac | `admit_world_fact`, `get_world_fact`, `list_world_facts` |
| Wake | `record_observed_outcome`, `get_observed_outcome`, `list_observed_outcomes` |
| Learning delta, one record | `get_MDR_learning_delta` |
| Working notes | `create_scratchpad`, `get_scratchpad`, `replace_scratchpad`, `delete_scratchpad`, `list_scratchpads`, `search_scratchpads` |

### Graph edges, staged records only

`update_MDR` may patch `supersedes` and `settlesOpenQuestion`. Targets must be admitted `mdrNumber`s.
`supersedes` may only target `Accepted` residents, once each; admission flips them to `Superseded`.
Reciprocals publish at admission.

### Almanac context, staged records only

`update_MDR` may set `hasContext` to an ordered list of World Fact or prior Observed Outcome ids from
the same ledger. The list freezes at admission. A record may not cite its own wake.

### Retractable

| Value | Meaning |
| --- | --- |
| `null` | Staged — patch freely; uncommit not applicable |
| `true` | Committed and not anchored — **still not patchable**; uncommit first |
| `false` | Anchored — mint a successor with `supersedes` |

This is operational metadata computed at read time. It is never exported.

## Not available — do not invent

### Matter dispositions beyond Path A and B

Only `MatterAnalyzed` (Path A) and `MatterDismissed` (Path B) are reachable. `MatterResolved`,
`MatterUnresolved`, `MatterDeclined`, `MatterBlocked`, and `MatterMoot` exist in the model
with no way to reach them.

Until they land, record refusal or blockage rationale in the Verdict prose at t₀, and use Path B for
honest non-decisions. Do not invent fields for `resolvesMatter`, `declinesMatter`,
`blocksResolutionOf`, or `rendersMoot`.

There is also no way to **re-analyze** a matter. One analysis per matter; a second look needs a
new matter.

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
`list_MDRs` and reason in conversation — do not claim the ledger surfaced it.

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

### Deletion

**`delete_scratchpad` is the only delete tool, for the only deletable entity.** Everything else is
immutable on admission (`Matter`, `WorldFact` substance) or sealed at t₀ (`DecisionRecord`), and
there is no delete for any of them — a matter registered by mistake is closed through Path B, a wrong
fact gets a correcting successor, a regretted record is uncommitted or superseded.

Do not generalise from the scratchpad tool. It exists only because a note is the one thing nothing
else can depend on.

### Scratchpad references

Nothing may reference a scratchpad, permanently and by design. Passing a `sp_…` id to `hasContext`,
`correctsFact`, `analyzes`, `tests.outcomeId` or `evidence` is refused by name.

There is also no link the other way — no "promoted from", no provenance edge. Content mined out of a
note is authored fresh with no trail back, and the user should be told that once.

## Naming

| Use | Not |
| --- | --- |
| `mdrHandle` | a database id as an address |
| `mdrNumber` (`null` while staged) | `adrNumber` |
| `matterId` | matter "number" |
| `worldFactId`, `observedOutcomeId` | invented ledger numbers |
| `scratchpadId` (`sp_`-prefixed) | a bare id — scratchpads alone carry a prefix, so a category error is refused by name |
| `mdlGuid` | `adlGuid` |

Tools take `mdrHandle`; as a convenience, `get_MDR` also accepts `mdrNumber` so a record someone cites by number can be read directly. Use the handle whenever you hold one.

In conversation, prefer `mdrNumber` once admitted and a head Claim paraphrase while staged.
