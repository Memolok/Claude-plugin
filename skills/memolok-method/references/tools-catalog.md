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
`discover_MDRs`. Two things bound that exception:

- **It is a read.** A number is not a durable address until anchoring — an uncommit releases it and the next
  admission takes it. A read that lands on the wrong record announces itself, because the response
  states both identifiers; a write would not, so no write tool accepts a number. **Anchoring is what
  makes a number safe to write down**, and `anchor_MDR` is how you cause it deliberately.
- **The response carries `mdrHandle`.** Once you have read the record, use its handle for everything
  else in that session — patches, transitions, wakes, an uncommit.

**Read `retractable` before suggesting an uncommit.**

### Every other entity carries a prefixed identifier

**The prefix says what the value addresses**, so a value in the wrong *reference field* is refused by
name — a Matter id in `correctsFact` fails as *a Matter*, not as malformed. Parameter names stay
descriptive; none is called `publicId`.

| Parameter | Shape |
| --- | --- |
| a Matter's `id`, `matterId` | `mt_` + 6 |
| `worldFactId` | `wf_` + 6 |
| `observedOutcomeId` | `oo_` + 6 |
| `analysisId` | `an_` + 6 |
| `scratchpadId` | `sp_` + 26 |
| `feedbackId` | `fb_` + 16 |
| `userId` | 16, no prefix |

Bodies are Crockford base32 — the ten digits and the letters except `I`, `L`, `O`, `U` — lowercase on
the wire, and a hyphen is refused rather than ignored. **Never construct, truncate, complete or
pattern-match one.** Pass back exactly what you were handed.

These replaced twenty-four hexadecimal characters, so an identifier quoted from an older session may
not resolve. **A shape complaint is not a diagnosis.** `scratchpadId must be 'sp_' followed by 26
Crockford base32 characters` is what you get for a typo *and* for a correctly-typed value from before
the change — the message cannot tell them apart, so neither can you. Read the value back to the user
and ask, rather than declaring it retired or repairing it by hand.

**`mdlGuid` is not in this table, and it is the one value that looks like another.** It is the same
shape as `userId` — sixteen Crockford base32 characters, no prefix — so nothing about the string says
which of the two you are holding. Never substitute one for the other, never infer the kind from the
value, and reason about nothing in it.

Ledger addresses were re-minted alongside the identifiers above, so one quoted from an older session
can fail two ways: refused as malformed, or accepted and simply not found. Neither is a permissions
problem and neither is a diagnosis — read the value back to the user.

All prose parameters use `{ markdown, lang? }` — see `prose-and-raci.md` for which are wrapped in a
`description` key.

## Read tools

### `ping` / `get_guidance`

`ping` takes no parameters and returns `"pong"`.

`get_guidance` returns the server's instruction text. **Always pass `pluginVersion`** — the version
stated at the top of `memolok-method` — so the server can say whether this pack still matches its
tools:

| Param | Type | Required |
| --- | --- | --- |
| `pluginVersion` | string | Send it always; older packs that cannot are still served |

The reply names the newest pack available and the oldest this server accepts. If it says your pack
is too old, its skills describe payloads the server now refuses — say so and recommend updating,
rather than proceeding and hitting write errors that read as data problems.

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

**One ledger's own metadata, not its contents.** For what is inside, use `discover_MDRs`,
`discover_matters`, `discover_world_facts` or `discover_observed_outcomes`.

**Not `get_MDR`.** One letter apart, and completely different: `get_MDL` takes only `mdlGuid` and
returns ledger metadata; `get_MDR` takes `mdlGuid` + a record key and returns a decision record. Reading
one while meaning the other produces a confident answer to the wrong question.

## Discovery reads: one shape, five tools

`discover_MDRs`, `discover_matters`, `discover_world_facts`, `discover_observed_outcomes` and
`discover_scratchpads` share a shape. Learn it once.

**One read per collection for choosing, and `get_*` for reading an entry whole.** All five answer as
prose you read rather than rows you filter.

| Param | Type | Required |
| --- | --- | --- |
| `mdlGuid` | string | yes |
| `query` | string | no |
| `limit` | int | no (default 25, clamped to 100) |
| `offset` | int | no (default 0) |

Each returns **Markdown, not an envelope**: a heading per entry, an identity line, the metadata lines
that collection carries, and its derived summary. The page states its own totals, says when you are
holding part of the ledger, and gives you the `offset` for the next one. Where an entry has nothing
derived the heading is the writer's own opening instead, and every page says which it is showing you
— permanently so on `discover_scratchpads`, which is the collection nothing derives onto.

