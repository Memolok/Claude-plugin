# MCP tools catalog

Every tool except `ping` and `get_guidance` requires authentication. All ledger tools are scoped by `mdlGuid`.

## Identity keys

| Field | When | Role |
| --- | --- | --- |
| `mdrHandle` | Every mint | Addressing key for record tools, as `MDRh7`, including after admission |
| `mdrNumber` | Admission only | Ledger citation; `null` while staged. Taken as `MDR-7` by the tools that read a record or its wake |
| `retractable` | Computed at read | `null` staged; `true` uncommit-eligible; `false` anchored |

Never invent either value. Never address a record tool by a raw database id.

**Every record tool takes one `mdr`: the record's citation.** `MDRh7` cites it by its handle and `MDR-7` by its number, spelled exactly so, and answers carry both keys as plain integers. **The handle is the standard path whenever you have one** — `MDRh7` is what every tool that changes a record takes. A read takes `MDR-7` as well, for the journey where someone cites "MDR-7" and you hold no handle: read it directly rather than scanning `discover_MDRs`. Two things bound that:

- **It is a read.** A number is not a durable address until anchoring — an uncommit releases it and the next admission takes it. A read that lands on the wrong record announces itself, because the response states both identifiers; a write would not, so no tool that changes a record accepts a number. **Anchoring is what makes a number safe to write down**, and `anchor_MDR` is how you cause it deliberately.
- **The response carries `mdrHandle`.** Once you have read the record, cite its handle as `MDRh` for everything else in that session — patches, transitions, an uncommit.

**Only a ledger resident has a wake or a learning delta**, so `get_MDR_learning_delta`, `discover_observed_outcomes` narrowed to one record and `record_observed_outcome` find no staged record, however it is cited. Recording an outcome changes no record; it adds an entry beside one, and anchors it, so it takes either citation.

**Read `retractable` before suggesting an uncommit.**

### Every entity carries a prefixed identifier

**The prefix says what the value addresses**, so a value in the wrong *reference field* is refused by name — a Matter id in `correctsFact` fails as *a Matter*, not as malformed. Parameter names stay descriptive; none is called `publicId`.

| Parameter | Shape |
| --- | --- |
| a Matter's `id`, `matterId` | `mt_` + 6 |
| `worldFactId` | `wf_` + 6 |
| `observedOutcomeId` | `oo_` + 6 |
| `analysisId` | `an_` + 6 |
| `scratchpadId` | `msp_` + 26 |
| `feedbackId` | `mfb_` + 16 |
| `mdlGuid` | `mdl_` + 16 |
| `userId` | `musr_` + 16 |

Every kind can be told apart by its prefix. Bodies are Crockford base32 — the ten digits and the letters except `I`, `L`, `O`, `U` — lowercase on the wire, and a hyphen is refused rather than ignored. **Never construct, truncate, complete or pattern-match one.** Pass back exactly what you were handed.

**An id exists only once the call that creates it has returned.** Anything that cites an entry, whether a reference field (`motivatedBy`, `hasContext`, `correctsFact`, `tests`) or an id written into prose, goes out after that reply and uses the value it carries: never in the same parallel batch, never with a stand-in to fix later. A reference field refuses an unknown id; prose is not checked, and a Matter cannot be edited at all. A record's handle and number follow the same rule. Ids you name yourself (`alt-`, `eo-`, `oq-`) are yours to choose. Code, commits and documents: `memolok-citations`.

All prose parameters use `{ markdown, lang? }` — see `prose-and-raci.md` for which are wrapped in a `description` key.

## Read tools

### `ping` / `get_guidance`

`ping` takes no parameters and returns `"pong"`.

`get_guidance` returns the server's instruction text. **Always pass `pluginVersion`** — the version stated at the top of `memolok-method` — so the server can say whether this pack still matches its tools:

| Param | Type | Required |
| --- | --- | --- |
| `pluginVersion` | string | Send it always; older packs that cannot are still served |

The reply names the newest pack available and the oldest this server accepts. If it says your pack is too old, its skills describe payloads the server now refuses — say so and recommend updating, rather than proceeding and hitting write errors that read as data problems.

### `whoami`

No parameters. Returns `{ userId, iri, givenName, familyName, mbox }`, the last three only when held. `iri` is the user as every record names them: write it as given, never assembled from `userId`.

### `resolve_names`

| Param | Type | Required |
| --- | --- | --- |
| `iris` | string[] | yes, up to 100 |

Returns `{ names: [{ iri, givenName, familyName }] }`, each IRI once, a name `null` where none is given to you. **This is how a person an entry names is named to the user**: any IRI you meet, a `createdBy`, a RACI value, a provenance `by`. You are given a name where you share a ledger with that person, or where Memolok credits them with something on a ledger you may read; being named in testimony gives nobody a name. **A `null` never says why**: an IRI naming nobody, a person you may not see named, one with no name held, and a person who vanished from the ledger (who shows as a pseudonym IRI) all answer alike. Say only that no name is known; never guess one or offer a reason.

### `get_provenance`

| Param | Type | Required |
| --- | --- | --- |
| `kind` | `MDL` \| `MDR` \| `matter` \| `world_fact` \| `observed_outcome` \| `analysis` \| `scratchpad` \| `feedback` | yes |
| `mdlGuid` | string | every kind but `feedback` |
| `entry` | string: `MDRh7`, `MDR-7`, or the entry's own id | every kind but `MDL` |

**Memolok's own account of who wrote what, and when** — the read for *"who created this?"*, *"who changed it?"*, *"who sealed it, with which roles named?"*. Returns `{ created, revisions, testimony, references, sealings }`: the creation's `by`, `at` and `basis` (the world fact a creation rests on where Memolok inferred its creator rather than witnessed it); every later write, oldest first, by `kind` (such as `edit`, `transition`, `share`, `accept`; new kinds appear as Memolok records them); each testimony value with the write that asserted it; for an analysis, every reference it has held, **retracted ones included**, which no other read shows; for a record, the roles it named at each sealing. Readable wherever what it describes is; a feedback report's is yours alone. Every `by` is an IRI: name it with `resolve_names`.

