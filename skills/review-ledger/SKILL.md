---
name: review-ledger
description: >-
  Read back what a Memolok ledger already holds — decisions, their status, unsettled deferrals, and
  parked matters nobody has picked up. Use when the user asks "what did we decide about X", wants
  a scan of records by status, asks which questions were left open, wants to know what is waiting to
  be looked at, or needs a record's status before changing it. Also use before acting on any claim
  that the ledger is missing something — importing material, checking whether a decision is already
  recorded, working out what is still worth recording — including when the ledger is one input
  among several in a larger task.
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
| Staged (`mdrNumber` is `null`) | **MDRh7**, or the head Claim where prose reads better |

**The two are different addresses**, and `MDRh7` and `MDR-7` routinely name different records. Never
write a bare handle — it reads as a number. Raw handles stay in tool arguments.

## Workflow

### 1. Establish the ledger

Use the `mdlGuid` in play, else `.memolok/mdl.yml`, else `get_MDLs`. If the user belongs to several
and has not said which, ask — reading the wrong ledger produces a confidently wrong "we never decided
that".

### 2. Hand the sweep off, if you can

**The ledger must be settled first.** That is why this step is here and not earlier: picking the
ledger can need the user, and a scout cannot ask anyone anything.

Ledger reads are delegated — the **reading invariant** in **`memolok-method`**. Read inline instead
and the pages land in the practitioner's own context, where they cannot be taken back out.

Almost every read on this journey is a sweep, so the useful half of the rule here is its floor. **The
reading invariant, applied — what stays inline:**

| The read | Why it stays |
| --- | --- |
| The user named a number — `get_MDR(mdrNumber=7)` | One call; spawning costs more |
| `get_MDL` for the ledger's purpose | One call, and the answer is the whole of it |
| `retractable` before a revision | Hold this first-hand, not on report |
| One record in full, handle already known | Bounded before you start |

Everything else on this journey — a topic search, the open-question sweep, an inbox past one page, a
status slice past one page, chasing a parked matter through to its records — is unbounded, and goes
to the scout.

**Handing off.** The scout has none of this conversation. Give it the `mdlGuid` explicitly, the
user's question in their own words, and what you need back. Never hand it a ledger the user has not
settled on.

**Getting it back.** Name records to the user per Rule F and keep raw handles in tool arguments. The
scout reports its coverage — pages read out of `total`, bodies opened — and **that coverage is
part of your answer**: "nothing about X in all 68 records" and "nothing about X in the first 25" are
different answers, and only one of them settles anything.

**If you cannot spawn one**, do the same reads inline. They are paged now, so this is ordinary work
rather than a degraded path. Per the reading invariant: no agent tool is not the user's problem and
goes unmentioned; a session whose own settings forbid spawning is theirs to change, so say it once.

### 3. Pick the read

| The user wants | Call |
| --- | --- |
| What this ledger is *for* | `get_MDL(mdlGuid)` |
| Everything, or a status slice | `list_MDRs(mdlGuid, status?)` |
| One record in full | `get_MDR(mdlGuid, mdrHandle)` |
| One record the user named — "what did MDR-7 say?" | `get_MDR(mdlGuid, mdrNumber=7)` |
| Unprocessed intake | `list_matters(mdlGuid, untaken: true)` |
| One matter | `get_matter(mdlGuid, matterId)` |
| What came of a matter | `get_matter`, then `get_MDR(mdrHandle)` |
| Why a matter was dismissed | `get_analysis(mdlGuid, analysisId)` |
| What a record's reasoning took up | `get_MDR` for `analysisId`, then `get_analysis` |
| Premises the ledger reasons from | `list_world_facts(mdlGuid)` |
| What happened after a decision | `list_observed_outcomes(mdlGuid, mdrHandle?)` |
| Promises versus reality for one record | `get_MDR_learning_delta(mdlGuid, mdrHandle)` |

**A topic search is `query`, not a listing you read yourself.** `list_MDRs(query="...")` searches the
whole fish — head **Claim**, **Verdict**, alternatives, deliberation facts, expected outcomes, open
questions — so a record is found by the reasoning inside it and not only by its Claim. Rows carry an
excerpt of the Claim rather than the whole of it, so `get_MDR` before quoting one back.

Search is lexical: terms are ORed and case-insensitive, there are no phrases or operators, and a
record that discusses the topic in other words will not surface. **An empty result means no record
used those words** — say which you tried, and treat it as weak evidence rather than as "we never
decided that".

Every listing pages: `limit` defaults to 25, and `total` counts the whole match. Read `total` before
answering, and never let a first page stand in for the ledger.