**Every page renders a date line** — `Decided:` on records, `Raised:` on matters, `Admitted:` on
facts, `Observed:` on outcomes, `Touched:` on notes — so *"what came in this week"* is answerable without opening
anything. It may be absent on an entry old enough to predate the field, and on a record it is absent
whenever the record is staged, because `Decided:` is t₀ and a staged record has not reached one.

**The total is the whole match, not the page.** Holding fewer entries than it states means you have
not seen the ledger, and an answer that does not say so is claiming coverage it does not have.

**A discovery read is for choosing. None of them returns a whole entry** — `get_MDR`, `get_matter`,
`get_world_fact`, `get_observed_outcome` and `get_scratchpad` are the reads that do.

**The query grammar is tiny, deliberately.** Whitespace-separated terms, **ORed**,
case-insensitive. No operators, no phrases, no regex, no case toggle. Whole hyphenated tokens match;
partial ones do not. An entry matching two terms outranks one matching a single term. Order is
relevance when `query` is sent, each tool's natural order otherwise.

**An empty result with a `query` means nothing matched those words** — not that nothing exists.
Say which words you tried; do not fall back to enumerating everything.

Per-tool filters and natural order are below. Nothing else about the shape varies.

### `get_MDR`

| Param | Type | Required |
| --- | --- | --- |
| `mdlGuid` | string | yes |
| `mdrHandle` | int | exactly one of the two |
| `mdrNumber` | int | exactly one of the two |

Returns the full record — fish body, `mdrHandle`, `mdrNumber`, `retractable`, and any graph edges,
plus `analysisId` (`null` when no analysis produced it) — and the derived `title`, `summary` and
`subjects` where Memolok has produced them.

**Those three appear only on a sealed record**, because only sealed records are derived: a staged
record's prose is still editable and nothing would notice a summary going stale against it. They are
Memolok's reading of the record, never the decider's words — quote the fish body.

Pass the handle when you have one. `mdrNumber` is here for the case where a person cites a number
and you would otherwise sweep `discover_MDRs` to find its handle — one call instead of a
ledger-wide read.
**Exactly one of them MUST be passed.** A number can have been released by an uncommit and taken by a
later record, so check the record you get back is the one meant; the handle it returns is what you
use for every other tool.

### `discover_MDRs`

Shared discovery params, plus `status` (string, optional). Ledger order: by number, then handle.
Answers in **prose**: per record a heading, its identity, when it was decided, what the ledger says
its status is, the terms it names, its summary and its head Claim.

**This is the read for working out which decisions bear on what you are doing.** A record found here
is read with `get_MDR` without translating anything.

**Each record is named by its number and its handle** — `MDR-137 (handle 141)`. The number is what
you cite in prose; the handle is what `update_MDR`, `transition_MDR_status`, `record_observed_outcome`
and `get_MDR_learning_delta` all take, and none of them accepts a number. A staged record has no
number and is named `MDRh141 (staged — no number yet)`.

**`Decided:` is t₀, not when the record was minted.** A staged record shows no date line at all,
because it has not been decided; `get_MDR` is where both that and `createdAt`, the mint time a staged
record does have, can be read. An Uncommit removes `decidedAt` and leaves `createdAt` untouched, so a
demoted record loses the line.

**`status` is validated.** An unknown value is refused by name, listing the valid statuses. It used
to answer a typo with an empty list, which read as "no records".

**The search _query_ covers the whole fish** — head **Claim**, **Verdict**, alternatives, deliberation facts,
expected outcomes, open questions. So a record can match on reasoning that no heading shows, and the entry
will look unrelated to the query. The match window is what shows where the hit came from; quote that
rather than the Claim when explaining why an entry is there.

**The heading and the summary were composed from the record's spine** — its head Claim, the
alternative it chose, and its Verdict. They have not read the options it rejected or the arguments
about them. That asymmetry is deliberate: search is generous, the label is decisive.

**`Status:` is what the ledger says a record is, and `Rejected` is a commitment.** It means the
decision was not to proceed — a real, sealed outcome and not an abandoned draft. `Superseded` means a
later record replaced this one; what it decided still happened. Never report a rejected record as
unfinished.