### `get_MDLs`

No parameters. Returns `{ mdls: [{ mdlGuid, title, role }] }`. A chooser across ledgers — it does not carry the Ledger Intent.

### `get_MDL`

| Param | Type | Required |
| --- | --- | --- |
| `mdlGuid` | string | yes |

Returns `{ mdlGuid, title, role, ledgerIntent, members }`. `ledgerIntent` is `null` when the ledger has never stated a purpose — that is normal, not an error.

`members` is everyone holding a role on the ledger, each once: `{ iri, givenName, familyName, roles }`, a name `null` where the account holds none. **It is where a person named in conversation is found.** Absent where your role may not list them.

**One ledger's own metadata, not its contents.** For what is inside, use `discover_MDRs`, `discover_matters`, `discover_world_facts` or `discover_observed_outcomes`.

**Not `get_MDR`.** One letter apart, and completely different: `get_MDL` takes only `mdlGuid` and returns ledger metadata; `get_MDR` takes `mdlGuid` + a record key and returns a decision record. Reading one while meaning the other produces a confident answer to the wrong question.

## Discovery reads: one shape, five tools

`discover_MDRs`, `discover_matters`, `discover_world_facts`, `discover_observed_outcomes` and `discover_scratchpads` share a shape. Learn it once.

**One read per collection for choosing, and `get_*` for reading an entry whole.** All five answer as prose you read rather than rows you filter.

| Param | Type | Required |
| --- | --- | --- |
| `mdlGuid` | string | yes |
| `query` | string | no |
| `limit` | int | no (default 25, clamped to 100) |
| `offset` | int | no (default 0) |

Each returns **Markdown, not an envelope**: a heading per entry, an identity line, the metadata lines that collection carries, and its derived summary. The page states its own totals, says when you are holding part of the ledger, and gives you the `offset` for the next one. Where an entry has nothing derived the heading is the writer's own opening instead, and every page says which it is showing you. On `discover_scratchpads` that fallback is a trim of whatever was pasted rather than an opening somebody wrote, because nobody types a title for a note. **A heading or summary is Memolok's wording, never the author's: quote a person only from the `get_*` read that returns the entry whole.** On world facts that matters most, because a paraphrase quoted as the admitted claim misstates what the ledger rests on.

**Every page renders a date line** — `Decided:` on records, `Registered:` on matters, `Admitted:` on facts, `Observed:` on outcomes, `Touched:` on notes — so *"what came in this week"* is answerable without opening anything. It may be absent on an entry old enough to predate the field, and on a record it is absent whenever the record is staged, because `Decided:` is t₀ and a staged record has not reached one.

**The total is the whole match, not the page.** Holding fewer entries than it states means you have not seen the ledger, and an answer that does not say so is claiming coverage it does not have.

**A discovery read is for choosing. None of them returns a whole entry** — `get_MDR_details`, `get_matter`, `get_world_fact`, `get_observed_outcome` and `get_scratchpad` are the reads that do. `get_MDR` returns what a record decided, which is usually what you want.

**The query grammar is tiny, deliberately.** Search is lexical: whitespace-separated terms, ORed, case-insensitive and English-stemmed; no operators, phrases or regex; a whole hyphenated token matches and a partial one does not. An empty result means no entry used those words — say which you tried. An entry matching two terms outranks one matching a single term. Order is relevance when `query` is sent, each tool's natural order otherwise; do not fall back to enumerating everything. Search reaches the derived title, subjects and summary as well as the author's words, so an entry can match a term nobody typed into it, and an entry naming your term among its subjects ranks first.

Per-tool filters and natural order are below. Nothing else about the shape varies.

### `get_MDR`

| Param | Type | Required |
| --- | --- | --- |
| `mdlGuid` | string | yes |
| `mdr` | string: `MDRh7` or `MDR-7` | yes |

**`get_MDR` answers what a record decided; `get_MDR_details` answers why and how.** This is the read for finding out about a record. Returns `mdrHandle`, `mdlGuid`, `mdrNumber`, `status`, `retractable`, `title`, `createdAt`, `createdBy`, `decidedAt`, `hasNeed`, `verdict`, every open question whole, and every edge — `hasContext`, `analysisId`, `supersedes`, `supersededBy`, `amends`, `amendedBy`, `dependsOn`, `conflictsWith`, `settlesOpenQuestion` — plus a `details` key sizing the prose it leaves out.

`createdBy` is Memolok's own account of who created the record: its owner, and who answers for it. Nobody writes it. The four RACI fields are testimony, and `authoredBy` is not its creator. It is `null` where Memolok holds no account of the creation; never read `authoredBy` in its place.

Cite the handle when you have one. The identity section above says why a number is a read-only convenience, and the handle the response carries, cited as `MDRh`, is what every other tool takes.

### `get_MDR_details`

Same parameters and rule as `get_MDR`. Returns the whole record: `get_MDR`'s fields less `details`, plus `alternatives`, `deliberationFacts`, `chosenAlternative`, `expectedOutcomes`, the four RACI fields, and the derived `summary` and `subjects` — those two only on a sealed record, since a staged one's prose may still change.

**Read it when the question is why or how** — the options weighed, the arguments, what was expected — never reconstructing an argument from the Verdict's wording; and **before patching an array you did not just draft**, since a patch replaces it whole. Its output can be sent back through `update_MDR` as it stands.

### `discover_MDRs`

Shared discovery params, plus `status` (string, optional; an unknown value is refused by name). Ledger order: by number, then handle. Per record: a heading, `MDR-7 (handle 11)` or `MDRh12 (staged — no number yet)`, `Decided:` (t₀ — absent while staged, and removed by an Uncommit), `Status:`, `Created by:`, the terms it names, its summary and an excerpt of its head Claim.

**Search covers the whole fish** — Claim, Verdict, alternatives, deliberation facts, expected outcomes, open questions — so a hit can look unrelated to its heading; the match window shows where it came from. **`Rejected` is a sealed commitment not to proceed and `Superseded` means a later record replaced this one**; neither is an unfinished record.

### `get_matter`

