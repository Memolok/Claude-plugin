---
name: wrap-up
description: >-
  Wrap up this interactive agent session by systematically saving what would be lost from
  the agent context into a durable place. Use when the user says "wrap up", "we're
  done here", "before I close this",  "end of session", or asks "any loose ends?",
  "anything else worth saving?", or is clearly preparing to finish up this work session.
argument-hint: "(nothing — the session is the input)"
---

# /memolok:wrap-up — Before the context is gone

This context is about to disappear, and with it everything the session established that was never
written down. Nobody will find out. That is *decision debt* at session scale.

You are the reason this skill exists. A human's amnesia is partial and slow; **yours is total and
immediate** — every session starts cold, with no memory beyond what the ledger and the project hold.
The user's is worse than they think: even in a solo project, the person reading this project
two weeks from now is effectively a different colleague, working from whatever you two leave behind today.

Memolok's premise is that skills mint ledger entries *from the conversation where the choice actually happened*
rather than from a form somebody fills in afterwards. This is that conversation, at the edge of "afterwards".

This skill teaches you **what to look for** — not what to do about it; you will have to use your judgement for that.
Every destination is owned by a skill or a tool that already knows its own rules. You decide, per item, with the user.

## Usage

```
/memolok:wrap-up
```

The session is the argument. Any text the user adds is a steer ("just the infra stuff"), not a scope
they have to supply.

## Non-negotiables

- **This skill writes nothing itself.** Every write goes through whatever owns that destination.
- **Never restate another skill's rules.** Name it and hand over. Two copies drift.
- **Record the durable admission, never the event.** Memolok is a witness: it records admissions about
  the world, not replicas of it. An entry already false when written is worse than no entry.
- **Never tidy reasoning.** See *Honesty over coherence* below. This is the one that looks like good
  writing and is actually the failure the ledger exists to prevent.
- **The output is writes plus an explicit dropped list.** A run that ends in a summary has failed,
  however good the summary is.
- Scope is what is in this context now. No watermark, nothing to look up — **this skill persists
  nothing about itself.**
- Say "captured" only after a write succeeded.

## The core: extraction, not triage

The question is never *is this worth keeping?* It is *what durable thing is this evidence of?*

`alice wasn't in adm` is disqualified for being true for ten minutes, not for being unimportant.
Behind it sits `/var/log/mail.log` is `root:adm 0640` — true before the session, during it, and after.
The fix did not create that fact; it revealed one. The event is bait. Ask what it is bait for.

Two questions, in order:

1. **What is the durable fact behind this?** No residue → nothing to record.
2. **Is it already recoverable from something that outlives the session — and at what cost?**

The second is a cost, not a yes/no, because the two kinds of content sit at opposite ends of MDR-O's
own dynamicity ordering — the fish sealed at t₀ is the least dynamic thing in the model, the world the
most.

| | **Observed** | **Reasoned** |
| --- | --- | --- |
| Comes from | The world: commands run, files read, systems probed | The participants: decisions taken, options rejected, constraints stated, judgments made |
| Recovery | Re-observe it | Re-run the whole exploration |
| Cost of loss | One command | The deliberation, possibly to a different answer |
| When noticed | Next time someone needs it | **Never** — the team simply re-decides |
| Bias | Record when a persisting artifact *depends* on it | **Record.** There is no cheap path back |

The asymmetry is the point. World facts look like findings and are the cheap class. Reasoned content
never looks like a finding — it is merely what the session was talking about — and it is the half with
no recovery path at all.

A decision settled in conversation and never recorded does not stay neutral. It becomes a **phantom
decision**: everyone glides over it as though it were resolved, and nobody can say who decided or why.
Preventing that is the entire product.

**Discard** is what remains when there is no residue, or the residue is already written somewhere that
persists. It is never a verdict that something was not worth much.

