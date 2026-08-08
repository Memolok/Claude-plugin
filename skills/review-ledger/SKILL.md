---
name: review-ledger
description: >-
  Read back what a Memolok ledger already holds — decisions, their status, unsettled deferrals, and
  parked matters nobody has picked up. Use when the user asks "what did we decide about X", wants
  a scan of records by status, asks which questions were left open, wants to know what is waiting to
  be looked at, or needs a record's status before changing it.
argument-hint: "[status | topic]"
---

# /memolok:review-ledger — Read the ledger

Read-only. Surfaces what has been decided, what is still in flight, what was deliberately deferred,
and what pain is sitting unprocessed.

## Usage

```
/memolok:review-ledger [status or topic]
```

Examples:

- `/memolok:review-ledger what did we decide about storage?`
- `/memolok:review-ledger show me everything still in Deliberating`
- `/memolok:review-ledger what open questions are outstanding?`
- `/memolok:review-ledger anything parked I haven't looked at?`

## Step 0 — Load the method

Load the **`memolok-method`** skill for the lifecycle vocabulary and Rule F, which governs how records
are named back to the user.

## How to name records

| State | Cite it as |
| --- | --- |
| Admitted (`mdrNumber` set) | **MDR-7** |
| Staged (`mdrNumber` is `null`) | The head Claim, or a short paraphrase |

Never volunteer `mdrHandle`. Cite it freely when the user asks for it.

## Workflow

### 1. Establish the ledger

Use the `mdlGuid` in play, else `.memolok/mdl.yml`, else `get_MDLs`. If the user belongs to several
and has not said which, ask — reading the wrong ledger produces a confidently wrong "we never decided
that".

### 2. Pick the read

| The user wants | Call |
| --- | --- |
| What this ledger is *for* | `get_MDL(mdlGuid)` |
| Everything, or a status slice | `list_MDRs(mdlGuid, status?)` |
| One record in full | `get_MDR(mdlGuid, mdrHandle)` |
| One record the user named — "what did MDR-7 say?" | `get_MDR(mdlGuid, mdrNumber=7)` |
| Unprocessed intake | `list_matters(mdlGuid, status="MatterReceived")` |
| One matter | `get_matter(mdlGuid, matterId)` |
| What came of a matter | `get_matter`, then `get_MDR(mdrHandle)` |
| Why a matter was dismissed | `get_analysis(mdlGuid, analysisId)` |
| Premises the ledger reasons from | `list_world_facts(mdlGuid)` |
| What happened after a decision | `list_observed_outcomes(mdlGuid, mdrHandle?)` |
| Promises versus reality for one record | `get_MDR_learning_delta(mdlGuid, mdrHandle)` |

`list_MDRs` returns `headClaimMarkdown` on each row, so a topic search usually means listing and
matching on that rather than fetching every record.

**When the user names a number, read it directly.** `get_MDR` takes `mdrNumber` in place of
`mdrHandle` — exactly one of the two — so "what did MDR-7 say?" is one call, not a full listing to
find a handle. Use the handle whenever you already have one, and use the handle the response carries
for anything you do to the record afterwards. A number released by an uncommit can be taken by a
later record, so read back what you got before answering on it.

**The `status` filter on `list_MDRs` is not validated.** A typo returns an empty list, which reads as
"nothing decided". Use exact status names, and treat a surprising empty result as suspect.

### 3. Answer the question that was asked

Lead with the answer, then the evidence.

> **Yes — MDR-7, Accepted.** You went with a single drive plus scheduled off-device backup, on the
> grounds that RAID doesn't protect against the failure modes you actually cared about (fire, theft,
> deletion).
>
> Still open on that record: the backup destination and schedule.

Not a table dump the user has to read for themselves.

### 4. Offer the next move

