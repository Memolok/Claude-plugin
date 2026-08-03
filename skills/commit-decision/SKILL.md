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

Load the **`memolok-method`** skill. Rule C governs this entire skill: t₀ is a separate, explicit
ceremony, never bundled into a recap.

## Non-negotiables

- Never invent `mdlGuid`, `mdrHandle`, or `mdrNumber` — the server mints them
- All prose is `{ "markdown": "...", "lang": "en" }`
- **Commit only when the user asked to, in their own words** — never because your draft reads firm
- Say "captured" only after the write succeeded
- Cite `mdrNumber` once admitted; never volunteer `mdrHandle` (Rule F)
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

If you are not certain, ask plainly: *"do you want this sealed on the ledger now, or left editable?"**
Sealing is not reversible in place, so the cost of asking is far lower than the cost of guessing.

## Workflow

### 1. Read the record

```
get_MDR(mdlGuid, mdrHandle)
```

Confirm it is still staged. If `mdrNumber` is already set, it was sealed previously — route to
**`revise-decision`** instead.

### 2. Check the gates for the target

| Target | Requires |
| --- | --- |
| **Accepted** | Head Claim · ≥1 alternative · `chosenAlternative` matching an alternative id · non-empty Verdict · ≥1 expected outcome |
| **Rejected** | Head Claim · non-empty Verdict explaining why not to proceed |

Anything missing gets patched **before** the ceremony, via `update_MDR` — see the `record-decision`
skill. Do not start a ceremony you will have to interrupt.

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

**Rejected** is a first-class t₀ commitment: the organization will **not** proceed under this head
Claim. It gets a ledger number, a Verdict, and the same permanence as an Accept.

Offer it as readily as Accept when deliberation has shown no viable path. A sealed rejection prevents
the same question being reopened every quarter, which is worth more than a weak Accept or an
indefinite draft.

A Rejected record may not carry `supersedes`.

## Formal versus informal

| Posture | Path |
| --- | --- |
| **Informal** (default) | Deliberating → **Accepted** or **Rejected** |
| **Formal** | Deliberating → Proposed → **Accepted** or **Rejected** |

**Proposed is not t₀.** It records that a candidate choice has been fixed, pending ratification. Use it
only where there is real governance — distinct deciders, a committee vote, compliance sign-off. On
informal work, do not surface it at all.

## After t₀

The body is sealed whether or not the record is retractable. `retractable: true` means only that an
admin or owner can uncommit it back to staged — it does **not** mean it can be patched.

Never tell a user a committed record can be edited in place. If they want a change, route to
**`revise-decision`**.

## Tips

- One-shot mint at `Accepted` or `Rejected` through `create_MDR` triggers the same seal. Run the
  ceremony first — the shortcut is in the tool call, not in the conversation.
- `transition_MDR_status` accepts `Decided`, `Settled`, and `Committed` as aliases for `Accepted`.
  `create_MDR` does not.
- If the transition is refused, the message names the missing gate. Patch it and return to step 3.
- Record the outcome ids (`eo-…`) from the response — a later wake needs them.

## References

| File | Load when |
| --- | --- |
| `references/ceremony.md` | Rejection, one-shot mint, or the user finds ceremony tiresome |
| `references/transition-payloads.md` | Building the transition call, or a gate error needs decoding |
