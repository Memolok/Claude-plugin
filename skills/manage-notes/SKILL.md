---
name: manage-notes
description: >-
  The ledger's scratchpads: keep, find, revise and throw-away freeform working notes inside MDLs — pasted quotes,
  scraped pages, rough figures, anything worth keeping that nobody is required to act on. Use when
  the user wants to "keep this", "save this somewhere", "jot this down", "what did I save about X?",
  "show me my notes", "add this to that note", or "delete that note". Also use when you have
  produced material that fits no typed entity and needs an honest home.
argument-hint: "<what to keep, or what you're looking for>"
---

# /memolok:manage-notes — Scratchpads

A **scratchpad** is a freeform Markdown note on a ledger. It makes no promise: no assertion the
ledger stands behind, no expectation that anyone will act on it, nothing downstream depending on it.
Write it, rewrite it, delete it, at any time.

It is the **only disposable thing in Memolok**. Everything else is immutable once admitted or sealed
at t₀.

## Usage

```
/memolok:manage-notes <what to keep, or what you're looking for>
```

Examples:

- `/memolok:manage-notes here's the quote they sent: 3 nodes @ $412/mo, 12-month term`
- `/memolok:manage-notes what did I save about Redis pricing?`
- `/memolok:manage-notes add the second quote to that note`
- `/memolok:manage-notes delete the scraped page from Tuesday`

## Step 0 — Load the method

Load the **`memolok-method`** skill. This journey writes, so the invariants apply — and one rule
there governs everything here: **a scratchpad may never be used as ledger grounding.**

## Non-negotiables

- Never invent `mdlGuid` or `scratchpadId` — the server mints them
- All prose is `{ "markdown": "...", "lang": "en" }`
- **Never delete a note on your own initiative** — not as cleanup, not because the content was
  promoted elsewhere, not because it looks stale
- **Never offer a scratchpad as context, evidence or justification for a decision**
- Say "saved" only after the write succeeded

## Capture costs one turn

The user pastes something and says some version of *"keep this"*. Call `create_scratchpad`. Say
*saved*. Return to whatever they were doing.

**Do not** ask for a title. Do not ask which category it belongs to. Do not ask whether this is
really a Matter. Do not summarize the content back at them. Do not announce a status.

> Saved.

That is the whole interaction. Ceremony is the failure mode here: every question asked at capture
time is a reason the user pastes into a text file outside Memolok next time.

```json
{
  "mdlGuid": "<mdlGuid>",
  "description": {
    "markdown": "Vendor quote 2026-08-06: 3 nodes @ $412/mo, 12-month term, support included.",
    "lang": "en"
  }
}
```

**When you are the one producing the material** — a batch of observations that fits no typed entity —
a scratchpad is the honest home. Do not stretch a `WorldFact` to hold it, and do not mint speculative
`Matter`s to look productive. Say what you did in one clause and move on.

## Which entity is this?

Getting this wrong is costly in both directions. A scratchpad used as intake means real work is never
processed; a `Matter` used as a notepad fills the queue with things nobody intends to decide.

| The user is… | Entity | Tell |
| --- | --- | --- |
| stating something true that decisions should rest on | **`WorldFact`** | *"legal now requires seven-year retention"* — an assertion the ledger should stand behind |
| raising something they expect to be looked at | **`Matter`** | *"we should probably deal with the login timeout"* — somebody is expected to look, however far off |
| keeping material with no expectation attached | **`Scratchpad`** | *"here's the quote they sent"*, *"jotting this before I forget"* — no assertion, no request, no queue |
| deciding something | **`DecisionRecord`** | a choice being made, with alternatives and a commitment |

Two questions settle nearly every case:

1. **Does the user expect anyone to act on this?** If yes, it is not a scratchpad.
2. **Would it be bad if this vanished?** If yes, it is not a scratchpad.

When the signal is genuinely ambiguous, **prefer the scratchpad** and mention the alternative in one
clause. Under-committing is cheap and reversible; over-committing puts noise in the matter queue or,
worse, unstable content in the almanac.

