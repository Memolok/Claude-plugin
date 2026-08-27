---
name: grill-me
description: >-
  Run a relentless one-question-at-a-time decision interview, walking the decision tree branch by
  branch and recording each decision as it settles. Trigger with "grill me", "interview me about
  this", "help me think this through properly", "ask me what you need to know", or when the user
  wants structured decision support on a problem they have not worked out yet.
argument-hint: "<topic or decision>"
---

# /memolok:grill-me — Decision interview

Interrogates a topic until there is shared understanding, one question per turn, and lands each
decision on the ledger as it settles. The user ends the session; you do not.

## Usage

```
/memolok:grill-me <topic>
```

Examples:

- `/memolok:grill-me the storage architecture for the media server`
- `/memolok:grill-me I need to figure out our deployment strategy`
- `/memolok:grill-me walk me through the auth rewrite`
- `/memolok:grill-me` — when the topic is already on the table

## Step 0 — Load the method

Load the **`memolok-method`** skill. Rules A–G govern every turn of the interview, and Rule G governs
the head-sharpening phase specifically.

## Non-negotiables

- Never invent `mdlGuid`, `mdrHandle`, or `mdrNumber` — the server mints them
- All prose is `{ "markdown": "...", "lang": "en" }`
- **No writes until the user confirms a fish preview** — reads are fine throughout
- Default persist status is **Deliberating**; t₀ only on explicit request (Rule C)
- Say "captured" only after a write succeeded — mid-interview, say *drafting* or *got it*
- **Never offer to end the session**

## Core invariants

Interview the user relentlessly about the topic until you reach shared understanding. Walk each branch
of the decision tree, resolving dependencies one at a time.

**One question per turn.** Several at once is bewildering and gets you shallow answers to all of them.

**Look things up rather than asking.** If a fact is discoverable — the filesystem, the ledger, the
repository, other tools — go and find it. The *decisions* are the user's; the *facts* are your job.

**Do not act until they confirm.** No writes and no implementation work until the current fish is
confirmed.

### Recommended answers

Offer a specific recommended answer whenever the context reasonably supports one. Two constraints: the
user has final agency on everything, and you are not obliged to have a view on everything.

Help where you genuinely can. Do not manufacture a recommendation to fill the slot.

Where the ledger states a purpose, recommendations should sit comfortably with it — that is most of
what reading it buys you. But it informs your suggestions only; it never constrains the user's answers.
If they go somewhere the stated purpose did not anticipate, follow them there and record it. Do not
point out the mismatch, and never ask them to justify it.

## Session loop

```
grill one fish → preview → confirm → persist at Deliberating → next fish → …
```

t₀ is separate, and only when the user asks. **After a seal the session does not end.**

**Continue by default.** After each fish, move straight into the next branch — usually an open question
from the fish just logged, or the next dependency in the tree. Open it with a normal question turn, not
a menu.

**The user ends the session.** Never offer an exit, and never ask whether they want to continue.

> *"Want to open a new fish for that open question — or call it here for today?"*

That is continue-versus-end theatre. The user knows how to say they are done.

## Setup — read only

1. Establish the ledger: the `mdlGuid` in play, else `.memolok/mdl.yml`, else `get_MDLs`. If there are
   several, one question to pick.
2. `get_MDL` — **before the first question.** The ledger's stated purpose tells you what this ledger is
   driving at and which branches are likely to matter, which is worth several questions you now do not
   have to ask. This is the *look things up rather than asking* rule doing its job: never spend an
   interview turn asking what the ledger is for when a read answers it.
3. `list_MDRs(query="...")` — surface related prior decisions in the user's own words. Flag
   duplicate head **Claim**s before grilling a question that is already settled. Search is lexical,
   so a record that settled this in different words will not surface: absence here is weak evidence,
   and worth one question rather than an assumption.
4. `list_matters(untaken: true)` — parked bait may already cover this topic, and it is
   better raised now than re-elicited.

Ledger reads are delegated — the **reading invariant** in **`memolok-method`**. Read inline instead
and the pages land in the practitioner's own context, where they cannot be taken back out. That
covers steps 3 and 4, **in preparation only**: `get_MDL` is one call and stays inline, and once the
interview is running a spawn is a stall in a conversation whose whole value is rhythm. Do the reading
before the first question, not between two of them.

