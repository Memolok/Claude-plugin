---
name: save-matter
description: >-
  Park something on a Memolok ledger without stopping to decide anything about it — register it in
  the user's own words and get straight back to what they were doing. Use when the user notices a
  bug, an annoyance, an idea, an opportunity, or an unanswered question mid-task and says "log this",
  "note that for later", "park it", "don't do anything about it now", or "remind me about X".
argument-hint: "<what you noticed, in your own words>"
---

# /memolok:save-matter — Park a matter

Registers a Matter verbatim on the ledger and stops. No sharpening, no analysis, no decision. The
matter waits, referenced by nothing, until some later session picks it up.

A Matter is any pre-analytical input to possible decision work — a defect, a desired feature, an
improvement, an opportunity, a mandate, or a question nobody has answered yet. It carries no
emotional polarity and no truth value; it is raw material, not a claim.

## Usage

```
/memolok:save-matter <what you noticed>
```

Examples:

- `/memolok:save-matter the CSV export mangles unicode in the header row`
- `/memolok:save-matter we could probably drop the nightly rebuild entirely`
- `/memolok:save-matter nobody can find the billing settings`
- `/memolok:save-matter park this: is our retention window actually seven years or did we assume that?`

## Step 0 — Load the method

Load the **`memolok-method`** skill for the bait-versus-Claim distinction and prose format. This
journey writes, so the invariants apply.

## Non-negotiables

- Never invent `mdlGuid` or `matterId` — the server mints them
- All prose is `{ "markdown": "...", "lang": "en" }`
- Register the user's words **verbatim** — matters are immutable, and the raw input is the point
- Do **not** call `create_analysis` in this journey
- Say "captured" only after the write succeeded
- One short confirmation, then stop — the user is in the middle of something else

## When this skill applies

The signal is not *what* the user brought — it is that they have **explicitly declined to deal with it
now**. "Log this and move on", "park it", "not now", "just so we don't forget".

| Situation | Skill |
| --- | --- |
| User wants it noted, not decided | **This one** |
| User wants to act on it now | **`record-decision`** |
| User is unsure whether to act | **`record-decision`** — its intake fork handles that |
| User wants it *kept*, with nobody expected to act on it | **`manage-notes`** — a scratchpad |

If the user did not signal parking, do not assume it on their behalf because they seem busy. Silently
downgrading a decision to a note loses the decision.

### Matter or scratchpad?

Both are one-turn captures; the difference is "actionability" – **whether anyone is expected to do something**.

A Matter carries an expectation that somebody will eventually look at it, however distant. A
scratchpad carries none: it is a pasted quote, a scraped page, rough figures, material worth keeping
that nobody owes anything to.

| The user brought | Entity |
| --- | --- |
| *"the CSV export mangles unicode"* | **Matter** — a defect somebody should fix |
| *"here's the quote they sent"* | **Scratchpad** — no request in it |
| *"we could probably drop the nightly rebuild"* | **Matter** — an idea somebody should weigh |
| *"jotting this before I forget"* | **Scratchpad** |

Getting it wrong costs in both directions: a scratchpad used as intake means real work is never
processed, and a Matter used as a notepad fills the unprocessed-bait inbox with things nobody intends
to decide — which is worse, because matters are immutable and cannot be deleted.

When genuinely ambiguous, prefer the **scratchpad** and say so in one clause. It is the reversible
choice: a note can be promoted into a matter later (albeit only via agent facilitation), whereas a matter can never be taken back.

## Workflow

### 1. Establish the ledger

Use the `mdlGuid` already in play. If there is none, check `.memolok/mdl.yml` when project files are
available, then `get_MDLs`. Pick the ledger whose worldview the matter belongs to.

**Do not create a ledger for a drive-by.** If the user has none, say the matter needs somewhere to
live and offer `start` — but do not make them stop and name a ledger when they were trying not to stop
at all. Holding the text in the conversation and offering to log it later is a reasonable answer.

### 2. Check for a duplicate

```
list_matters(mdlGuid, untaken: true)
```

If the same thing is already parked, say so and stop — do not register a second one. Matters are
immutable, and there is no way to merge or delete them.

A duplicate that does get registered is not a disaster, and there is nothing to clean up: one
analysis can take up every copy at once, which records what actually happened better than a merge
would. Checking first is still cheaper than explaining three identical matters later.

Skip this when the ledger is large and the user is clearly mid-flow; a duplicate is a smaller cost
than an interruption.

### 3. Register verbatim

```json
{
  "mdlGuid": "<mdlGuid>",
  "description": {
    "markdown": "The CSV export mangles unicode in the header row.",
    "lang": "en"
  }
}
```

Use the user's own phrasing. Do not:

- turn it into a target (*"exports must preserve UTF-8"*) — that is sharpening, and it is the next
  session's job with the user present
- add diagnosis you inferred but they did not say
- tidy it into something more professional-sounding

Light cleanup is fine: dropping a stray "ugh", fixing an obvious typo, expanding "it" when the referent
would be lost. Preserving meaning is the bar, not preserving keystrokes.

### 4. Confirm and get out

One line. Name what was saved and where it will resurface:

> Saved on **{ledger title}** — it'll show up in the unprocessed list next time you go looking.

Then return to whatever the user was doing. Do not offer to sharpen it, do not ask whether they want
to decide now, and do not summarize the methodology.

## What happens next

Nothing references the matter, which is what the unprocessed-bait inbox means. A later session finds
it with `list_matters(untaken: true)` and takes it through **`record-decision`**, which sharpens it
into a head Claim and mints a record.

That deferred pickup is exactly why parking is safe: the provenance chain from the user's original
words to the eventual decision stays intact, however long the gap.

If it turns out no decision was warranted, the pickup session closes it honestly through Path B.

## Tips

- Several matters at once is fine — register each separately. One matter, one input.
- A parked matter is not a to-do item and carries no priority, assignee, or due date. Memolok is
  not an issue tracker; if the user needs one of those, they need their tracker as well as this.
- If the user starts reasoning about the fix while logging it, that is a decision emerging — offer
  `record-decision` rather than quietly discarding their reasoning.
- Nothing here is destructive, but nothing is deletable either. A matter registered by mistake is
  closed through Path B in a later session, not removed.

## References

How a matter comes to rest, and which of those endings have no tool yet, are covered by
**`record-decision`**. Load that skill rather than reaching into its files.
