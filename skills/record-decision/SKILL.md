---
name: record-decision
description: >-
  Record a decision on a Memolok ledger — work out whether you have a raw symptom or a sharpened
  need, capture the alternatives and reasoning behind the choice, and persist it as a Memolok
  Decision Record (MDR). Use when the user makes or describes a consequential choice, reports a
  problem they intend to act on, says "record this" or "log this decision", proposes a solution like
  "we should switch to X", works through options they want kept, or acts on something the ledger
  already holds — a world fact or a recorded outcome that now calls for a decision.
argument-hint: "<the decision, problem, or choice>"
---

# /memolok:record-decision — Record a decision

Produces a Memolok Decision Record persisted at **Deliberating**, with the head Claim, the options the
user actually weighed, the reasoning, a Verdict, expected outcomes, and any acknowledged deferrals.

## Usage

```
/memolok:record-decision <what was decided, or the problem behind it>
```

Examples:

- `/memolok:record-decision we're going with Postgres over DynamoDB for the events table`
- `/memolok:record-decision users keep complaining that the export times out`
- `/memolok:record-decision picked a 4-drive RAID-Z1 array for the media server`
- `/memolok:record-decision I need to choose a language for the stats pipeline`

## Step 0 — Load the method

Load the **`memolok-method`** skill before any write. It carries the fish model, the Decision
Transaction Principle, the lifecycle gates, and facilitation Rules A–G.

## Non-negotiables

- Never invent `mdlGuid`, `mdrHandle`, or `mdrNumber` — the server mints them
- All prose is `{ "markdown": "...", "lang": "en" }`
- Default persist status is **Deliberating**; t₀ only on the user's explicit request (Rule C)
- Say "captured" only after a write succeeded — before that, say *drafting* or *got it*
- Cite `MDR-{n}` when admitted, `MDRh{handle}` or a head Claim paraphrase while staged; never a bare handle (Rule F)
- Check `retractable` before proposing an uncommit
- Writing a record number outside this session → anchor it first (`references/citing-records.md`)

## Workflow

### 1. Establish the ledger

Use the `mdlGuid` already in play. If there is none, check `.memolok/mdl.yml` in the project when
files are available, then fall back to `get_MDLs`.

Then `get_MDL(mdlGuid)` — the ledger's stated purpose, when it has one, tells you what this ledger is
usually about, which helps both the intake fork below and sharpening the Claim in the user's own domain
language rather than generic phrasing. It is background, never a test: a decision that looks unrelated
to the stated purpose is recorded exactly like any other, with no comment and no request to justify it.

If the user has no ledger, agree a title and call `create_MDL`. Draft a purpose from what they have
already told you and offer it in one line — *"I'll note this ledger is for the storage rebuild"* — and
take a "skip" without asking twice. They came here to record a decision; never delay it for this.

If there may be parked bait, `discover_matters(untaken: true)` shows what nobody has picked up, with
enough of each to tell whether it is the same thing. If one matches what the user just raised, take
that matter up rather than registering a duplicate — and if several do, take up all of them in one
analysis.

The same goes for the almanac. If what prompted this is a fact or an outcome already admitted —
`list_world_facts`, `list_observed_outcomes` — take that entry up **as itself**. There is no inbox
for the almanac and no "unused" state to scan for, so this is a lookup you do when the conversation
points at one, not a sweep.

### 2. Sort the intake

**This is the step that most often goes wrong.** Do not skip it, and do not let the phrasing of the
user's opening line decide it for you.

| What the user brought | It is | Route |
| --- | --- | --- |
| A symptom, in someone's words, no target | **bait** | Step 3a |
| A target a future observation could settle | **head Claim** | Step 3b |
| A mechanism with the need amputated | **a Verdict with no Need** | Back the need out, then re-sort |
| A premise true regardless of any choice | **World Fact** | Admit it via `manage-almanac` first |
| An observation on an already-sealed record | **wake** | Record it via `record-outcome` first |