| Param | Type | Required |
| --- | --- | --- |
| `mdlGuid` | string | yes |
| `matterId` | string | yes |

Returns `{ id, mdlGuid, description, createdAt, createdBy, raisedBy, authoredBy, takenUpBy }`, plus `title`, `summary` and `subjects` where Memolok has derived them. Error: `Matter not found.`

**`createdBy` is the account that registered it; `raisedBy` and `authoredBy` are what that caller stated** — who had the initiative, and who wrote the words. `raisedBy` empty or `authoredBy` `null` means unstated, never nobody, and never the registering account.

**`description` is its author's words; the other three are Memolok's reading of them.** Nothing in the response marks which is which. Quote `description` when you are quoting the person.

`takenUpBy` is one entry per analysis that took this matter up — `{ analysisId, created, concludedAt, late, producesDecision }` — and `[]` when nobody has. `producesDecision` holds `{ mdrHandle, mdrNumber }` per record **that analysis** minted; those records belong to the reasoning, not to this matter, and nothing says any of them answers it. No rationale here — that is `get_analysis`.

### `discover_matters`

Shared discovery params, plus `untaken` (bool, optional): `true` is the unprocessed-bait inbox — every matter no analysis references; `false` is the complement, which is not the same as no filter. There is no status filter, because **a matter has no status**. Registration order, oldest first. Per matter: a heading, its id, `Raised by:` (by name where Memolok gives you one, anyone else as stated, `not recorded` where nobody was), `Registered:`, subjects, summary, and which analyses took it up — each named by its `analysisId` — with what they produced. The match window prefers the author's own words.

### `discover_world_facts`

Shared discovery params. Admission order, oldest first. Per fact: a heading, its id, `Admitted:`, subjects, summary, and `Corrects:` where it corrects an earlier fact.

**The almanac only ever grows.** A corrected fact stays beside the one correcting it, there is no live-facts-only filter, and `Corrects:` says what an entry replaced, never whether it *was* replaced — nothing can yet answer that. Do not present an old premise as current merely because it came back.

### `discover_observed_outcomes`

Shared discovery params, plus `mdr` (`MDRh7` or `MDR-7`, optional), which narrows to one record's wake. Observation order, oldest first. Per outcome: a heading, its id, the record it was realized from in both keys, how it was discovered, `testResult` where it tests a promise, `Observed:`, `Created by:`, subjects and summary.

**The heading never judges the decision**: the derivation sees the observer's claim alone, not the expectation the entry tests. Whether a promise held is `testResult`, nothing else.

### `get_analysis`

| Param | Type | Required |
| --- | --- | --- |
| `mdlGuid` | string | yes |
| `analysisId` | string | yes |

Returns `{ id, mdlGuid, references, producesDecision, analysisRationale, performedBy, createdAt, concludedAt }`. Error: `Analysis not found.` Every role on the ledger may read an analysis.

Each entry in `references` is `{ motivatedBy, created, late }`. **`motivatedBy` carries its own kind in its prefix** — `mt_` a Matter, `wf_` a World Fact, `oo_` an Observed Outcome — so nothing has to be read alongside it to know what it addresses. It is **`null` where the input has been deleted**: the reference still says an input was taken up and when, and can no longer say which. A reference has no id of its own; it is addressed by the `{ analysisId, motivatedBy }` pair. `late` true means the input was attached after `concludedAt`, so the rationale does not account for it; **`null` means unanswerable** — a backfilled reference with no date, or an analysis with no conclusion — and never "not late".

Point read; there is no `list_analyses`. Reach it by `analysisId` from `get_matter` or `get_MDR`.

### `get_world_fact`

`get_world_fact` takes `mdlGuid` + `worldFactId` and returns the whole admission — `worldFactId`, `manifests`, `createdAt`, `createdBy`, `authoredBy`, `observedBy`, optional `correctsFact` — plus the derived `title`, `summary` and `subjects` where Memolok has produced them. `createdBy` is the account that admitted it; `authoredBy` and `observedBy` are testimony of who wrote its words and who observed what it states.

### `get_observed_outcome`

`get_observed_outcome` takes `mdlGuid` + `observedOutcomeId` and returns the whole observation, plus the derived `title`, `summary` and `subjects` where Memolok has produced them. `createdBy` is the account that recorded it, Memolok's own account; `observedBy` and `authoredBy` are testimony of who saw it and who wrote its words.

### `get_MDR_learning_delta`

| Param | Type | Required |
| --- | --- | --- |
| `mdlGuid` | string | yes |
| `mdr` | string: `MDRh7` or `MDR-7` | yes |

Only a ledger resident has a learning delta, so a staged record is not found, however it is cited.

Returns each expected outcome with the wakes that test it, plus every Emergent and Deducible wake again on its own, so one that tests an outcome appears in both places. Coexisting assessments are all returned — there is no "current result" field. Each wake carries `createdBy` as `get_observed_outcome` does.

## Write tools

**A write answers with an acknowledgement, never the entry it wrote**: the entry's identifiers, its lifecycle state, the times the server stamped, and a `body` key — `"2,432 characters, not echoed; get_MDR reads it"` — giving how much prose the entry now holds. The count is a sanity check: roughly what you sent, or whether a patch grew or shrank the record. Read the entry with the tool `body` names when you need what landed; a field missing from an acknowledgement is not a fault. The exceptions are named where they occur.

### `create_MDL`

| Param | Type | Required |
| --- | --- | --- |
| `title` | string | yes |
| `ledgerIntent` | `{ description: { markdown, lang? } }` | no |

Caller becomes `owner`. Returns the same shape as `get_MDL`. Nothing is mailed and no consent is asked: a creator consents to what the ledger shows of them each time they confirm an invitation to it (`share_MDL`).

Refused with `You do not have the necessary permissions to create ledgers.` when the user's account is not allowed to create one. Every account can from the day it is created, so this is an administrator's matter: say so, and do not retry.

Pass `ledgerIntent` when the user has agreed a purpose — one call, not create-then-set. Note the nested shape: `{ "description": { "markdown": "…" } }`, **not** a flat `{ markdown }`.

