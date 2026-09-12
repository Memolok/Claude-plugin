# Facilitation — the fish, the transaction, the lifecycle, Rules A–E and G

> N.B.: This file is loaded by every skill that writes a record; it's deliberately terse for token economy. Do not emulate this writing style.

## The fish model

An **MDR** is one self-contained unit of reasoning — a fish:

```
     head Claim          ← sharpened, falsifiable need; `hasNeed` (sealed at t₀; revisable while staged)
    ╱──────────╲
   alternatives          ← belly: options the decider explored
   deliberationFacts     ← belly: arguments on those options (belly widens)
   hasContext            ← context: world-fact / prior-wake refs (freezes at t₀)
    ╲──────────╱
      Verdict            ← waist: moment of agency ("we commit because…"); `verdict`, `chosenAlternative`
    ╱──────────╲
   expectedOutcomes      ← tail: measurable bets at t₀ — gains, costs, risks, deps; assessed later
                            against the wake
   openQuestions         ← alongside: what this record explicitly does not settle
```

Silhouette is a diagnostic: starts wide, converges twice, or has no tail → authoring broke down there.

## Falsifiability and analysis

**Falsifiable, precisely:** a Need is falsifiable if and only if some future admission — an observed
outcome tested against a resulting expected outcome, or a corrected world fact — could confirm or
refute whether the chosen path satisfied it. Property of the **Need's own testability**, not of how
specific the chosen mechanism is.

**Analysis** = first-class bridge from raw input to sharpened need, preserving provenance:

| Path | `producesDecision` | Outcome |
| --- | --- | --- |
| **A — mint** | `true` + `claimDescription` | **New** record(s); analysis concludes in the same call |
| **B — honest dismissal** | `false` | No record; rationale explains why |

`motivatedBy` = **list**: every input taken up. Input = **Matter | WorldFact | ObservedOutcome**, any
mix; cite the entry itself, never a Matter restating it — that loses the link, unrepairably.
Fan-in/fan-out independent — *n* in, *m* out. **Never ask which record answers which input**;
ordinarily no such fact, none to invent. Input recognized later → `attach_analysis_reference`
(post-conclusion, dated, reads as late).

**Expert path** bypasses analysis: `create_MDR` with a claim. Nothing on a record says which path
minted it. Intake routing → `record-decision`. Parking unanalyzed → `save-matter`;
`discover_matters(untaken: true)` = unprocessed-bait inbox, **matters only**, answered as prose
carrying each matter's summary and subjects, so you can weigh candidates without opening one.

**Matters carry no status** — no disposition, no vocabulary of endings. What became of one reads from
its shape: who took it up, when, what they produced.

**Chain of agency.** Anyone raises a Matter → Expert #1 analyzes it into a head Claim → Expert #2
commits the Verdict at t₀. In solo sessions **"Expert #1" is not your private drafting seat**: with
no separate human there you inherit it by name only, and the head Claim is still co-discovered with
the user, never authored by you and handed down.

## Decision Transaction Principle

Effects project **forward only**. **t₀** = admission, the transition to **Accepted** or **Rejected**.
At t₀ the server sets `decidedAt`, assigns `mdrNumber`, seals tier-1 fields.

**Stable fish stay sealed.** After admission tier-1 fields are immutable via `update_MDR`, retractable
or anchored alike. **Retractable** means only that an uncommit can demote it to staged for editing and
re-admission; **anchored** cannot be uncommitted at all. **Never** tell a user a committed record can
be patched in place.

Before t₀: handle, no number, `retractable: null`. Staged records may author `amends`, `supersedes`,
`dependsOn`, `conflictsWith` and `settlesOpenQuestion`; targets are admitted numbers.

**Correction path** turns on how much is wrong, not on `retractable`: nothing, it should never have
said that → uncommit (needs `true`); part of it → `amends`, original stays **Accepted**; all of it →
`supersedes`. **Amend keeps everything you don't change; supersede keeps none of it** — so an
amendment states deltas only, and restating the original is not amending it. Both need `Accepted` at
each end. Amending Anchors what it amends, permanently, and reads gentler than it is. Detail →
`revise-decision`.

Value is **honesty, not correctness.** A well-reasoned decision that failed teaches more than one
retrofitted to look prescient. Wake evidence showing a violated commitment → re-decide at a new t₀; the
wake becomes bait for the next fish.

## Lifecycle

| From | To |
| --- | --- |
| New | Deliberating, Proposed |
| Deliberating | Proposed, Accepted, Rejected |
| Proposed | Accepted, Rejected |
| Accepted, Rejected, Superseded | *terminal for transitions; retractable records may be uncommitted* |

**Superseded** is never a transition target — a record reaches it when another admits carrying
`supersedes`.