**Only sealed records carry a derived artifact.** A staged record's prose is still editable and
nothing would notice a summary going stale against it, so a recent record appearing with its head
Claim as the heading is the ordinary state rather than a gap.

Where a record has been summarised the heading is Memolok's wording; where it has not, the heading is
the decider's own Claim. Every page says which it is showing you. **Quote the decider from `get_MDR`,
never from a heading.**

### `get_matter`

| Param | Type | Required |
| --- | --- | --- |
| `mdlGuid` | string | yes |
| `matterId` | string | yes |

Returns `{ id, mdlGuid, description, createdAt, raisedBy, takenUpBy }`, plus `title`, `summary`
and `subjects` where Memolok has derived them. Error: `Matter not found.`

**`raisedBy` absent means unrecorded, not anonymous.** Matters registered before the ledger
recorded a raiser have none and never will — nothing in storage could recover one, so nothing
was invented. Never read a missing `raisedBy` as evidence about who raised it.

**`description` is the raiser's words; the other three are Memolok's reading of them.** Nothing
in the response marks which is which. Quote `description` when you are quoting the person.

`takenUpBy` is one entry per analysis that took this matter up — `{ analysisId, created,
concludedAt, late, producesDecision }` — and `[]` when nobody has. `producesDecision` holds
`{ mdrHandle, mdrNumber }` per record **that analysis** minted; those records belong to the reasoning,
not to this matter, and nothing says any of them answers it. No rationale here — that is
`get_analysis`.

### `discover_matters`

Shared discovery params, plus `untaken` (bool, optional). Registration order, oldest first. Answers in
**prose**: per matter a heading, its id, when it was raised, the subjects it names, its summary, and
which analyses took it up — each named by its `analysisId` — with what they produced.

**This is the read for deciding what to work on.** A matter found here is read with `get_matter`
without translating anything, and the analysis id can go straight to `get_analysis` without opening
the matter first.

`untaken: true` is the unprocessed-bait inbox — every matter no analysis references. `untaken: false`
gives the complement — matters an analysis **has** taken up, which is not the same as no filter at
all. There is no status filter, because **a matter has no status**: what became of one reads from its
shape.

**Search reaches more than the raiser's words.** A query matches the derived title, subjects and
summary too, so a matter can come back for a term nobody typed into it — which is the point, since
the raiser was describing a problem rather than naming it.

**A matter naming your term among its subjects ranks first.** Those are what Memolok picked out as
what the matter is about, so they beat prose that merely repeats the word. Unprocessed matters are
still found, lower down. The match window shows which text matched, and prefers the raiser's own.

**A heading is Memolok's label, and nobody typed it.** Where a matter has not been summarised the
heading is the raiser's own opening instead, and every page says which it is showing you. That
matters here more than elsewhere: a matter is bait in somebody's own words, and paraphrasing a
machine's label back to them is how the words stop being theirs. **Quote the raiser from
`get_matter`** — the read that returns the body whole — never from a heading.

The answer states its own totals, says when you are holding only part of the ledger, explains an
empty result, and gives you the `offset` for the next page.

### `discover_world_facts`

Shared discovery params. Admission order, oldest first. Answers in **prose**: per fact a heading,
its id, when it was admitted, the terms it names, its summary, and what it corrects if anything.

**This is the read for working out which premises bear on what you are doing.** A fact found here is
read with `get_world_fact` without translating anything.

**The almanac only ever grows.** A corrected fact stays on the ledger beside the one correcting it,
so this read returns superseded premises alongside live ones and there is no live-facts-only filter.
An entry's `Corrects:` line says what it replaced, never whether it *was* replaced — nothing can yet
answer that. Do not present an old premise as current merely because it came back in a read.

Search reaches the derived title, summary and subjects as well as the admitted claim, so a fact can
match a term nobody typed into it.

**The headings and summaries are Memolok's words, and on this collection that matters more than
anywhere else.** A world fact is a premise decisions are reasoned from, so quoting a paraphrase of one
back as the admitted claim misstates what the ledger rests on. Where a fact has not been summarised
the heading is the admitter's own opening instead, and every page says which it is showing you.

### `discover_observed_outcomes`

Shared discovery params, plus `mdrHandle` (int, optional), which narrows to one record's wake.
Observation order, oldest first. Answers in **prose**: per outcome a heading, its id, the record it
was realized from in both keys, how it was discovered, what it says about the promise it tests, when
it was observed, the terms it names and its summary.

