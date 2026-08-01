# Fish preview

Present the draft in this shape before any write. The default intended status is **Deliberating**, and
confirming the preview is not t₀.

---

**Head Claim:** Storage must hold a 500GB–2TB FLAC library with ongoing growth from continued
digitization, run on hardware already owned, and survive a single drive failure without data loss via
an off-device backup — full RAID-level redundancy is not required.

**Alternatives:**

| id | label | summary |
| --- | --- | --- |
| `alt-backup` | Single drive + scheduled backup | Library on one repurposed drive; off-device backup on a schedule |
| `alt-raid` | Software RAID1 mirror | Mirror on existing hardware; no separate backup |
| `alt-naked` | Single drive, no backup | Accept full loss risk |

**Deliberation:**

- `alt-backup` — Works with hardware on hand; backup covers fire, theft, and deletion. Requires an
  ongoing backup habit.
- `alt-raid` — Survives one drive failure locally. No protection against fire, theft, or accidental
  deletion; more complexity.
- `alt-naked` — Violates the survivability intent in the head Claim.

**Verdict:** Run the library on a single repurposed drive and rely on a scheduled off-device backup
rather than RAID.

**Chosen:** `alt-backup`

**Expected outcomes:**

- No RAID complexity — works with hardware already on hand
- Ongoing backup habit; a missed window means a loss-exposure gap
- Backup destination and mechanism still to be chosen

**Open questions (scope boundaries — they travel with the record):**

- Exact backup destination and schedule: external drive or cloud, and how often

**Intended status:** Deliberating — the record stays editable on the ledger

> Does this cover what we discussed? Out-of-scope items stay as open questions; we do not need to
> settle them to record this, or to Accept it later.

---

## Notes on the format

**Lead with the head Claim.** It is what everything else is answerable to, and showing it first makes
an over-sharpened Need obvious — if it already names the chosen mechanism, you will see it here.

**Show alternatives as a table** when there are three or more, as a list when there are one or two. An
alternative with no `description` is fine; it means the user named the option without elaborating.

**Attribute the deliberation to the options**, not to a generic pros-and-cons split. Each line should
be traceable to something the user actually said.

**Label the open questions as scope boundaries** in the preview itself. Users read a bare "open
questions" heading as a to-do list and try to clear it, which is exactly the behaviour Rule D exists
to prevent.

**State the intended status explicitly** and say the record stays editable. That is what stops preview
confirmation from being mistaken for commitment.

**Ask one closing question and wait.** If the answer is no, return to the region that was wrong — do
not persist a draft the user has not endorsed.

## What not to put in the footer

Do not offer **Proposed** on informal work, and do not offer **Accepted** unless the user has already
raised sealing. A preview footer that reads *"…or shall I mint this straight to Accepted?"* infers
commitment from your own draft.