### `set_ledger_intent`

| Param | Type | Required |
| --- | --- | --- |
| `mdlGuid` | string | yes |
| `ledgerIntent` | `{ description: { markdown, lang? } }` | yes |

**Full replacement, not an append.** No earlier wording can be read back; `get_provenance` says only who wrote it and when. Read the current statement with `get_MDL` first, then send the complete new one.

Requires `admin` or `owner`. Returns the same shape as `get_MDL`.

### `set_MDL_title`

| Param | Type | Required |
| --- | --- | --- |
| `mdlGuid` | string | yes |
| `title` | string | yes |

Renames the ledger. Its `mdlGuid`, and every citation of it or of its entries, stay as they are. Requires `admin` or `owner`. Returns the same shape as `get_MDL`.

A `.memolok/mdl.yml` carrying `mdlTitle` holds the old name from this moment on — where project files are available, offer to update it.

### `share_MDL`

| Param | Type | Required |
| --- | --- | --- |
| `mdlGuid` | string | yes |
| `iri` | string | yes |

**Prepares an invitation and grants nothing by itself.** The inviter confirms each invitation on their Memolok sharing and privacy page, from the mail Memolok sends them, and the invitee then accepts it on theirs. The inviter's confirmation is their consent: it tells them the invitation carries their name and details, and that whoever accepts reads everything on the ledger and, like everyone who joins, sees the people on it with their roles; only the acceptance gives them `visitor`, which reads everything on the ledger but notes and changes nothing. Neither step can be taken through any tool, by you or for them, and a request not confirmed within seven days lapses. Requires `admin` or `owner`. Returns `{ iri, role, invited }`: `invited` true for an invitation prepared, false only for a demo account, which is given the role at once or already holds it.

`iri` is the account's `…/m/<musr_…>#id`, exactly as `whoami` gives it to its holder or a ledger you share gives it in `get_MDL`'s `members`. There is no lookup by email or name, and no page shows it: ask the person to read theirs with `whoami` in their own session, as their Memolok identifier; never say "IRI" to a person.

**Confirm the ledger and the person with the user before the call, then say what is still to do**: they confirm from their own mail, the invitee accepts from theirs. Refused: `already-member` (they hold a role there, which an invitation never changes), `holds-another-role` (a demo account holding another role, which it keeps), `already-invited` (one of yours waits for them; it is withdrawn on their Memolok sharing and privacy page, not through any tool), `unknown-account` (no account, or an expired one). No tool takes someone else's access away or gives a wider role; those are a Memolok administrator's.

### `leave_MDL`

| Param | Type | Required |
| --- | --- | --- |
| `mdlGuid` | string | yes |
| `leaveWithNoOwner` | bool | no (default false) |

**Destructive, and at once**: gives up the caller's own role, so they no longer read or write the ledger, and only a new invitation brings them back. What they wrote there still names them; that is `request_MDL_vanish`. Returns `{ mdlGuid, left }`. Memolok mails them a receipt. The last owner is refused with `last-owner`: send `leaveWithNoOwner: true` **only after telling the person the ledger will be left with no owner and hearing them agree**, never on the refusal alone.

### `request_MDL_vanish`

| Param | Type | Required |
| --- | --- | --- |
| `mdlGuid` | string | yes |
| `leaveWithNoOwner` | bool | no (default false) |

**Prepares the caller's own vanishing, and nothing happens until they confirm it**: Memolok mails them a link to their Memolok sharing and privacy page, and only confirming there makes them vanish. The request lapses after seven days. A vanish the person starts on that page themself takes effect there at once. **Once confirmed it cannot be undone**: every read of the ledger shows a pseudonym in their place, with no name, on everything it held then, and they leave the ledger. Memolok then mails them a receipt. Only a person can make themself vanish; there is no asking it for anyone else. Returns `{ mdlGuid, requested }`. Same `last-owner` rule as `leave_MDL`; `already-vanished` where they have vanished already and nothing has named them there since.

**`mail-unavailable`** on either preparing tool (`share_MDL`, `request_MDL_vanish`): the request stands, its mail did not go; the same call again sends it.

### `register_matter`

| Param | Type | Required |
| --- | --- | --- |
| `mdlGuid` | string | yes |
| `description` | `{ markdown, lang? }` | yes |
| `raisedBy` | string[] | no |
| `authoredBy` | string | no |

Returns `{ id, mdlGuid, createdAt, body }`. Record the words **verbatim** — do not sharpen here. Cites another entry? Create it first and use the id it returns; a matter's words cannot be edited.

`raisedBy` is whoever had the initiative — whose suggestion, report or request this is, however it reached you — and `authoredBy` whoever wrote the words. Neither is filled in for you, and **who raised it is known only now**: state it on this call. The registering account is recorded as `createdBy`.

**Naming a person.** The user is `whoami`. Anyone else named in conversation: look up their given and family name in `get_MDL`'s `members` — one match → that `iri`; several → ask which; none → ask for something that identifies them, and write that. Unattended → an `iri` only when both names match exactly one member; otherwise the name as given.

### `create_analysis`

| Param | Type | Required |
| --- | --- | --- |
| `mdlGuid` | string | yes |
| `motivatedBy` | string[] (`mt_` / `wf_` / `oo_` ids) | yes |
| `analysisRationale` | `{ markdown, lang? }` | yes |
| `producesDecision` | bool | no (default `true`) |
| `claimDescription` | `{ markdown, lang? }` | Path A only |

`motivatedBy` is a list: pass **every** input this reasoning took up, in any mix of the three kinds. Empty raises. An admitted World Fact or Observed Outcome goes in as itself — do not register a Matter restating it, which records neither the entry as the input nor the link, and cannot be repaired later because Matters are immutable. Repeats of one id attach once, across kinds as within one. The analysis concludes in this call, stamping `concludedAt` with the same instant it dates the references, so they read as on time. The whole body is checked before anything is written, so a refused call leaves nothing behind.