> Saved as a note — say the word if you'd rather it went in as a matter for someone to pick up.

## Getting it back

Notes have no titles, so **content search is the way**, not enumeration.

```
search_scratchpads(mdlGuid, query="redis pricing")
```

Rows are previews — an excerpt, a length, and `matchExcerpt` around the hit. Read `matchMode`:

| `matchMode` | Meaning |
| --- | --- |
| `text` | Stemmed, relevance-ranked. The normal path |
| `regex` | The text pass found nothing, so a substring pass answered. Fine — but the user's words did not match the note's words |

An empty result means **no note matched**. It does not mean "now list everything and read it".

`list_scratchpads(mdlGuid)` is for browsing, most recently touched first. Fetch a full body with
`get_scratchpad` only when you actually need the text.

**Naming a note in prose.** There are no titles, so build a handle from the excerpt and the date —
*"the Redis quote from the 7th"*, *"the scraped pricing page"*. Do not volunteer the `sp_` id; cite
it if the user asks.

## Revising

Read it, compose the new body, replace it. `replace_scratchpad` is a **full replacement** — there is
no append and no splice.

No version prompt, no change justification, no diff review unless the user asks for one. The
contributor set grows silently; do not narrate it.

A note may legitimately be edited into something unrecognizable. That is not an error.

## Throwing away

*"Delete that"* deletes it. Confirm only which entry you mean — one line, not a warning about
consequences, because there are none.

> That's the vendor quote from the 6th — deleting it now. It's immediate and can't be undone.

Say that last part. There is **no recovery window**, and a user who assumes there is a trash can will
find out at the worst moment.

An agent that treats deletion as risky has misunderstood the entity: nothing may reference a
scratchpad, so nothing breaks downstream, ever. But the note is the user's, so **never delete one
unprompted** — not to tidy up, not because you just promoted its content, not because it looks old.

## When a note turns out to matter

Notice it, and say so. Do not act on it silently.

> That pricing note looks like a premise the caching decision would rest on — want me to admit it as
> a World Fact?

If the user agrees, author the `WorldFact`, `Matter` or `DecisionRecord` **fresh, through its own
ordinary path**, in its own wording, with its own admission. Do not paste the note in verbatim as
though it were a citation. Do not modify or delete the source note unless asked.

Then tell them, once, plainly:

> Admitted. Worth knowing: the ledger keeps no trail back to the note — the fact stands on its own
> now, and the note stays a note.

That is a deliberate property, not an oversight. Saying it once prevents a false expectation later.

## Scratchpads never argue for a decision

You may read a scratchpad while helping someone think, exactly as you may read anything they show
you in conversation.

What you may **never** do is present its content as ledger grounding: offering it as `hasContext`,
citing it in a Verdict, or letting it stand in for a premise nobody admitted. If material is
load-bearing for a decision, it must be admitted properly first.

The tools enforce this — a scratchpad id is refused by every reference field — but the point is not
to get caught. **The whole disposability contract rests on this.** Notes are only safe to rewrite and
delete freely because no record's reasoning can be resting on one.

If a user asks you to cite a note, explain rather than just refusing:

> I can't point the record at the note — nothing in the ledger can reference one, precisely so you
> can rewrite or bin it without breaking anything. If that pricing figure is a premise this decision
> rests on, let's admit it as a World Fact and cite that instead.

## Tips

- A ledger with no scratchpads is entirely normal, not a gap.
- Notes are **MDL-shared**: anyone on the ledger can read, edit and delete them. Do not describe them
  as private.
- Notes are scoped to one ledger. There are no account-level or cross-ledger notes.
- Bodies cap at 64 KB. If a paste is refused, offer to split it or to keep a pointer to where the
  source lives, rather than silently truncating.
- Nothing sweeps or reviews scratchpads. A note untouched for a year is behaving correctly — never
  surface it as stale.
- Whatever the user typed is the note. Do not add auto-summaries, extracted entities or your own
  annotations to the body. Anything you infer belongs in conversation, not in their note.