The last two rows are a **detour, not an exit.** Admit the fact or record the outcome, then come
back: if it calls for a decision, that entry is the analysis input — pass its id in `motivatedBy`.
Do **not** register a Matter restating it. That records neither the entry as the input nor that
anything connected the two, and Matters are immutable, so it cannot be repaired afterwards.

**The discriminator is not who is speaking — it is whether a future observation could settle it.**
*"P99 auth latency must stay under 50ms"* is a Claim. *"Login takes forever"* is bait, even when the
user saying it is the only engineer on the project. Being solo does not convert a symptom into a need.

**When it is ambiguous, ask — one question, then proceed.** Something like *"is that your own read of
what needs to be true, or is it what someone reported was wrong?"* Guessing is recoverable here: a
matter you registered can be taken up later, and an analysis can take up an input of any kind it did
not start with. What you cannot recover is the raiser's wording once you have sharpened it away.

**If the user only wants to park it** — they noticed a problem but do not want to leave what they are
doing — that is the `save-matter` skill. Do not drag them through a decision they did not ask for.

Worked examples of ambiguous intakes: `references/intake-fork.md`.

### 3a. Bait branch

1. `register_matter` with the stakeholder's words **verbatim**. Do not sharpen, summarize, or
   translate into a target — the raw input is the point, and matters are immutable. Skip this step
   entirely when the input is an already-admitted world fact or observed outcome: it is recorded, it
   has an id, and it goes into step 3 as itself.
2. Sharpen the need **with the user**, not for them. The head Claim is a falsifiable objective, and it
   must not name the mechanism you are about to choose (Rule G).
3. `create_analysis` with `producesDecision: true` and the sharpened `claimDescription`. Pass every
   input this reasoning took up in `motivatedBy` — not just the one that prompted the conversation,
   and in whatever mix of matters, world facts and observed outcomes actually applies. This mints
   the record at **New** and concludes the analysis in the same call.
4. `transition_MDR_status` to **Deliberating**, then continue at step 4.

If sharpening shows there is no decision to make, that is **Path B**: `create_analysis` with
`producesDecision: false` and a rationale explaining why. Confirm with the user first. No record is
minted, and that is an honest outcome, not a failure.

Payloads: `references/matter-payloads.md`. How a matter comes to rest, and what is not yet
recordable: `references/matter-closure.md`.

### 3b. Expert branch

`create_MDR` with `claimDescription` and `status: "Deliberating"` — the default, even when you already
have most of the body. Reserve `Accepted` or `Rejected` at creation for the narrow case in Rule C
where the user has already stated commitment in their own words.

Payloads: `references/expert-payloads.md`.

### 4. Deliberate

**Draft from what the user already said before asking anything** (Rule E). Mine the conversation for
the favoured path, discarded options, "versus doing nothing", and constraints that ruled things out.

- `alternatives` — `label` is a short handle, `description` is the option's substance
- `deliberationFacts` — arguments the user actually made, each paired with `onAlternative`

Do not invent options to make the belly look fuller. One short honesty check is fine when a single
path has no acknowledgment of what else was in play; it is not a gate, and it is not a brainstorm.

### 5. Converge

Draft the `verdict` and set `chosenAlternative`. Add `expectedOutcomes` — measurable tail commitments
covering gains, costs, risks, and dependencies. These are required before a future Accept, and they
are what the wake will later be tested against.

Drafting all of this while the record sits at **Deliberating** is normal pre-commit work.

**Converging on a *no* is converging.** If the Verdict is turning into an explanation of what the user
will **not** do, this is a **Rejected** record and `chosenAlternative` should stay unset. That holds
even when other work is proceeding perfectly well — a Claim can be declined while something else
ships, and that pair is two records, not one. The full set of situations, and the shaping rules the
validators do not enforce, are in the `memolok-method` skill.

#### Building against a staged record

Where the decision is about something you can build, a spike is deliberation and belongs in the belly.
The loop: draft to **Deliberating**, build against it, fold what building taught back into the record,
*then* seal. Nothing about a staged record is fixed, so this costs only the patch.