**Under a `query`, `total` counts matches — not the ledger.** A size question needs an unfiltered
call. A filtered `total` quoted as a ledger size is wrong in the direction that makes the almanac look
emptier than it is, which is how "I did not find it" becomes "it was never recorded".

**When the user names a number, read it directly.** `get_MDR` takes `mdrNumber` in place of
`mdrHandle` — exactly one of the two — so "what did MDR-7 say?" is one call, not a full listing to
find a handle. Use the handle whenever you already have one, and use the handle the response carries
for anything you do to the record afterwards. A number released by an uncommit can be taken by a
later record, so read back what you got before answering on it.

**The `status` filter is validated now.** An unknown value is refused by name rather than answering
with an empty list that read as "nothing decided". A genuinely empty result therefore means what it
says — though only for the page you asked for, so check `total` before concluding anything from it.

### 4. Answer the question that was asked

Lead with the answer, then the evidence.

> **Yes — MDR-7, Accepted.** You went with a single drive plus scheduled off-device backup, on the
> grounds that RAID doesn't protect against the failure modes you actually cared about (fire, theft,
> deletion).
>
> Still open on that record: the backup destination and schedule.

Not a table dump the user has to read for themselves.

### 5. Offer the next move

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

`get_matter` answers "did anything ever happen about X?" without a ledger-wide scan. Read `takenUpBy`:

| Response | Means |
| --- | --- |
| `takenUpBy: []` | Nobody has picked it up — still raw bait |
| An entry with `producesDecision: []` | Somebody looked and concluded nothing needed deciding |
| An entry with `producesDecision: [{ mdrHandle }]` | Read the record with `get_MDR(mdrHandle)` |
| An entry with `late: true` | Attached after that reasoning closed, so the rationale does not cover it |

More than one entry is normal, not a defect: a matter can be taken up by several analyses.

`takenUpBy` exists on matters only, though a world fact or observed outcome can equally be an
analysis input — see *What the ledger cannot tell you* below before answering "what did this fact
lead to?".

**Answer with what the records decided, and do not claim any one of them was "for" this matter** —
an analysis produces records for the reasoning as a whole, and one of them may exist because of
something no raiser mentioned. Say what was decided and let the user draw the link. Call
`get_analysis` only when asked *why* — fetching it per matter turns one review into a dozen calls.

A matter with `late: true` entries and nothing closing it is worth surfacing on its own: somebody
assumed existing work covered it, and nothing has confirmed that.

### The unprocessed inbox

`list_matters(untaken: true)` returns every **matter** no analysis references — mostly ones parked
through **`save-matter`** during other work. Almanac entries never appear here, whether or not any
analysis has taken them up: there is no unprocessed-almanac inbox, because an admitted fact is not
waiting on anybody.

Present them as raw signal, in the words they were logged in. Do not sharpen them into needs while
summarizing; that is the pickup session's job, with the user present.

**A row's words are not all the raiser's.** `excerpt` is their own opening, trimmed. `title`, where
it appears, is Memolok's own label for the matter — nobody typed it, and nothing in the response says
so. Reading a title back as what somebody logged presents a paraphrase as an utterance, which is the
one thing this section exists to prevent.

So: use the title to say *which* matters are waiting, and quote the excerpt when quoting the raiser.
Where the exact wording carries the point, `get_matter` is where the words are.

**`discover_matters(untaken: true)` is the better read for this section** — the same inbox, answered
as prose with each matter's summary, so you can say what is waiting rather than only that something
is. It labels which words are Memolok's and which are the raiser's, which is the distinction above
made structural.

### Scratchpads are not ledger contents

A review covers decisions, deferrals and unprocessed bait. **Scratchpads are none of those** — they
carry no assertion, no disposition and no expectation, so they never belong in a sweep.

Do not list them alongside matters as though they were waiting for something, and never surface one
as stale, untouched or needing attention. **A note that has sat for a year is behaving correctly.**
There is no review signal here to compute and none to invent.

If the user asks about their notes specifically, that is the **`manage-notes`** journey — reach for
`list_scratchpads` with a `query`, not a review.

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
> stale-constraint review, cross-record open-question queries, or portfolio-wide outcome latency. Nor
> for reading an analysis input backwards: `get_matter` returns `takenUpBy`, but a world fact or an
> observed outcome cannot report which analyses took *it* up.

**Delegating a sweep moves its cost; it does not turn it into a query.** A scout reads and reasons
exactly as you would, in a context you do not have to pay for. Nothing above becomes available
because one was used.

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