**This is the read for working out which of a record's wake bears on what you are doing** — a long
tail of observations against one record is exactly the case where opening each one to find out is
the expensive way.

**The heading describes what was observed. It never judges the decision.** The derivation is shown
the observer's own claim and nothing else — not the expectation the entry tests, not the record it
came from. A heading that sounds like a verdict is describing an observation that sounded like one.
The verdict is `testResult`, rendered separately on every entry that has one, and it is the field to
read when you want to know whether a promise held.

Search reaches the derived title, summary and subjects as well as the observer's claim.

### `get_analysis`

| Param | Type | Required |
| --- | --- | --- |
| `mdlGuid` | string | yes |
| `analysisId` | string | yes |

Returns `{ id, mdlGuid, references, producesDecision, analysisRationale, performedBy, concludedAt }`.
Error: `Analysis not found.`

Each entry in `references` is `{ motivatedBy, created, late }`. **`motivatedBy` carries its own kind
in its prefix** — `mt_` a Matter, `wf_` a World Fact, `oo_` an Observed Outcome — so nothing has to
be read alongside it to know what it addresses. It is **`null` where the input has been deleted**:
the reference still says an input was taken up and when, and can no longer say which. A reference has
no id of its own; it is addressed by the `{ analysisId, motivatedBy }` pair. `late` true means the
input was attached after `concludedAt`, so the rationale does not account for it; **`null` means
unanswerable** — a backfilled reference with no date, or an analysis with no conclusion — and never
"not late".

Point read; there is no `list_analyses`. Reach it by `analysisId` from `get_matter` or `get_MDR`.

### `get_world_fact`

`get_world_fact` takes `mdlGuid` + `worldFactId` and returns the whole admission —
`worldFactId`, `manifests`, `createdAt`, `createdBy`, optional `correctsFact` — plus the derived
`title`, `summary` and `subjects` where Memolok has produced them.

It is the read that returns the admitted claim whole, and therefore **the only one to quote a premise
from**. A heading on a discovery page is Memolok's label for the claim, not the claim.

A world fact says `createdBy` where a matter says `raisedBy`. The two mean the same thing and are
named for different ontology predicates; nothing turns on the difference when you are reading. Neither
is rendered on a discovery page — both are bare IRIs, and what a readable form would be is an open
question rather than a settled one.

### `get_observed_outcome`

`get_observed_outcome` takes `mdlGuid` + `observedOutcomeId` and returns the whole observation, plus
the derived `title`, `summary` and `subjects` where Memolok has produced them.

The read that returns the observer's wording whole — **quote an observer from here**, never from a
heading. And neither a heading nor a summary is the verdict: `testResult` is.

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

Returns `{ id, mdlGuid, description, createdAt, raisedBy, takenUpBy: [] }`. Record the raiser's
words **verbatim** — do not sharpen here.

`createdAt` and `raisedBy` are minted by the server from the call itself; there is no parameter
for either. **`raisedBy` is the one thing here that cannot be recovered later** — the creation
time survives in storage regardless, but who raised it is known only while the call is happening.

### `create_analysis`

| Param | Type | Required |
| --- | --- | --- |
| `mdlGuid` | string | yes |
| `motivatedBy` | string[] (`mt_` / `wf_` / `oo_` ids) | yes |
| `analysisRationale` | `{ markdown, lang? }` | yes |
| `producesDecision` | bool | no (default `true`) |
| `claimDescription` | `{ markdown, lang? }` | Path A only |

`motivatedBy` is a list: pass **every** input this reasoning took up, in any mix of the three
kinds. Empty raises. An admitted World Fact or Observed Outcome goes in as itself — do not register
a Matter restating it, which records neither the entry as the input nor the link, and cannot be
repaired later because Matters are immutable. Repeats of one id attach once, across kinds as within
one. The analysis concludes in this call, stamping `concludedAt` with the same instant it dates the
references, so they read as on time.

Returns `{ analysis, mdr? }`. Path A mints the record at `New`; Path B omits `mdr` entirely (absent,
not null).

### `attach_analysis_reference`

| Param | Type | Required |
| --- | --- | --- |
| `mdlGuid` | string | yes |
| `analysisId` | string | yes |
| `motivatedBy` | string (`mt_` / `wf_` / `oo_` id) | yes |

Takes up an input the analysis did not originally reference. **Allowed after the analysis
concluded** — that is what it is for. The reference is dated now, so it reads as `late: true` and the
sealed rationale is untouched. Returns the reference.