It is worth the round trip because a Verdict can name a mechanism the implementation then contradicts
— and after t₀ that is a successor, not an edit. Finding it while staged costs one `update_MDR`.

When building shows the draft was wrong, record the correction as a deliberation fact **if the wrong
turn teaches something** — if it shows why the obvious approach fails. Otherwise just edit the draft;
staged records are freely editable and most corrections are not interesting.

### 6. Name the open questions

Record what this decision deliberately leaves unsettled as `openQuestions`. Prefer logging an
out-of-scope item and moving on over expanding the record to resolve it (Rule D). They never block
anything.

### 7. Preview and persist

Show the fish back — head, alternatives, deliberation, verdict, expected outcomes, open questions,
intended status — and ask whether it covers what was discussed. Wait for a yes.

Then `update_MDR` with the body. Format: `references/fish-preview.md`. Payloads:
`references/patch-payloads.md`.

Confirming the preview is **not** t₀. Do not offer **Proposed** on informal work, and do not offer
**Accepted** unless the user has raised sealing themselves.

### 8. Offer next steps

- The user wants to seal it → **`commit-decision`**
- They want to check what landed → **`review-ledger`**
- Another decision surfaced → start again at step 2 for that one, as its own record

If deliberation revealed a **second distinct decision**, open a separate record for it rather than
folding both into one body. A record that converges twice is malformed.

The commonest instance is a **declined Claim hiding inside an Accept** — the record does one thing and
spends a paragraph explaining what it is not doing. That second decision is easy to miss precisely
because a rejection may not be in mind as something you can produce, so it gets written as prose
instead of as a record.

## Field quality

**Head Claim.** Bad: *"we need better performance."* Good: *"P99 API latency must stay under 200ms at
500 concurrent users."*

**Sharpen is not resolve.** When the honest need is mechanism-agnostic, keep it that way:

- Over-sharpened: *"the project must use Python for its mature pandas/numpy/scipy stack."*
- Correctly sharp: *"the chosen language must have a mature, well-supported statistical-analysis
  library ecosystem."*

The second is still falsifiable — a later observation can confirm or refute whether the ecosystem held
up — without pre-committing to which language. The language choice is Verdict-shaped, and belongs in
`alternatives` and `verdict`.

Full worked correction arc: `references/need-vs-verdict-drift.md`.

## Tips

- A complete fish body may sit at **Deliberating** indefinitely. Completeness is not commitment.
- Patches are incremental — send only the keys that changed.
- Read the outcome ids (`eo-…`) back from the response; a later wake needs them.
- If the user's need turns out to already be covered by an existing record, say so and offer
  `review-ledger` instead of minting a duplicate.

## References

| File | Load when |
| --- | --- |
| `references/intake-fork.md` | Step 2 is ambiguous — sharpening bait away destroys the raiser's wording, and that is the part no later call can reconstruct |
| `references/matter-payloads.md` | Before `register_matter` — the verbatim rule has edges, and a sharpened matter cannot be un-sharpened |
| `references/expert-payloads.md` | Before your first `create_MDR` — it carries the two shape asymmetries that fail the call outright, and the allowed creation statuses |
| `references/patch-payloads.md` | Before `update_MDR` — patches replace whole arrays rather than merging into them |
| `references/fish-preview.md` | Presenting the recap at step 7, if you want the shape that reads back cleanly |
| `references/need-vs-verdict-drift.md` | The Need may have absorbed the answer — the conflation survives light rewording, so spotting it needs the worked arc |
| `references/matter-closure.md` | Any analysis touching more than one input, or an input surfacing after the analysis concluded — passing one `motivatedBy` where several apply records reasoning that did not happen, and three of the five ways a matter comes to rest have no tool yet |
| `references/citing-records.md` | Before writing a record number into a file, a document or a message — the citation must be anchored *first*, and an unanchored number can come to name a different decision |