Returns `{ analysis, mdr? }`: `analysis` is `{ id, mdlGuid, createdAt, concludedAt, references, body }`, and `mdr` is the record acknowledgement `create_MDR` returns. Path A mints the record at `New`, naming the analysis in its `analysisId`; Path B omits `mdr` entirely (absent, not null). Performing an analysis is for members, admins and owners, and Path A also needs the permission to create decision records.

**A call that fails part-way says so**, with code `incomplete`: the analysis was only partly created, and repeating the call creates a separate analysis rather than finishing this one. Do not repeat it on your own. Tell the user, quote the Reference the error carries, and check `get_matter`'s `takenUpBy` for what was written before deciding anything.

### `attach_analysis_reference`

| Param | Type | Required |
| --- | --- | --- |
| `mdlGuid` | string | yes |
| `analysisId` | string | yes |
| `motivatedBy` | string (`mt_` / `wf_` / `oo_` id) | yes |

Takes up an input the analysis did not originally reference. **Allowed after the analysis concluded** — that is what it is for. The reference is dated now, so it reads as `late: true` and the sealed rationale is untouched. Returns the reference.

At most one reference per (input, analysis) pair. Attaching an input the analysis already references changes nothing and answers the reference as it stands, with the time it was first attached, so a repeated call is safe.

### `retract_analysis_reference`

| Param | Type | Required |
| --- | --- | --- |
| `mdlGuid` | string | yes |
| `analysisId` | string | yes |
| `motivatedBy` | string (`mt_` / `wf_` / `oo_` id) | yes |

**Addressed by the pair it joins**, because that pair is the reference's identity. There is no reference id anywhere on the surface to pass.

Withdraws a reference attached in error, including against a committed record: an analysis must describe reasoning that occurred. Returns `{ retracted: { analysisId, motivatedBy } }` — the pair echoed back, so retracting several in one turn stays correlatable. A pair that is not attached answers `Analysis reference not found.`, on the first call and on every repeat.

**Admins and owners only**, whoever performed the analysis: a member is refused, even on their own.

### `reopen_analysis`

| Param | Type | Required |
| --- | --- | --- |
| `mdlGuid` | string | yes |
| `analysisId` | string | yes |

Clears `concludedAt`. Refused once any produced record carries `decidedAt` — uncommit that record first. Use it when the reasoning genuinely was not finished; for scope that arrived afterwards, a late reference is the honest record, not a reopen. **Admins and owners only**, as for retracting. Returns the analysis acknowledgement `create_analysis` returns.

### `create_MDR`

| Param | Type | Required |
| --- | --- | --- |
| `mdlGuid` | string | yes |
| `claimDescription` | `{ markdown, lang? }` | yes |
| `status` | string | yes — canonical values only |
| `alternatives` | `[{ id, label?, description, satisfies? }]` | no |
| `deliberationFacts` | `[{ onAlternative?, description }]` | no |
| `expectedOutcomes` | `[{ id, description }]` | no |
| `openQuestions` | `[{ id, description }]` | no |
| `verdict` | `{ description }` | no |
| `chosenAlternative` | string | no |
| `authoredBy`, `decidedBy` | string | no |
| `consulted`, `informed` | string[] | no |

Expert path only — there is no matter parameter. Returns the **record acknowledgement**, `{ mdrHandle, mdlGuid, mdrNumber, status, retractable, createdAt, decidedAt, body }`, with a minted `mdrHandle`; `mdrNumber` only if created at `Accepted` or `Rejected`. Every record write returns this shape.

### `update_MDR`

| Param | Type | Required |
| --- | --- | --- |
| `mdlGuid` | string | yes |
| `mdr` | string: `MDRh7` | yes |
| `patch` | object | yes |

Patch keys: `hasNeed`, `hasContext`, `alternatives`, `deliberationFacts`, `expectedOutcomes`, `openQuestions`, `verdict`, `chosenAlternative`, the four RACI fields, `amends`, `supersedes`, `dependsOn`, `conflictsWith`, `settlesOpenQuestion`. Patches may be incremental — send only what changed.

| Key | Value |
| --- | --- |
| `hasContext` | ordered `string[]` of `wf_`/`oo_` ids from this ledger |
| `amends`, `supersedes`, `dependsOn`, `conflictsWith` | `int[]` of admitted `mdrNumber`s |
| `settlesOpenQuestion` | `[{ hostMdrNumber, openQuestionId }]` |

The five graph keys and `hasContext` are **staged-only** — on a resident even `[]` is refused — and a Rejected record carries no `amends`, `supersedes` or `settlesOpenQuestion`. What each edge does to its target, and what anchors: `lifecycle-and-gates.md`. `amendedBy` and `supersededBy` are server-minted and refused on write.

You name every id on `alternatives`, `expectedOutcomes` and `openQuestions`: prefixed `alt-`, `eo-`, `oq-`, then only lowercase letters, digits, hyphens and underscores, unique in its list, required on every item, and named for what it is — `prose-and-raci.md` has the rule and the refusals. `chosenAlternative` and `deliberationFacts[].onAlternative` must name an alternative that exists once the patch lands.

Returns the record acknowledgement. An array you did not just draft starts from `get_MDR_details`.

### `transition_MDR_status`

`mdlGuid`, `mdr` as `MDRh7`, `status`. Admission to `Accepted` or `Rejected` sets `decidedAt`, assigns `mdrNumber`, and mints reciprocals on the records it names. Returns the record acknowledgement, which carries all three of `status`, `decidedAt` and `mdrNumber` to confirm the move by. Refused with `incomplete` → **repeat the identical call**, which completes it (`write-failures.md`).

### `uncommit_MDR`

`mdlGuid`, `mdr` as `MDRh7`, optional `reason`. Requires `admin` or `owner`, status `Accepted` or `Rejected`, and `retractable: true`. Demotes to staged, clearing `mdrNumber` and `decidedAt`. Returns the record acknowledgement. Refused with `incomplete` → **repeat the identical call**, which completes it (`write-failures.md`).

### `anchor_MDR`

| Param | Type | Required |
| --- | --- | --- |
| `mdlGuid` | string | yes |
| `mdr` | string: `MDRh7` | yes — must be a ledger resident |
| `kind` | `project` \| `other` | yes |