| Observed in session | Residue | Verdict |
| --- | --- | --- |
| `alice` wasn't in `adm`, fixed live | `mail.log` is `root:adm 0640` | **Record** — the runbook says grep it and never mentions `adm` |
| Read `redeploy.bash` to check its assumptions | It assumes a clean checkout | **Nowhere** — the script says so; read it again any time |
| Suite passed on the second run | The suite is flaky on first run | **Record** — nothing else knows this |
| User said one affected domain "is no biggie" | A judgment that scopes future work | **Record** — reasoned, unrecoverable |
| Container took ~40s to start | Cold start is ~40s | **Judgment** — record if a timeout depends on it |

## Honesty over coherence

When you extract reasoned content, you will be tempted to write the *clean* version — the reasoning as
it would have gone if everybody had been sharp, with the false starts filed off and the conclusion
made to look inevitable. Resist it completely.

A ledger's value is not correctness, it is honesty. A record showing a well-reasoned decision that
failed teaches something; a record smoothed to look prescient teaches nothing, because it is fiction.
The deliberation was messy, some options were rejected for unflattering reasons, and some of it was
guesswork — **that is the content**, not noise to be cleaned up on the way to storage.

Concretely, when persisting a decision this session produced:

- Keep the alternatives that were dismissed quickly, and *why* they were dismissed.
- Keep the constraint the user stated in passing, in something close to their words.
- Keep the uncertainty. If they committed while unsure, the record says they committed while unsure.
- Do not promote your own tidied restatement over what they actually said.

The user is writing for their future self, who needs to know what they were afraid of at the time.

## Where to look

Two sweeps. Not a procedure to execute in order — the two places durable content comes from, matching
the two rows above.

**Tool trace → mostly observed.** Walk the non-trivial commands run and files read. For each: did its
*finding* land anywhere durable, or was it reported in prose and abandoned? Unpushed commits,
permission surprises, version drift and tooling gaps hide here.

**User utterances → mostly reasoned.** Walk what the user said that exists nowhere else. Constraints,
preferences, corrections, options ruled out, calls made in passing, things decided while talking about
something else.

The mapping is not clean — reading an implementation produces reasoned conclusions, and users report
observations. Use both sweeps on everything.

## Routing

MDR-O sorts the world into four scopes, and they are the routing question: **is this about the fish,
the ledger, the project, or the world?** Each destination is owned by something else. Hand over; do
not reimplement.

**Ledger — MDR and MDL scope.** Sealed reasoning and the epistemic substrate around it.

| The item is | Destination | Owned by |
| --- | --- | --- |
| A choice made, with alternatives and a commitment | MDR | **`record-decision`**, then **`commit-decision`** |
| Something someone should look at, nobody has yet | Matter | **`save-matter`** |
| A premise decisions should rest on | World fact | **`manage-almanac`** |
| What actually happened after a sealed record | Observed outcome | **`record-outcome`** |
| Worth keeping, nobody owes it anything | Scratchpad | **`manage-notes`** |

**Project scope — the deliverable.** The repo and everything in it. The ledger is not a substitute for
it, and a fact the project needs belongs *in the project*.

| The item is | Destination |
| --- | --- |
| Operational knowledge someone on call will need | Runbook, README, docs |
| A constraint the code should enforce or state | Code, config, comment |
| The *why* behind work already done | Commit message |

**Outside both.** Neither scope covers these; be honest about that rather than forcing them in.

| The item is | Destination | Note |
| --- | --- | --- |
| Useful to you across sessions, not to the team | Agent memory | A tooling concern, not a ledger artifact |
| Friction with Memolok itself, hit this session | Memonos, the team behind Memolok | **`send-feedback`** — leaves the user's machine |
| Anything another connected tool owns | That tool | Its rules, not yours |
| No residue, or the residue is already written | **Nowhere** | Say so in the dropped list |

**Destinations are not exclusive.** One item may legitimately go to two places — a Memolok fault worth
reporting may also be worth a matter on the user's own ledger, and they serve different readers.
Propose both rather than picking.