At most one reference per (input, analysis) pair; a second raises rather than replacing the first,
because two attachment times make lateness unanswerable.

### `retract_analysis_reference`

| Param | Type | Required |
| --- | --- | --- |
| `mdlGuid` | string | yes |
| `analysisId` | string | yes |
| `motivatedBy` | string (`mt_` / `wf_` / `oo_` id) | yes |

**Addressed by the pair it joins**, because that pair is the reference's identity. There is no
reference id anywhere on the surface to pass.

Withdraws a reference attached in error. Ungated, including against a committed record: an analysis
must describe reasoning that occurred. Returns `{ retracted: { analysisId, motivatedBy } }` — the
pair echoed back, so retracting several in one turn stays correlatable.

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
`openQuestions`, `verdict`, `chosenAlternative`, the four RACI fields, `amends`, `supersedes`,
`dependsOn`, `conflictsWith`, `settlesOpenQuestion`. Patches may be incremental — send only what
changed.

The five graph keys are **staged-only**, and on a resident even `[]` is refused: presence is what is
refused, because clearing an edge after admission would undo something the ledger has published. Each
of them Anchors the record it names at admission, so that record can never be Uncommitted by anyone
afterwards — `amends` included, which is the one that catches people out, since it sounds lighter
than superseding and costs the target exactly the same.

`amends` and `supersedes` additionally need an **Accepted** record at each end: both act on content,
and only an Accepted record has content still in force. `amendedBy` and `supersededBy` are
server-minted and refused on write.

**You name every id on `alternatives`, `expectedOutcomes`, `openQuestions`** — prefixed `alt-`,
`eo-`, `oq-` for their list, unique within it, on `create_MDR` and `update_MDR` alike. Name them for
what they are: `alt-cache-layer` is what a reader meets in `chosenAlternative` later. Omitting an id
is refused, since these lists replace wholesale and an id-less item cannot say whether it is the old
one or a new one; a missing prefix is refused with the correction rather than added for you, because
rewriting an id would orphan references the patch does not carry. `chosenAlternative` and
`deliberationFacts[].onAlternative` must name an alternative that exists once the patch lands.

### `transition_MDR_status`

`mdlGuid`, `mdrHandle`, `status`. Admission to `Accepted` or `Rejected` sets `decidedAt`, assigns
`mdrNumber`, and mints reciprocals.

### `uncommit_MDR`

`mdlGuid`, `mdrHandle`, optional `reason`. Requires `admin` or `owner`, status `Accepted` or
`Rejected`, and `retractable: true`. Demotes to staged, clearing `mdrNumber` and `decidedAt`.

### `anchor_MDR`

| Param | Type | Required |
| --- | --- | --- |
| `mdlGuid` | string | yes |
| `mdrHandle` | int | yes — must be a ledger resident |
| `kind` | `project` \| `other` | yes |

**Declares that something outside the ledger cites this record**, which stops an Uncommit releasing
its number. Call it **before** writing the citation, not after — the **`record-decision`** skill
carries when, and in what form the citation is written.
Member-level, deliberately: gating it above the write it accompanies would leave the citation
unanchored.

`project` is a citation in a project artifact — a source file, a document in the repository. `other`
is anywhere else one can go: an email, a chat message, a ticket, a slide. **Only the kind is
recorded, never where the citation lives**, so there is no location parameter to look for and none to
supply. Declaring the same kind twice is a no-op.

**Permanent, and unverified.** There is no withdrawal — that is what makes it worth anything — and
the server cannot see your files or your mail, so nothing checks the claim. Say so if a user asks to
undo one: it is corrected by a later record, not by a call.

Returns the record. A staged record has no number for anything to cite, so this refuses one.

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

**A `scratchpadId` is the longest identifier in the product** — `sp_` followed by **26** Crockford
base32 characters, against six for a ledger entity — and that is deliberate rather than incidental.
It is untypeable and unmemorable, which discourages citing a note in the one way a rule cannot: at
the moment of writing, before anyone consults a rule.

Pass a `scratchpadId` to `hasContext`, `correctsFact`, `motivatedBy`, `tests.outcomeId` or `evidence`
and the call fails telling you the value **is a scratchpad** — a category error, not a typo. Nothing
in a ledger may reference one; if the material matters to a decision, admit it as a World Fact and
cite that.

