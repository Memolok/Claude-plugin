---
name: record-decision
description: >-
  Record a decision on a Memolok ledger — work out whether you have a raw symptom or a sharpened
  need, capture the alternatives and reasoning behind the choice, and persist it as a Memolok
  Decision Record (MDR). Use when the user makes or describes a consequential choice, reports a
  problem they intend to act on, says "record this" or "log this decision", proposes a solution like
  "we should switch to X", or works through options they want kept.
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
- Cite `mdrNumber` when admitted, head Claim paraphrase while staged; never volunteer `mdrHandle` (Rule F)
- Check `retractable` before proposing an uncommit

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

If there may be parked bait, `list_matters(status="MatterReceived")` shows it. If one matches
what the user just raised, use that matter rather than registering a duplicate.

### 2. Sort the intake

**This is the step that most often goes wrong.** Do not skip it, and do not let the phrasing of the
user's opening line decide it for you.

| What the user brought | It is | Route |
| --- | --- | --- |
| A symptom, in someone's words, no target | **bait** | Step 3a |
| A target a future observation could settle | **head Claim** | Step 3b |
| A mechanism with the need amputated | **a Verdict with no Need** | Back the need out, then re-sort |
| A premise true regardless of any choice | **World Fact** | Hand off to `manage-almanac` |
| An observation on an already-sealed record | **wake** | Hand off to `record-outcome` |

**The discriminator is not who is speaking — it is whether a future observation could settle it.**
*"P99 auth latency must stay under 50ms"* is a Claim. *"Login takes forever"* is bait, even when the
user saying it is the only engineer on the project. Being solo does not convert a symptom into a need.

**When it is ambiguous, ask — one question, then proceed.** Something like *"is that your own read of
what needs to be true, or is it what someone reported was wrong?"* Do not guess: a record minted on
the expert path can never be given matter provenance afterwards, because `promptedBy` is
mint-frozen.

**If the user only wants to park it** — they noticed a problem but do not want to leave what they are
doing — that is the `save-matter` skill. Do not drag them through a decision they did not ask for.

Worked examples of ambiguous intakes: `references/intake-fork.md`.

### 3a. Bait branch

1. `register_matter` with the stakeholder's words **verbatim**. Do not sharpen, summarize, or
   translate into a target — the raw input is the point, and matters are immutable.
2. Sharpen the need **with the user**, not for them. The head Claim is a falsifiable objective, and it
   must not name the mechanism you are about to choose (Rule G).
3. `create_analysis` with `producesDecision: true` and the sharpened `claimDescription`. This mints
   the record at **New** with `promptedBy` set.
4. `transition_MDR_status` to **Deliberating**, then continue at step 4.

If sharpening shows there is no decision to make, that is **Path B**: `create_analysis` with
`producesDecision: false` and a rationale explaining why. Confirm with the user first. No record is
minted, and that is an honest outcome, not a failure.

Payloads: `references/matter-payloads.md`. Terminal dispositions:
`references/matter-dispositions.md`.

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
- **Rejected** is a first-class outcome. When deliberation shows no viable path, offer it as readily
  as Accept.
- If the user's need turns out to already be covered by an existing record, say so and offer
  `review-ledger` instead of minting a duplicate.

## References

| File | Load when |
| --- | --- |
| `references/intake-fork.md` | Step 2 is ambiguous, or the user pushed back on your routing |
| `references/matter-payloads.md` | Running the bait branch |
| `references/expert-payloads.md` | Running the expert branch |
| `references/patch-payloads.md` | Building the `update_MDR` body |
| `references/fish-preview.md` | Presenting the recap at step 7 |
| `references/need-vs-verdict-drift.md` | The Need may have absorbed the answer |
| `references/matter-dispositions.md` | The user asks how a matter ends other than Path A or B |