**Declares that something no typed link shows cites this record** (text outside the ledger, or prose on it), which stops an Uncommit releasing its number. Call it **before** writing the citation, not after — the **`memolok-citations`** skill carries when, and in what form the citation is written. Member-level, deliberately: gating it above the write it accompanies would leave the citation unanchored.

`project` is a citation in a project artifact — a source file, a document in the repository, a commit message, notes kept for later sessions. `other` is anywhere else one can go: an email, a chat message, a ticket, a slide, another entry's prose on the ledger. **Only the kind is recorded, never where the citation lives**, so there is no location parameter to look for and none to supply. Declaring the same kind twice is a no-op.

**Permanent, and unverified.** There is no withdrawal — that is what makes it worth anything — and the server cannot see your files or your mail, so nothing checks the claim. Say so if a user asks to undo one: it is corrected by a later record, not by a call.

Returns the record acknowledgement. A staged record has no number for anything to cite, so this refuses one.

### `admit_world_fact`

`mdlGuid`, `claimDescription`, optional `correctsFact`, `authoredBy`, `observedBy`. Use `correctsFact` only for a fact that was wrong when admitted — never for ordinary world drift. Returns `{ worldFactId, mdlGuid, createdAt, body }`.

`authoredBy` is whoever wrote the claim's words, typically you; `observedBy` whoever observed what it states. Neither is filled in for you, and any two of them and the admitting account may be one person. **Naming a person.** The user is `whoami`. Anyone else named in conversation: look up their given and family name in `get_MDL`'s `members` — one match → that `iri`; several → ask which; none → ask for something that identifies them, and write that. Unattended → an `iri` only when both names match exactly one member; otherwise the name as given.

### `record_observed_outcome`

| Param | Type | Required |
| --- | --- | --- |
| `mdlGuid` | string | yes |
| `mdr` | string: `MDRh7` or `MDR-7` | yes — a ledger resident |
| `claimDescription` | `{ markdown, lang? }` | yes |
| `discoveryType` | `Expected` \| `Emergent` \| `Deducible` | yes |
| `tests` | `{ outcomeId }` or an id string, and nothing else | Required when `Expected` |
| `testResult` | `Satisfied` \| `Violated` \| `Inconclusive` | Required when `Expected` |
| `observedBy`, `authoredBy`, `correctsFact` | string | no |

There is no `observedAt` parameter — the server stamps the current time, so a wake cannot be backdated. Recording one typically Anchors the source record. A visitor may read outcomes and may not record one. Returns `{ observedOutcomeId, mdlGuid, mdrHandle, mdrNumber, createdAt, observedAt, body }`.

## Scratchpad tools

Disposable working notes. See the **`manage-notes`** skill for the journeys.

A note is its author's. Members, admins and owners read every note; a visitor reads none and keeps none. Only the author replaces a note; the author, or an admin or owner, deletes one. To build on somebody else's, read it and create a note of your own.

Pass a `scratchpadId` to `hasContext`, `correctsFact`, `motivatedBy`, `tests.outcomeId` or `evidence` and the call fails telling you the value **is a scratchpad** — a category error, not a typo. Nothing in a ledger may reference one; if the material matters to a decision, admit it as a World Fact and cite that.

### `create_scratchpad`

| Param | Type | Required |
| --- | --- | --- |
| `mdlGuid` | string | yes |
| `description` | `{ markdown, lang? }` | yes |

Flat prose, like `register_matter` — **not** the nested `{ description: { markdown } }` shape that `ledgerIntent` and `verdict` take. The body is the entry; there is no second level.

Returns `{ scratchpadId, mdlGuid, createdAt, modifiedAt, body }`.

### `get_scratchpad`

`mdlGuid` + `scratchpadId`. The only tool that returns a full body. `createdBy` and each of `contributors` are `{ iri, name }`, `name` `null` where none is given to you; the note is the user's when `createdBy.iri` is `whoami`'s `iri`.

### `replace_scratchpad`

`mdlGuid`, `scratchpadId`, `description`. **Full replacement, not an append.** There is no partial update: read with `get_scratchpad`, compose the whole new body, send that. Advances `modifiedAt`. Only the note's author may call it, whatever anyone else's role; `createdBy` on `get_scratchpad` says who that is. Returns what `create_scratchpad` returns.

### `delete_scratchpad`

`mdlGuid` + `scratchpadId`. **The only delete tool in Memolok.** Immediate and final — no recovery window. The note's author, or an admin or owner. Returns `{ scratchpadId, mdlGuid, deleted: true }`.

### `discover_scratchpads`

Shared discovery params — they and the query grammar are in the discovery preamble above. Most recently touched first, which is what the `Touched:` line on each entry states. **Never carries a body**: `get_scratchpad` is the only read that does.

`query` is how you answer *"what did I save about X?"*; browsing a queue of pasted material answers nothing.

### `submit_feedback`

| Param | Type | Required |
| --- | --- | --- |
| `reports` | array of report objects | yes |
| `pluginVersion` | string | Send it always: the pack version, as for `get_guidance` |

**A batch always** — a single report is a batch of one, and one call means one preview-and-confirm. Up to 25 per call. Validation is all-or-nothing: a malformed third report leaves the first two unwritten, and the error names the offender by index (`reports[2].evidence[0]: …`).

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

`artifacts[].kind` is one of `skill`, `mcp_tool`, `mcp_server`, `plugin`, `model`. Artifacts name what was **in play**, not what is at fault — a skill contradicting a tool is two artifacts.

An **evidence item** is either a call record or a citation, and must carry a `response` or an `excerpt`:

| Field | Type | For |
| --- | --- | --- |
| `occurredAt` | ISO-8601 string | call record |
| `toolName` | string | call record |
| `request` / `response` | `{ markdown, lang? }` | call record |
| `requestId` | string | call record (`X-Memolok-Request-Id`) |
| `source` / `excerpt` | string / `{ markdown, lang? }` | citation |

There is no `outcome` field and no report-class field. A call that returned 200 and did the wrong thing is the case worth catching, and a success/error flag would file it under "success".