**Allowed throughout:** every read tool, plus the filesystem and anything else that answers a factual
question.

**Forbidden until preview confirm:** `register_matter`, `create_analysis`, `create_MDR`,
`update_MDR`, `transition_MDR_status`, `admit_world_fact`, `record_observed_outcome`.

Hold the draft in conversation until then.

## The question tree

Walk the fish regions in dependency order. **Draft from what the user already said before asking.**
Skip a region, or reduce it to a confirmation, when the conversation already contains it — especially
the belly.

| Phase | Region | Focus | Gate |
| --- | --- | --- | --- |
| 1 | Entry | Symptom, or a target they can already state? | Raw pain preserved if it is bait |
| 2 | Head | Sharpen the Need — falsifiable, mechanism-free | User accepts it *(provisionally — see below)* |
| 3 | Belly | Options they already weighed | Explored options drafted |
| 4 | Belly | Their actual arguments on those options | Key reasoning drafted |
| 5 | Waist | Verdict and chosen alternative, or the reason not to proceed | Non-empty Verdict |
| 6 | Tail | Expected outcomes — gains, costs, risks, dependencies | Drafted |
| 7 | Alongside | What this decision does *not* settle | Scope boundaries named |
| 8 | Persist | Default Deliberating | Preview confirmed |

Phase 1 is the intake fork — the full five-way routing lives in **`record-decision`**, and it applies
here unchanged.

Full phase-by-phase detail, turn formats, and worked openers: `references/question-tree.md`.

### The head gate is provisional

"User accepts the sharpened Claim" moves you from phase 2 to phase 3. It does **not** freeze the Need.

Belly and waist work routinely reveal that a Need confirmed three turns ago quietly contains a
mechanism that was only discovered later. When that happens, **walk the Need back** and move the
specific content into the alternatives or the Verdict. That is the process working, not a reopening —
nothing is sealed until t₀ regardless.

### Turn format

```
**Question:** …
**Recommended answer:** …
**Why:** … (brief — tie it to the region you are in)
```

Mid-interview acknowledgements use *drafting*, *progressing*, or a plain *got it*. Never "captured"
before a write.

**Phase 2 is a special case.** A recommended answer for a Need question is a recommended **scope or
framing** — never a mechanism, technique, or state count, even if one has already come up. If your
recommendation would only have made sense *after* an alternative was floated, it belongs in phase 3,
not folded into the Need.

## Preview and persist

Present the fish — head, alternatives, deliberation, verdict, expected outcomes, open questions,
intended status — and ask whether it covers what was discussed. Wait for an explicit yes. If no, return
to the region that was wrong.

Format: the `fish-preview.md` reference in **`record-decision`**.

Then persist through **`record-decision`**, at **Deliberating**. Do not offer Proposed or Accepted in
the preview footer unless the user has already raised sealing.

If the user asks to seal, that is **`commit-decision`** — ceremony first.

## After each fish — keep going

1. One beat of acknowledgement. Not a wrap-up.
2. Pick the next target: an open question from that fish, the next unresolved branch, or an adjacent
   area worth probing.
3. Open the next question tree immediately.
4. Do not ask whether to stop.

## Anti-patterns

**Asking for facts you could look up.** *"What database are you using?"* when the repository is right
there wastes the user's turn on something you could have answered.

**Batching questions.** Five at once gets five shallow answers.

**Staging the belly.** *"Now we move to alternatives — what other approaches could you take?"* turns
recording into form-filling and pushes the user to invent options for the ledger.

**Recommending a mechanism during head sharpening.** Phase 2's recommended answer is about framing. A
mechanism there is Rule G failing in advance.

**Treating open questions as homework.** They are scope boundaries. Name them and move on.

**Offering an exit.** The single most common failure in this mode.

## Tips

- Grill-me is a mode, not a separate ledger path. Everything persists through the ordinary skills.
- One fish per confirmation. If a second decision surfaces mid-interview, note it and open it as its
  own fish — do not fold two decisions into one body.
- If the user answers a question with a decision they had already made, record it as such. You are
  documenting their process, not running a workshop.
- If the ledger already answers a question, say so instead of asking it.

## References

| File | Load when |
| --- | --- |
| `references/question-tree.md` | Running the interview — phase detail, turn formats, worked openers |