Three calls that are easy to get wrong:

- **Project over memory** when anyone but you benefits. Memory helps you; a file in the repo helps
  whoever is on call at 3am. Prefer the file whenever both would work.
- **World fact over scratchpad** when a decision might rest on it. Nothing may ever cite a note — that
  is what makes notes disposable — so load-bearing material has to be admitted properly.
- **Memolok friction is easy to miss, because much of it not errored.** A call that returned 200 and
  did the wrong thing, or two calls that only disagree with each other, leaves no obvious failure to notice —
  and you are the only party still holding the evidence, since the user will not write it up and the
  next session starts cold. `send-feedback` already treats the end of a working session as its
  batching moment, so this is a handoff it expects, not an intrusion. It owns its own permission and
  preview rules — do not restate them.

That last one still sits as **one row, not a finale**. It earns its place on the merits; making it a
privileged closing prompt turns this skill into a feedback funnel and trains users to skip it.

## Workflow

### 1. Sweep

Both sweeps, silently. Do not narrate the sweep.

### 2. Extract

Per candidate, run the two questions. Write down the *residue*, not what happened. If you cannot state
a durable fact, there isn't one — a legitimate result, not a failure to try harder.

### 3. Propose, once

The whole list in one pass — item, residue, proposed destination. One batched proposal, not per-item
interrogation: the user is finishing up, and twenty questions is why they will not run this again.

> Six things here that live nowhere. Proposing: two world facts (`mail.log` permissions, model tag
> drift), one matter (no linter on the web service), one runbook edit, one memory. Dropping two —
> `redeploy.bash` assumptions (in the script) and the container timing (nothing depends on it).
> Change anything?

Let them add, remove and re-route. They will keep things you dropped; that is the safety net working.

### 4. Write

Through the owning skill or tool, one at a time. If a write fails, say which and stop — never claim a
capture you did not get.

### 5. Report

Writes, then drops. Nothing else.

> Captured: MDR-14, two world facts, one matter. Runbook updated with the `adm` requirement.
> Dropped, deliberately: `redeploy.bash` assumptions, container start timing.

## Failure modes

| Failure | Why it happens | Tell |
| --- | --- | --- |
| **Session summary** | "Wrap up" invites narration | Ends in prose nobody reads, and no writes |
| **Losing the reasoned half** | Decisions don't look like findings | Every write is a world fact or a file edit |
| **Narrative smoothing** | Tidy prose reads as good work | The record is cleaner than the session was |
| **Almanac spam** | Recording work instead of premises | Entries that read like commit messages |
| **Recording events** | Skipping the extraction step | An entry already false when admitted |
| **Rule duplication** | Being helpful about another skill's mechanics | This file explains how `send-feedback` previews |
| **Interrogation** | Routing item by item | The user stops running the skill |

Against almanac spam the guard is: **record the premises and the gaps, not the work — if it is
derivable from the diff, it belongs in the commit message.** Apply it to observed content only. Never
to reasoned content, which is cheap-looking and expensive to lose.

## Limits

- **Compaction is out of scope.** On a session long enough to have been summarized, some sweep material
  is already gone and this skill cannot tell. Expect it to underperform on exactly the long sessions
  where it is worth the most.
- **Nothing makes this fire.** It runs when someone types it. The most common failure by count will be
  sessions where it would have helped and nobody ran it.
- Running it twice in a session is fine and cheap — the second pass sees the first pass's writes in
  context and simply finds less.

## Tips

- A session with nothing to persist is normal. Say so in one line and stop; never manufacture
  candidates to look useful.
- Run it mid-session too. The trigger is "this would be lost", not "we are finishing".
- If the user starts reasoning about an item rather than routing it, that is a decision emerging —
  offer **`record-decision`** rather than filing their reasoning as a note.
- Working solo is not an exemption. A solo project is a team of one across time, and the future member
  of that team has no access to this context at all.
