---
name: record-outcome
description: >-
  Record what actually happened after a decision, and compare it against what was promised at
  commitment. Use when the user reports a result, a target that was met or missed, an unexpected side
  effect of an earlier choice, or asks how prior decisions actually turned out — "that didn't work",
  "we hit the number", "turns out nobody used it".
argument-hint: "<what happened>"
---

# /memolok:record-outcome — Record the wake

Registers an Observed Outcome against a decision already on the ledger, and reads the gap between what
was promised at t₀ and what the world did.

## Usage

```
/memolok:record-outcome <what happened>
```

Examples:

- `/memolok:record-outcome the cache landed and P99 is at 140ms`
- `/memolok:record-outcome we missed the latency target, still around 400ms`
- `/memolok:record-outcome nobody has used the export feature since launch`
- `/memolok:record-outcome how did the storage decision actually turn out?`

## Step 0 — Load the method

Load the **`memolok-method`** skill. The Decision Transaction Principle is what makes the wake a
separate record instead of an edit.

## Non-negotiables

- Never invent `mdlGuid`, `mdrHandle`, or `observedOutcomeId` — the server mints them
- All prose is `{ "markdown": "...", "lang": "en" }`
- **Never rewrite the tail to match what happened** — that is ledger fraud
- The source record must be a ledger resident; staged records have made no bets
- Say "captured" only after the write succeeded
- Recording a wake usually **anchors** the source record
- Judge the outcome against that record's expected outcomes — never against the ledger's stated purpose

## The core rule

The decision froze at t₀. The wake is what the world did afterwards, and it is a **separate record**.

If a decision promised P99 under 200ms and the result was 400ms, you record an outcome saying 400ms.
You do **not** edit the expected outcome to say 400ms. The gap between the two is the entire value —
erasing it destroys the only evidence of how good the team's forecasting is.

## Workflow

### 1. Find the source record

`list_MDRs(query="...")` in the words the user used, or `get_MDR` if the record is known. Rows carry
an excerpt of the head **Claim**, not the whole of it, so read the record before deciding it is the
right one. It must be **Accepted**, **Rejected**, or **Superseded** — a wake cannot attach to a
staged record.

### 2. Decide the discovery type

| Type | Meaning |
| --- | --- |
| **Expected** | Assesses a specific tail commitment the record made. Requires `tests` and `testResult` |
| **Deducible** | Could have been foreseen at t₀ but was not written down |
| **Emergent** | Nobody could reasonably have predicted it |

For **Expected**, read the outcome id (`eo-…`) from `get_MDR` — you need it for `tests`.

The split between Deducible and Emergent is an honest judgement, and it matters: a pattern of
Deducible outcomes says the team's forecasting has a blind spot, while Emergent ones are just the world
being the world. Do not label something Emergent to spare anyone's feelings.

### 3. Apply the observation test

> Could this outcome exist without a human or an agent making an epistemic judgement?

If yes, it is telemetry, not an observed outcome. *"CPU hit 91% at 14:03"* is a metric. *"The cache
did not hold P99 under target during peak"* is an observation — someone assessed it.

The ledger is not a monitoring system. Do not funnel raw metrics into it.

### 4. Record it

```json
{
  "mdlGuid": "<mdlGuid>",
  "mdrHandle": 1,
  "claimDescription": {
    "markdown": "P99 read latency measured 410ms during the December peak, against a 200ms target.",
    "lang": "en"
  },
  "discoveryType": "Expected",
  "tests": { "outcomeId": "eo-3f9a2c11" },
  "testResult": "Violated"
}
```

`testResult` is `Satisfied`, `Violated`, or `Inconclusive` — and it belongs to the **expectation** being
assessed, not to the observation event.

Payloads and error cases: `references/wake.md`.

### 5. Read the delta

```
get_MDR_learning_delta(mdlGuid, mdrHandle)
```

Returns each expected outcome with the wakes that tested it, plus unmatched Emergent and Deducible
outcomes. Coexisting assessments all come back — there is no single current result, by design, because
two honest observations can disagree.

### 6. Offer what follows

A `Violated` result is **evidence for re-evaluation, not an automatic supersession**. What it earns is
a fresh look, at a new t₀.

- The user wants to act on it → **`record-decision`**; the wake becomes the input for the next
  fish. Hand over the `observedOutcomeId`: the next analysis takes the wake up by that id, and
  must not restate it as a Matter
- They just want it noted → stop; recording it was the point
- They want to change the original record → **`revise-decision`**, and expect it to be anchored now

**The successor is often a Rejection, and that is easy to miss.** A `Violated` result frequently means
the record's *promise* was wrong rather than the world — the decision shipped and stands, and what
needs recording is that the organization declines the commitment it made. That is a **Rejected**
record, even while the work it relates to proceeds untouched.

It cannot supersede the original: a Rejection carries no `supersedes`, so the two stand side by side
and the prose has to do the linking. Cite this wake in the successor's `hasContext` — that is how a
later reader gets from the violated commitment to the decision about it. The `memolok-method` skill
carries the shape.

## Conflicting observations

Mint a new outcome. Never edit a prior one.

Correct an earlier observation only when it was **wrong at the time it was made** — a mis-read
instrument, a bad query — using `correctsFact`. The world simply moving on is not a correction; that is
a new observation that coexists with the old one.

## Anchoring

Recording a wake sets `realizedFrom` on the source, which typically makes it **anchored**
(`retractable: false`). After that it can only be changed by a successor carrying `supersedes`.

If the user may still want to revise the sealed record, do that **first** — see `revise-decision`.

## Tips

- There is no `observedAt` parameter. The server stamps the current time, so wakes cannot be backdated.
  If the observation is old, say so in the prose.
- A record can carry many wakes over time. That is the point of a long tail.
- Long latency is not failure; **unmeasured** latency is. A commitment with no observations after a
  year is worth surfacing.
- A `Rejected` record can carry a wake too — what happened after deciding *not* to act is just as
  informative.
- `list_observed_outcomes(mdlGuid, mdrHandle)` shows everything already recorded against a record.

## References

| File | Load when |
| --- | --- |
| `references/wake.md` | Before recording a wake — `Expected` needs two fields the others must not carry, and `correctsFact` is the one edge you cannot take back |
