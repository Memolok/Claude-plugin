---
name: manage-almanac
description: >-
  Admit and review the world facts a Memolok ledger reasons from — budgets, constraints, policies,
  team capabilities, anything true independently of a particular decision. Use when the user states a
  premise rather than a choice ("legal requires seven-year retention", "the budget is $10k", "nobody
  here writes Rust"), wants to correct a fact that was wrong when admitted, or wants a decision to
  cite its context before it is sealed.
argument-hint: "<the fact>"
---

# /memolok:manage-almanac — The world almanac

Admits premises to the ledger's almanac and cites them from staged records. World facts are minted
independently of any decision, so correcting one never means editing a sealed record.

## Usage

```
/memolok:manage-almanac <the fact>
```

Examples:

- `/memolok:manage-almanac legal requires seven-year retention on customer records`
- `/memolok:manage-almanac the hardware budget for this project is $10k`
- `/memolok:manage-almanac nobody on the team writes Rust`
- `/memolok:manage-almanac what facts is this ledger working from?`

## Step 0 — Load the method

Load the **`memolok-method`** skill for the almanac's role and the prose format.

## Non-negotiables

- Never invent `mdlGuid` or `worldFactId` — the server mints them
- All prose is `{ "markdown": "...", "lang": "en" }`
- `hasContext` can only be patched while a record is **staged** — it freezes at t₀
- A record may never cite **its own** observed outcome as context
- `correctsFact` is for facts that were **wrong when admitted**, never for the world changing
- Say "captured" only after the write succeeded

## What belongs here

A world fact is a claim about the world that is true whether or not anyone decides anything about it.

| It is a world fact | It is not |
| --- | --- |
| *"The hardware budget is $10k"* | *"We'll spend $8k on drives"* — a decision |
| *"Legal requires seven-year retention"* | *"We should add archival storage"* — a Verdict with no Need |
| *"Nobody on the team writes Rust"* | *"Rust is too risky for us"* — a deliberation fact |
| *"The vendor's SLA is 99.5%"* | *"The vendor is unreliable"* — a judgement, unless someone observed it |

The test: **would this still be true if the decision it relates to had never come up?**

If the user brought a *symptom* rather than a premise, that is bait — use **`save-matter`** or
**`record-decision`** instead.

### The almanac is not a notepad

A World Fact is a **citable admission the ledger stands behind**, immutable in substance once
admitted and correctable only by a successor. That discipline is the whole point, and it is the wrong
container for material that does not want it.

Raw working material — a pasted quote, a scraped page, a price list, a batch of observations that fit
no typed entity — belongs in a **scratchpad** (**`manage-notes`**), not here. The pull toward
bundling such material into one World Fact is real and should be resisted: it puts unstable content
in the almanac, where every future record may cite it and no correction path fits.

| The user brought | Where it goes |
| --- | --- |
| *"the vendor's SLA is 99.5%"* | **World Fact** — an assertion, checkable, worth standing behind |
| *"here's their whole pricing page"* | **Scratchpad** — source material, not an admission |
| *"legal requires seven-year retention"* | **World Fact** |
| *"rough figures from an afternoon of digging"* | **Scratchpad** — nothing here is settled |

The test: **is the user asserting this, or just keeping it?** If a decision could reasonably rest on
it and they would stand behind it, admit it. Otherwise keep it as a note — and if it later turns out
to matter, admit it *then*, freshly worded.

## Workflow

### 1. Establish the ledger

The `mdlGuid` in play, else `.memolok/mdl.yml`, else `get_MDLs`. Facts are ledger-scoped, so a fact
admitted in one ledger is invisible in another.

### 2. Check what is already there

```
list_world_facts(mdlGuid)
```

Avoid admitting a near-duplicate. If a prior fact covers the same ground but is now wrong, that is a
correction — step 4.

### 3. Admit the fact

```json
{
  "mdlGuid": "<mdlGuid>",
  "claimDescription": {
    "markdown": "Customer records must be retained for seven years under the retention policy legal issued in 2026-03.",
    "lang": "en"
  }
}
```

Write it so it is checkable later. *"The budget is tight"* is not a premise anything can be reasoned
from; *"the hardware budget is $10k"* is.

### 4. Correcting versus drifting

This distinction is the one that matters here.

| Situation | What to do |
| --- | --- |
| The fact was **wrong when admitted** — misread the policy, bad figure | New fact with `correctsFact` naming the prior one |
| The **world changed** — budget was raised, policy was rewritten | New fact with **no** `correctsFact` |

```json
{
  "mdlGuid": "<mdlGuid>",
  "claimDescription": {
    "markdown": "The retention requirement is seven years, not five; the earlier fact misread the policy's transitional clause.",
    "lang": "en"
  },
  "correctsFact": "<priorWorldFactId>"
}
```

A correction says *we were wrong*. Drift says *it changed*. Conflating them makes the ledger unable to
distinguish "we reasoned from bad information" from "we reasoned correctly and the world moved" — and
those call for very different responses.

Each prior admission accepts at most one direct corrector.

### 5. Cite it from a decision

While the record is still **staged**:

```json
{
  "mdlGuid": "<mdlGuid>",
  "mdrHandle": 1,
  "patch": {
    "hasContext": ["<worldFactId>", "<priorObservedOutcomeId>"]
  }
}
```

An ordered list of world fact and prior observed outcome ids from the same ledger. It **freezes at
t₀** — this is the record's epistemic snapshot, the premises it actually reasoned from.

Do this **before** committing. There is no way to add context afterwards, and a record whose premises
were never cited cannot later be found when one of those premises turns out to be wrong.

## Why premises live outside records

Facts live in the almanac rather than embedded in each record so that correcting one does not require
editing sealed history. When a fact is corrected, the records that cited it stay exactly as they were —
honest at their own t₀ — and the correction is what lets someone later ask which decisions rested on a
premise that has since changed.

That question — *polluted premises*, decision decay — has no tool yet. You can answer it by reading
`hasContext` across records, but say that you read it rather than implying the ledger computed it.

## Tips

- `get_world_fact(mdlGuid, worldFactId)` fetches one; `list_world_facts(mdlGuid)` lists all.
- Observed outcomes are a **kind** of world fact — a record's `hasContext` can cite a prior wake, just
  never its own.
- Cross-team handoff works through this skill: admit another team's decision as a world fact in your
  ledger. A reference, not a merge. See `scopes-and-bridging.md` in **`memolok-method`**.
- There is no way to delete or edit a fact. Facts are immutable in substance once admitted; the only
  move is a correcting successor.
- If the user states a premise while working through a decision, admit it and cite it rather than
  folding it into the Verdict prose — it is reusable by every later record.
- A fact is true regardless of what the ledger says it is for. Never decline one as off-purpose, and
  never cite the ledger's stated purpose as context — only World Facts and prior Observed Outcomes are
  citable.

## References

The wake side of the almanac — observed outcomes and the learning delta — lives in the
**`record-outcome`** skill.