**Scratchpad ids were re-minted and the old form is gone.** A stale `sp_` + 24 hex value does not
resolve. It comes back as a shape complaint, which is the same answer a typo gets — so do not tell a
user their id is retired on the strength of that message; read it back to them and ask.

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

### `discover_scratchpads`

Shared discovery params — they and the query grammar are in the discovery preamble above. Most
recently touched first, which is what the `Touched:` line on each entry states. **Never carries a
body**: `get_scratchpad` is the only read that does.

**A note's title is Memolok's, because a note has no other.** Nobody types one, so where nothing has
been derived yet the heading is a cut of what somebody pasted — and the page says which of the two it
is showing you.

**`query` is how you answer "what did I save about X?"** Search reaches the derived title, summary
and subjects as well as the body, so a note can be found by a term it does not itself contain — which
matters here more than anywhere, because whoever pasted a vendor's pricing page was not choosing
search terms. Paging through everything to read it is what this shape exists to prevent.

**Where a note has no summary yet, the opening words are a positional trim.** They are a handle for
naming the note, not a description of it, and a long note's opening says nothing about what the note
argues. When the question is about content, `get_scratchpad` the body.

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
| `feedbackId` | string (`fb_` + 16) | yes |

Your own reports only. A stranger's id returns *not found* rather than a permission error, and so
does a report deleted during triage.

### `update_feedback`

| Param | Type | Required |
| --- | --- | --- |
| `feedbackId` | string (`fb_` + 16) | yes |
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
| `supersedes may only target Accepted residents (MDR-{n} is {status}).` | Bad supersession target. A `Superseded` one has already been retired, so a second attempt has nothing to retire |
| `amends may only target Accepted residents (MDR-{n} is {status}).` | Same rule, same reason: only an `Accepted` record has content for the amendment to leave standing |
| `A Rejected Memolok Decision Record must not carry amends (…).` | `amends` on a record admitting as `Rejected` — nothing it says is in force, so it changes nothing |
| `Field {x} cannot be authored over MCP: it is either minted by the server at admission or not yet writable.` | `amendedBy` / `supersededBy` on a patch |
| `This Memolok Decision Record is Anchored ({kind}) and cannot be Uncommitted.` | Uncommit on an anchored record. **The kind is named** — `project` or `other` for a declared anchor, `conflict pair` when the record is held only by a `conflictsWith` edge (which anchors both sides, with no way to dissolve it), otherwise `ledger` for a derived cause |
| `Only Accepted or Rejected Memolok Decision Records may be Uncommitted.` | Wrong status for uncommit |
| `Only a ledger-resident Memolok Decision Record can be Anchored. A staged record has no ledger number for anything to cite.` | `anchor_MDR` on a staged record — wait for admission |
| `kind must be one of project, other.` | `anchor_MDR` with anything else; the vocabulary is closed |
| `Observed Outcomes can only realizeFrom a ledger-resident Memolok Decision Record...` | Wake on a staged record |
| `discoveryType Expected requires tests referencing an expectedOutcome.` | Missing `tests` |
| `An analysis must take up at least one input; motivatedBy is empty.` | Empty `motivatedBy` |
| `That input is already referenced by this analysis.` | Duplicate attach — the existing reference stands |
| `{field} is a scratchpad id.` | A `sp_…` value passed to a reference field. Nothing may cite a note — admit a World Fact instead |
| `{field} must be 'sp_' followed by 26 Crockford base32 characters.` | Something that is not a scratchpad id passed where one was expected |
| `Scratchpad not found.` | Missing note, wrong ledger, or already deleted |
| `A scratchpad body may be at most 65536 bytes;…` | Paste too large — split it, or keep a pointer to the source |
| `claimDescription is required when analysis produces a Memolok Decision Record.` | Path A without a claim |
| `A Memolok Decision Record cannot cite its own Observed Outcome in hasContext...` | DTP violation |
| `{field} must be a feedback report id of the form 'fb_<16 Crockford base32 characters>'.` | A ledger id passed to a feedback tool — different kinds of address |
| `{field} is a feedback report id.` | An `fb_…` value passed to a ledger reference field. A report is not a ledger entity |
| `Feedback report not found.` | Not yours, wrong id, or deleted during triage |
| `reports[n] may not set serverVersion, …` | The server records the build; a caller cannot claim one |
| `reports[n].evidence[m] must carry either a response … or an excerpt` | An evidence item that asserts nothing |
