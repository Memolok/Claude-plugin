---
name: revise-intent
description: >-
  State or revise what a Memolok ledger is for — its purpose, who it serves, and roughly where it is
  heading. Use when someone asks "what is this ledger for", says the team's focus or the project's
  scope has shifted, wants to write down why a ledger exists, or wants to update a purpose that has
  gone stale. Not for changing decisions — that is a different job.
argument-hint: "[the ledger's purpose]"
---

# /memolok:revise-intent — What this ledger is for

Writes or rewrites a ledger's stated purpose. One short piece of prose per ledger, revised freely,
read at the top of a cold session by whoever picks the ledger up next.

## Usage

```
/memolok:revise-intent [what this ledger is for]
```

Examples:

- `/memolok:revise-intent what is this ledger for?`
- `/memolok:revise-intent we've widened this from just the API to the whole platform`
- `/memolok:revise-intent this ledger has never said what it's about — let's fix that`
- `/memolok:revise-intent the purpose is out of date, we stopped doing the migration work`

## Step 0 — Load the method

Load the **`memolok-method`** skill, then its `ledger-intent.md` reference for how a purpose is
drafted — the three elements, the two shapes, and the length.

## Non-negotiables

- Never invent `mdlGuid` — the server mints it
- All prose is `{ "markdown": "...", "lang": "en" }`, nested under `description`
- **`set_ledger_intent` replaces the whole statement** — it never appends, and there is no history
- Read the current statement before replacing it; a rewrite from memory loses what was there
- Intent is never cited by a record and never justifies, blocks, or explains one
- Say "captured" only after the write succeeded

## What this skill is not

| The user wants to change | Skill |
| --- | --- |
| What this ledger is *for* | **This one** |
| A decision already sealed on the ledger | **`revise-decision`** |
| A premise the ledger reasons from | **`manage-almanac`** |

A stated purpose is not a decision. It is not sealed, it has no t₀, nothing depends on it, and
rewriting it costs nothing — which is exactly why it is a separate job from revising a record.

## Workflow

### 1. Establish the ledger

The `mdlGuid` in play, else `.memolok/mdl.yml`, else `get_MDLs`. If the user belongs to several and has
not said which, ask — rewriting the wrong ledger's purpose is silent and confusing.

### 2. Read what is there

```
get_MDL(mdlGuid) → { mdlGuid, title, role, ledgerIntent }
```

`ledgerIntent: null` means the ledger has never stated one. That is not a defect; it is most ledgers.
Say so plainly and offer to write the first.

Read the existing statement back to the user before changing it. They are usually revising one clause,
not the whole thing, and they cannot tell you which clause if they cannot see it.

### 3. Draft the replacement with them

Follow `ledger-intent.md` in **`memolok-method`**. Two things that go wrong here:

- **Sending a fragment.** The tool replaces everything. If the user says *"add that we also cover
  billing now"*, you send the whole statement with billing in it — not the clause.
- **Sharpening it into a Claim.** A purpose has no falsifiability bar. *"Heading toward one shared
  pipeline"* is a good intent and would be a terrible head **Claim**. Rule G does not apply here.

Show them the full replacement and get agreement before writing.

### 4. Write it

```json
{
  "mdlGuid": "<mdlGuid>",
  "ledgerIntent": {
    "description": {
      "markdown": "Rebuilding the home media server so it survives a drive failure and a house fire. One hobbyist, weekends only. Heading toward simple hardware with backups that get tested.",
      "lang": "en"
    }
  }
}
```

Requires `member` or above. A `visitor` cannot write one — say so rather than letting them draft one
first.

Confirm briefly and stop. This is a small job; it does not need a summary of the whole ledger.

## When the purpose and the ledger have drifted apart

Sometimes the reason someone is here is that the records no longer look like the stated purpose. Both
readings are legitimate and it is the user's call which:

- **The purpose is stale** — the work moved and nobody updated the sentence. Revise it.
- **The purpose is right and the work wandered** — that is worth noticing, but it is not something this
  skill fixes, and it is **not** a finding you deliver as a judgement.

What you must not do is treat the mismatch as a problem with the *records*. A decision that diverges
from stated intent needed no justification when it was made and needs none now. Never suggest revising
or superseding a record because it does not match the ledger's current purpose — a record stands on its
own reasoning, not on what the ledger says it is about today.

## Tips

- There is no way to remove a purpose once stated — only to replace it. If the user wants it gone,
  the honest move is prose that says the ledger's scope is currently open.
- Nothing cites it, so revising it can never invalidate a record, pollute a premise, or affect any
  outcome. It is one of the few things on a ledger that is genuinely safe to change.
- If a ledger's purpose has to be long to be accurate, that is usually a sign of two ledgers wearing
  one name. See `scopes-and-bridging.md` in **`memolok-method`**.
- Non-members get `Memolok Decision Ledger not found.` rather than a permission error — deliberate, so
  it leaks nothing. If someone expected access, an administrator needs to add them.
