---
name: commit-decision
description: >-
  Seal a Memolok Decision Record at t₀ as Accepted or Rejected, after checking the gates and marking
  the moment with the user. Trigger with "lock it in", "accept this", "reject that path", "commit
  it", "we're going with that", or when the user explicitly asks to finalize a staged record. Both
  outcomes are real commitments and neither can be edited in place afterwards.
argument-hint: "[accept | reject] <record>"
---

# /memolok:commit-decision — Seal at t₀

Takes a staged record across t₀. The server stamps `decidedAt`, assigns the ledger number, and seals
the body permanently.

## Usage

```
/memolok:commit-decision [accept | reject] <which record>
```

Examples:

- `/memolok:commit-decision accept the caching decision`
- `/memolok:commit-decision lock in MDR for the storage layout`
- `/memolok:commit-decision reject the microservices migration`
- `/memolok:commit-decision` — when the record just drafted is the obvious subject

## Step 0 — Load the method

Load the **`memolok-method`** skill and its `facilitation.md` reference. Rule C governs this entire
skill: t₀ is a separate, explicit ceremony, never bundled into a recap.

## Non-negotiables

- Never invent `mdlGuid`, `mdrHandle`, or `mdrNumber` — the server mints them
- All prose is `{ "markdown": "...", "lang": "en" }`
- **Commit only when the user asked to, in their own words** — never because your draft reads firm
- Say "captured" only after the write succeeded
- Cite `MDR-{n}` once admitted (anywhere), `MDRh{handle}` or a head Claim paraphrase while staged (only in chat sessions and in transient plans); never a bare handle (Rule F)
- Open questions do **not** block commit (Rule D)
- The ledger's stated purpose bears on **nothing** here — never raise a mismatch with it before a seal

## The one precondition that matters

The user must have **explicitly** asked to seal. "Accept this", "lock it in", "reject that path", "yes
commit it".

These are **not** commitment:

- A confirmed fish preview — that endorses accuracy, not commitment
- A Verdict you drafted that sounds decisive
- The record looking complete
- The user saying "sounds good" about your summary

If you are not certain, ask plainly: *"do you want this sealed on the ledger now, or left editable?"*
Sealing is not reversible in place, so the cost of asking is far lower than the cost of guessing.

## Workflow

### 1. Read the record

```
get_MDR(mdlGuid, mdrHandle)
```

Confirm it is still staged. If `mdrNumber` is already set, it was sealed previously — route to
**`revise-decision`** instead.

### 2. Check the gates for the target

The gates are the method's lifecycle table. Anything missing gets patched **before** the ceremony, via
`update_MDR` — see the `record-decision` skill. Do not start a ceremony you will have to interrupt.

Open questions are never a gate. If the user wants to clear one first, that is their call, but do not
propose it.

### 3. Run the ceremony

A separate turn, before the transition. Brief, warm, and specific — not formalism.

State what is being committed, name what seals, and ask once:

> **Ready to seal this on the ledger.**
>
> You're committing to **a single drive plus a scheduled off-device backup** for the FLAC library.
> The moment I commit, the server stamps the decision time and the record seals — Verdict,
> alternatives, expected outcomes and open questions all become immutable ledger evidence.
>
> **Still open by design:** backup destination and schedule. That travels with the record and can be
> settled by a later decision.
>
> Shall I commit this as **Accepted**?

Then wait for an explicit yes.

Naming the open questions here is deliberate: it shows they are intended, so the user is not surprised
to find them frozen on the record later.

Ceremony variants — rejection, one-shot mint, terse mode: `references/ceremony.md`.

### 4. Transition

```json
{ "mdlGuid": "<mdlGuid>", "mdrHandle": 1, "status": "Accepted" }
```

Payloads and the formal-governance `Proposed` step: `references/transition-payloads.md`.

### 5. Verify and mark it

`get_MDR` and confirm `status`, `decidedAt`, `mdrNumber`, and `retractable`. Then one positive beat,
citing the number the record now has:

> **MDR-7 is Accepted** — sealed just now. That's on the ledger.

Then carry on with whatever the user was doing. Do not offer to end the session.

## Rejection is a real outcome

Offer **Rejected** as readily as Accept, and in more situations than an exhausted deliberation — the
`memolok-method` skill carries the full set. A Rejected record carries no `amends`, `supersedes` or
`settlesOpenQuestion`: nothing it says is in force, so it changes, retires and settles nothing.

## After t₀

**The record now has a number, which is the thing the work could not name pre-t₀.** If anything
outside the ledger is going to cite it — code, documentation, a commit message, a message to
somebody — this is the moment that becomes possible, and the citation has to be anchored before it
is written. The **`memolok-citations`** skill carries the order and the form.

## Tips

- One-shot mint at `Accepted` or `Rejected` through `create_MDR` triggers the same seal. Run the
  ceremony first — the shortcut is in the tool call, not in the conversation.
- If the transition is refused, the message names the missing gate. Patch it and return to step 3.
- The `eo-…` ids you chose come back on every read; a later wake needs them.

## References

| File | Load when |
| --- | --- |
| `references/ceremony.md` | Before sealing anything but a routine Accept — a rejection ceremony reads differently, and the terse variant still has one part that must never be dropped |
| `references/transition-payloads.md` | A gate error needs decoding — the message names the missing field, and the fix is rarely the one the wording first suggests |