| Target | Requires |
| --- | --- |
| Deliberating | head Claim |
| Proposed | ≥1 alternative, `chosenAlternative` among their ids, non-empty Verdict |
| Accepted | the above, plus ≥1 expected outcome |
| Rejected | head Claim, Verdict; carries no `amends`, `supersedes` or `settlesOpenQuestion` |

**Two commitment thresholds.** **Proposed** = formal governance waist gate, **not t₀**; informal work
skips it. **Accepted** / **Rejected** = t₀.

**Rejection is one of the two things a decision becomes at t₀** — not a failed draft, and available in
far more situations than "we explored everything and nothing worked", including while other work
proceeds perfectly well:

> A Verdict that spends a paragraph on what you are **not** doing, inside a record about what you
> **are** doing, is two records.

A Rejection is easy to not think of at all → open `rejection.md` *before* deciding how work
decomposes, not after.

Full matrix and validator behaviour: `lifecycle-and-gates.md`.

## Facilitation rules

Apply in **every** mode — casual conversation, expert mint, matter path, portfolio review, grill-me.
Rule F, naming records, is in the method's core and applies to reads as well.

### Rule A — Discussion is deliberation; default persist status is Deliberating

Shaping a fish *with* the user is deliberation. Unless formal governance applies (Rule B): do not mint
or transition to **Proposed**, and do not offer **Accepted** or **Rejected**, without explicit user
commitment (Rule C).

**New** is exceptional — external import and analysis Path A only, never a conversational outcome. A
complete fish body may sit at **Deliberating**; completeness is not commitment. Never infer commitment
from agent-authored Verdict prose, a firm-sounding draft, or a confirmed read-back.

### Rule B — Proposed is the formal governance path

| Posture | Signals | Path |
| --- | --- | --- |
| **Informal** (default) | Solo decider, hobby, small team without ratification | Deliberating → t₀; **skip Proposed** |
| **Formal** | Distinct RACI deciders, committee vote, compliance sign-off | Deliberating → Proposed → t₀ |

Infer posture from context. Unclear → **default informal**, and do not surface **Proposed** in
user-facing choices.

### Rule C — t₀ is a separate, explicit ceremony

Two steps, never collapsed: **persist / recap** at Deliberating → **commit**, only when the user
explicitly asks to seal.

**Narrow exception:** one-shot mint at **Accepted** or **Rejected** when the user has unambiguously
stated commitment in their own words ("accept this", "lock it in", "reject this path") — never because
you drafted a convincing Verdict.

Rejection deserves equal facilitation, in more situations than an exhausted deliberation — a Claim
declined while other work proceeds is the case most often missed (`rejection.md`).

### Rule D — Open questions keep scope tight; they do not block commit

Open questions name what a record deliberately leaves unsettled — *"we have not validated vendor B's
SLA under peak load"*. Scope boundaries acknowledged at commitment, not unfinished deliberation.
Settlement requires a **later Accepted** record, never an edit to this one.

**Default posture:** something falls outside this Claim → record it as an open question and proceed.
Prefer a correctly scoped Accepted record with open questions over scope creep. **Never** treat a
remaining open question as a gate.

### Rule E — Record the explored process; do not stage theater

Record the **user's** decisional process; do not run a workshop that invents a fuller-looking belly.

**Draft first.** Mine what the user already said or implied — favoured path, discarded options, "vs
doing nothing", constraints that ruled things out. Never invent options they did not explore.

**Probe lightly, only for honesty gaps.** One short check suffices when a single favoured path has no
acknowledgment of what else was in play. Not a gate. **Never stage the section** — no "now we move to
alternatives" as form-filling.

**Deliberation is not only conversational.** Building a spike, reading the implementation, probing a
running system: real exploration, and what they establish belongs in the belly exactly as spoken
reasoning does. The prohibition is on options nobody explored, wherever the exploring happened.
Decision about something buildable → draft at **Deliberating**, build against it, correct before t₀,
**then** write the references in — a staged record has no number, so nothing you build can cite the
decision it implements, and sealing early to buy one spends the correction window. Build loop →
`record-decision`; the reference pass and the anchoring before it → **`memolok-citations`**.

### Rule G — The Need is not a running summary

Rule E guards the belly against over-*invention*. This guards the head against over-*enrichment* with
content that exists only because of downstream deliberation.

**Proof analogy.** Need = hypothesis, Verdict = conclusion, belly = the proof connecting them. Two
symmetric failures: **over-sharpening** smuggles the conclusion into the hypothesis, making everything
downstream circular; **under-sharpening** leaves a hypothesis with no determinate truth conditions.

**The test:** would this clause have been askable, word for word, before any alternative was discussed?
A mechanism, technique, or scope boundary that became known only through exploring alternatives belongs
in the belly or the Verdict — **no matter who said the words**.

The Need is expected to be **walked back** as belly work proceeds; de-sharpening an over-specific Claim
is correct, not a failure to hold a line. Caught the conflation after already restating it once →
re-derive from *"what did we know before we explored anything?"*; it survives light rewording.
