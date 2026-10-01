---
name: record-preview
description: >-
  How to show the user a Memolok Decision Record you have drafted, before it is written to a
  ledger: a new record, one that amends, supersedes or settles another, or a correction to a draft
  already shown. You MUST NOT present such a draft without first loading this skill, once per
  session — users learn to read drafts from one layout repeated, and an improvised preview beside
  the one in this layout teaches them two. The seal is not a preview; `commit-decision` has its
  own words for that moment.
user-invocable: false
---

# Record preview

Every decision record you draft is shown to the user before it is written, and **every one is shown
in the layout below**: the first preview of a session and the tenth, a new record and one correcting
a sealed record alike. The user comes to know where the Need sits, where the choice is marked and
where the scope boundaries are, and reads a preview by looking in those places. A preview laid out
from memory beside one laid out here teaches two layouts, even when both say the same thing.

**Every number in this file is illustrative.** `MDR-7` names nothing; do not repeat these examples
to the user as though they were entries on their ledger.

## The layout

---

**Need:** Storage must hold a 500GB–2TB FLAC library with ongoing growth from continued
digitization, run on hardware already owned, and survive a single drive failure without data loss via
an off-device backup — full RAID-level redundancy is not required.

**Alternatives:**

| id | label | summary |
| --- | --- | --- |
| ❌ `alt-raid` | Software RAID1 mirror | Mirror on existing hardware; no separate backup |
| ✅ `alt-backup` | Single drive + scheduled backup | Library on one repurposed drive; off-device backup on a schedule |
| ❌ `alt-naked` | Single drive, no backup | Accept full loss risk |

**Deliberation:**

- `alt-raid` — Survives one drive failure locally. No protection against fire, theft, or accidental
  deletion; more complexity.
- `alt-backup` — Works with hardware on hand; backup covers fire, theft, and deletion. Requires an
  ongoing backup habit.
- `alt-naked` — Violates the survivability intent in the Need.

**Verdict:** Run the library on a single repurposed drive and rely on a scheduled off-device backup
rather than RAID.

**Chosen:** `alt-backup`

**Expected outcomes:**

- `eo-no-raid` — No RAID complexity; works with hardware already on hand
- `eo-backup-habit` — A missed backup window is a loss-exposure gap, and none lasts longer than a week

**Open questions (scope boundaries — they travel with the record):**

- `oq-backup-target` — Exact backup destination and schedule: external drive or cloud, and how often

**Intended status:** Deliberating — the record stays editable on the ledger

> Does this cover what we discussed? The open questions are what this decision deliberately leaves
> unsettled; we do not need to settle them to record this, or to Accept it later.

---

## Two regions that appear only when they have something to say

**What it changes** goes directly under the Need, on a record that amends or supersedes another. It
names the record and says how much of it this one replaces:

> **What it changes:** Amends MDR-7, on the backup schedule only; everything else in MDR-7 still
> governs.

> **What it changes:** Supersedes MDR-7 whole. The library moves off the repurposed drive.

An amendment's preview shows its deltas and nothing carried forward: the rest of the original still
governs, so restating it in the preview would present a rewrite the record is not making. Read the
original with `get_MDR_details` before drafting either line — what it changes may sit in an
alternative, an argument or an expected outcome.

**Context and links** goes after the deliberation. It lists what the record rests on and points at:
the world facts and earlier outcomes it cites as context, the records it depends on or conflicts
with, the open question it settles. Say what each one is in words, beside its identifier:

> **Context and links:** rests on `wf_…` (the library is 1.4TB today); settles `oq-backup-target`
> on MDR-7.

Show them even though they are written in a later call than the body. The preview is of the record,
not of the next call.

Leave either region out when there is nothing to put in it. Never write "none" there.

## Notes on the layout

**Label the head "Need".** In conversation it is also the head Claim, and the two may stand side by
side there; in the preview the label is always **Need**. It comes first because everything else is
answerable to it, and showing it first makes an over-sharpened Need obvious: if it already names the
chosen mechanism, you will see it here.

**Show alternatives as a table** when there are three or more, and as a list when there are one or
two. An alternative with no `description` is fine; it means the user named the option without
elaborating.

**Mark the chosen alternative ✅ and the others ❌** once `chosenAlternative` is set. With none
chosen, mark nothing. Keep the order the record holds them in, here and in the deliberation: the
chosen one need not be first, and a preview reordered around it no longer matches the record the
user will read later.

**Attribute the deliberation to the options**, never to a generic split of arguments for and
against. Each line should be traceable to something the user actually said.

**Every region above the two optional ones appears in every preview**, in this order. One the draft
does not have yet says so in place, with what it is needed for: *Expected outcomes: none yet —
needed before the record can be Accepted.* A missing region reads as one you forgot.

**Label the open questions as scope boundaries** in the preview itself. Users read a bare "open
questions" heading as a to-do list and try to clear it, which is exactly the behaviour Rule D exists
to prevent.

**State the intended status explicitly** and say the record stays editable. That is what stops
preview confirmation from being mistaken for commitment.

**Ask the one closing question, and wait.** If the answer is no, return to the region that was
wrong; do not persist a draft the user has not endorsed.

**A correction shows only the regions it changed**, under the same labels and in the same order,
and says the rest stands. A second full preview for a one-line fix buries the fix. On a persisted
draft, take the regions from `get_MDR_details` rather than from memory: a write's reply does not carry
them.