**Never send** `submittedBy`, `submittedAt`, `serverVersion`, `modelVersion` or `apiVersion` in a report — the servers record those and refuse a caller that sets them. The pack's own version goes in `pluginVersion`, beside `reports` and never inside one; `artifacts[].version` is for anything else you can actually read, such as a skill's.

Returns `{ reports: [{ feedbackId, title, kind, submittedAt }, …] }` in submission order — a digest, not the bodies. Keep the ids. Refused with `incomplete` → **never repeat it on your own**: some reports may be stored, and a repeat may store them twice. Tell the user.

### `get_feedback`

| Param | Type | Required |
| --- | --- | --- |
| `feedbackId` | string (`mfb_` + 16) | yes |

Your own reports only. A stranger's id returns *not found* rather than a permission error, and so does a report deleted during triage.

### `update_feedback`

| Param | Type | Required |
| --- | --- | --- |
| `feedbackId` | string (`mfb_` + 16) | yes |
| `patch` | object | yes |

Owner-only, no time limit. Patchable: `title`, `kind`, `report`, `userVerbatim`, `mdlGuid`, `artifacts`, `evidence`, `expectation`. **Arrays replace, they do not merge** — send the full list. `pluginVersion` is recorded at submission and cannot be patched. Returns `{ feedbackId, title, kind, modifiedAt, body }`; `get_feedback` shows what landed.

## Common errors

**A refusal names its code.** Every refusal passed on from Memolok reads as its sentence, then `Code: <code>.`, then `Reference: mrq_….` where there is one: `You are this Memolok Decision Ledger's last owner: … Code: last-owner.` Decide from the code, never from the sentence's words, and quote the sentence to the person. Messages the MCP server writes itself (an outage, an internal error, a refused sign-in, invalid tool arguments) carry no code: `write-failures.md`.

**A specific code decides what you do**, whatever the sentence says; the first table. **`invalid-request` and `not-found` are generic on purpose**: Memolok deliberately does not tell their cases apart by code. For those, and only those, read the sentence and correct the call; the second table explains each sentence.

| Code | Example | Cause |
| --- | --- | --- |
| `access-denied` | `You do not have the necessary permissions to … matters on this ledger.`, `… world facts …` | The role does not allow it: a visitor reads matters and world facts, and registers or admits neither |
| `access-denied` | `You do not have the necessary permissions to … scratchpads on this ledger.` | A visitor, who reads no notes and keeps none |
| `access-denied` | `You do not have the necessary permissions to replace a scratchpad you did not create. Read it and create your own instead.` | `replace_scratchpad` on somebody else's note, whatever the caller's role. Offer the copy |
| `access-denied` | `You do not have the necessary permissions to delete a scratchpad you did not create.` | A member deleting somebody else's note. An admin or owner may |
| `access-denied` | `You do not have the necessary permissions to … analyses on this ledger.` | The role does not allow it. Performing and attaching need a member, admin or owner; retracting and reopening need an admin or owner, even on an analysis the caller performed |
| `access-denied` | `You do not have the necessary permissions to create decision records on this ledger.` | `create_MDR` by someone whose role on the ledger does not let them create records, such as a visitor |
| `access-denied` | `You do not have the necessary permissions to edit this decision record.` | `update_MDR`, `transition_MDR_status` or `anchor_MDR` by someone whose role does not let them edit that record |
| `access-denied` | `You do not have the necessary permissions to record observed outcomes on this ledger.` | `record_observed_outcome` by someone whose role does not let them record, such as a visitor |
| `access-denied` | `You do not have the necessary permissions to uncommit decision records on this ledger.` | `uncommit_MDR` by someone whose role does not let them uncommit, such as a member |
| `access-denied` | `You do not have the necessary permissions to configure this ledger.` | `set_MDL_title` or `set_ledger_intent` by someone on the ledger whose role does not let them configure it |
| `create-ledger-closed` | `You do not have the necessary permissions to create ledgers.` | `create_MDL` by someone whose account is not allowed to create one. Every account is from the day it is created, so an administrator changes it; do not retry |
| `incomplete` | `This analysis was only partly created: …` | See `create_analysis` above: do not repeat it on your own |
| `incomplete` | `This batch of feedback reports was only partly written: …` | The service failed after sending the batch, so some reports may be stored. Tell the user; repeating may store them twice, which triage cleans up |
| `incomplete` | `This change to the Memolok Decision Record was only partly made: …` | `transition_MDR_status` or `uncommit_MDR`: repeat the identical call to complete it (`write-failures.md`). **The same code on `create_analysis` or `submit_feedback` means the opposite**, as the rows above say: the tool decides which |
| `change-pending` | `An earlier transition of this Memolok Decision Record to Accepted was only partly made. …` | Repeat the earlier call it names first, then this one |
| `record-changed` | `This Memolok Decision Record changed while the change was being made, and nothing was written. …` | Read the record again, and repeat only if the change still applies |
| `already-member` | `That account already holds a role on this Memolok Decision Ledger.` | `share_MDL` for someone holding a role there already; an invitation never changes a role |
| `holds-another-role` | `That account already holds another role on this Memolok Decision Ledger.` | `share_MDL` for a demo account holding a role other than `visitor`, which it keeps |
| `already-invited` | `An invitation to this Memolok Decision Ledger already waits for that account. …` | One of your invitations waits for them already; withdrawn on the portal |
| `unknown-account` | `No Memolok account has that IRI.` | The `iri` names no Memolok account, or one that expired. Ask them for theirs from `whoami` |
| `last-owner` | `You are this Memolok Decision Ledger's last owner: … send leaveWithNoOwner: true.` | `leave_MDL` or `request_MDL_vanish` by the last owner. Tell them the ledger is left with no owner; only on their yes, the same call with `leaveWithNoOwner: true` |
| `already-vanished` | `You vanished from this Memolok Decision Ledger already: …` | They vanished from it already, and nothing has named them there since |
| `mail-unavailable` | `The request was made, but its message could not be sent just now. …` | The request stands; its mail did not go. The same call again sends it |