- They want to change something admitted → **`revise-decision`**
- The ledger's stated purpose is stale or missing → **`revise-intent`**
- They want to seal something staged → **`commit-decision`**
- Something is genuinely undecided → **`record-decision`**
- Parked bait is worth picking up → **`record-decision`**, which sharpens it

## Common reviews

### What is in flight

`list_MDRs(status="Deliberating")` — records with a head Claim and no commitment yet. Some are live
work; some are abandoned drafts that were never going anywhere. Both are legitimate; a staged record
carries no obligation.

### What was left open

There is no query for unsettled deferrals. Read the records and collect `openQuestions` where
`settledIn` is absent.

Say plainly that you assembled this by reading, not that the ledger surfaced it — Memolok has no
open-question registry yet.

### What the ledger says it is for

`get_MDL(mdlGuid)` returns the ledger's stated purpose alongside its title and your role. Read it back
as what it is — the ledger's own account of itself, written by a practitioner and revisable at any
time.

It is **not** evidence of what was decided. If someone asks what was decided about X, the answer comes
from the records, never from the purpose. And a record that looks unrelated to the stated purpose is
not an anomaly worth flagging: intent is revised in place and often lags the work.

`ledgerIntent: null` means nobody has written one. Say so plainly and offer **`revise-intent`**.

### What came of a parked matter

`get_matter` answers "did anything ever happen about X?" without a ledger-wide scan:

| Response | Means |
| --- | --- |
| `analysisId: null` | Never analyzed — still raw bait |
| `analysisId` set, `producesDecision: []` | Analyzed and dismissed |
| `producesDecision: [{ mdrHandle }]` | Read the record with `get_MDR(mdrHandle)` |

Answer with what the record decided, not the matter's status. Call `get_analysis` only when asked
*why* — fetching it per matter turns one review into a dozen calls.

### The unprocessed inbox

`list_matters(status="MatterReceived")` returns everything registered but never analyzed —
mostly matters parked through **`save-matter`** during other work.

Present them as raw signal, in the words they were logged in. Do not sharpen them into needs while
summarizing; that is the pickup session's job, with the user present.

### Scratchpads are not ledger contents

A review covers decisions, deferrals and unprocessed bait. **Scratchpads are none of those** — they
carry no assertion, no disposition and no expectation, so they never belong in a sweep.

Do not list them alongside matters as though they were waiting for something, and never surface one
as stale, untouched or needing attention. **A note that has sat for a year is behaving correctly.**
There is no review signal here to compute and none to invent.

If the user asks about their notes specifically, that is the **`manage-notes`** journey — reach for
`search_scratchpads`, not a review.

### Before changing a record

`get_MDR` and read two fields:

| Field | Means |
| --- | --- |
| `mdrNumber` | `null` → staged, patch it freely |
| `retractable` | `true` → committed but uncommittable; `false` → anchored, needs a successor |

This read is what decides between the two paths in `revise-decision`. Do not guess it.

### How a decision actually turned out

`get_MDR_learning_delta` joins expected outcomes against the wakes that tested them. Coexisting
assessments all come back — there is no single current result, by design.

Ledger residents only; a staged record has made no bets yet.

## What the ledger cannot tell you

> **Not built yet.** No tool exists for impact analysis, decision-decay or polluted-premise detection,
> stale-constraint review, cross-record open-question queries, or portfolio-wide outcome latency.

You can often *reason* toward these by reading records — that is fine, and useful. What is not fine is
implying the ledger computed it. Say "reading through these, three records lean on that assumption"
rather than "the ledger flags three affected records".

## Tips

- Non-members get `Memolok Decision Ledger not found.` on reads — deliberately identical to a missing
  ledger, so it leaks nothing. If the user expected access, they need to be added by an administrator.
- `visitor` role can read everything and write nothing.
- A `Superseded` record is history, not a mistake. It records what was true and in force at its own t₀.
- When a search finds nothing, say so plainly and offer to record the decision now rather than
  speculating about what might have been decided elsewhere.