| Message | Code | Cause |
| --- | --- | --- |
| `mdr must cite a record by its handle, as MDRh7: …` | `invalid-request` | A tool that changes a record was given `MDR-7`, or anything but `MDRh7`. Read the record and cite its `mdrHandle` |
| `mdr must cite a record by its number, as MDR-12, or by its handle, as MDRh7: …` | `invalid-request` | A bare integer, or a citation misspelt: the prefix is matched exactly, case included, and the digits carry no leading zero |
| `Memolok Decision Ledger not found.` | `not-found` | Does not distinguish a ledger the user cannot see from one that is not there. Report it as ambiguous: they may not be a member, or this address may be stale |
| `Memolok Decision Record not found.` | `not-found` | No record under that key on a ledger you can read. A ledger you cannot read answers `Memolok Decision Ledger not found.` instead |
| `Use transition_MDR_status to change Memolok Decision Record status.` | `invalid-request` | `status` in a patch |
| `Cannot change the {label} on a ledger-resident Memolok Decision Record ({status}).` | `invalid-request` | Tier-1 patch after t₀ |
| `Cannot change {field}: inter-record graph links may only be authored on staged Memolok Decision Records.` | `invalid-request` | Graph patch on a resident |
| `No updatable fields were provided.` | `invalid-request` | Empty or no-op patch |
| `Cannot transition a Memolok Decision Record from {from} to {to}.` | `invalid-request` | Illegal transition |
| `supersedes may only target Accepted residents (MDR-{n} is {status}).` | `invalid-request` | Bad supersession target. A `Superseded` one has already been retired, so a second attempt has nothing to retire |
| `amends may only target Accepted residents (MDR-{n} is {status}).` | `invalid-request` | Same rule, same reason: only an `Accepted` record has content for the amendment to leave standing |
| `A Rejected Memolok Decision Record must not carry amends (…).` | `invalid-request` | `amends` on a record admitting as `Rejected` — nothing it says is in force, so it changes nothing |
| `Field {x} cannot be authored over MCP: it is either minted by the server at admission or not yet writable.` | `invalid-request` | `amendedBy` / `supersededBy` on a patch |
| `This Memolok Decision Record is Anchored ({kind}) and cannot be Uncommitted.` | `invalid-request` | Uncommit on an anchored record. **The kind is named** — `project` or `other` for a declared anchor, `conflict pair` when the record is held only by a `conflictsWith` edge (which anchors both sides, with no way to dissolve it), otherwise `ledger` for a derived cause |
| `Only Accepted or Rejected Memolok Decision Records may be Uncommitted.` | `invalid-request` | Wrong status for uncommit |
| `Only a ledger-resident Memolok Decision Record can be Anchored. A staged record has no ledger number for anything to cite.` | `invalid-request` | `anchor_MDR` on a staged record — wait for admission |
| `kind must be one of project, other.` | `invalid-request` | `anchor_MDR` with anything else; the vocabulary is closed |
| `{field} names a user in the retired form https://www.memolok.ai/users/<user id>; …` | `invalid-request` | An agent written in a form Memolok no longer accepts. Write the user's `iri` from `whoami` |
| `{field} names no principal Memolok holds: {value}.` | `invalid-request` | A Memolok IRI naming no account Memolok holds, often an id misremembered. Take it from `whoami` or a record that carries it; nothing was written |
| `{field} is under Memolok's namespace but is no principal's IRI; …` | `invalid-request` | A value under `https://app.memolok.ai/m/` that is not a principal's `…/m/<musr_…>#id` or `…/m/<msvc_…>#id`. Correct the form, or write someone outside Memolok without that prefix |
| `discoveryType Expected requires tests referencing an expectedOutcome.` | `invalid-request` | Missing `tests` |
| `An analysis must take up at least one input; motivatedBy is empty.` | `invalid-request` | Empty `motivatedBy` |
| `motivatedBy[i] was not found in this Memolok Decision Ledger. An analysis input is a Matter, a World Fact or an Observed Outcome.` | `invalid-request`, `not-found` | An input the ledger does not hold, including another ledger's entry. The index is the position in your list; nothing was written. `hasContext` and `correctsFact` answer another ledger's entry the same way, worded for their field |
| `Analysis reference not found.` | `not-found` | Retracting a pair that is not attached, or no longer is |
| `This analysis produced MDR-{n}, which has been committed. Uncommit that record before reopening the reasoning it was committed on.` | `invalid-request` | `reopen_analysis` once a produced record is sealed |
| `{field} is a scratchpad id.` | `invalid-request` | A `msp_…` value passed to a reference field. Nothing may cite a note — admit a World Fact instead |
| `{field} is not a well-formed {kind} identifier. Pass it back exactly as Memolok returned it.` | `invalid-request` | The value is not an identifier of the kind the field takes: mistyped, truncated or edited. The sentence is the same whatever is wrong, so fetch the value again and pass it back unchanged |
| `{field} is a {kind} identifier, which does not address {kind}.` | `invalid-request` | A real identifier of another kind — a Matter where a World Fact belongs, or a ledger id passed to a feedback tool. Pass the kind the field takes |
| `Scratchpad not found.` | `not-found` | Missing note, wrong ledger, or already deleted |
| `A scratchpad body may be at most 65536 bytes;…` | `invalid-request` | Paste too large — split it, or keep a pointer to the source |
| `This Memolok Decision Ledger already holds {n} scratchpads, …` | `invalid-request` | The ledger's note limit. The user deletes notes of theirs they no longer need; an admin or owner can delete anyone's |
| `claimDescription is required when analysis produces a Memolok Decision Record.` | `invalid-request` | Path A without a claim |
| `A Memolok Decision Record cannot cite its own Observed Outcome in hasContext...` | `invalid-request` | DTP violation |
| `Feedback report not found.` | `not-found` | Not yours, wrong id, or deleted during triage |
| `reports[n] may not set serverVersion, …` | `invalid-request` | The server records the build; a caller cannot claim one |
| `reports[n].evidence[m] must carry either a response … or an excerpt` | `invalid-request` | An evidence item that asserts nothing |
