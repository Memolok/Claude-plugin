---
name: memolok-method
description: >-
  Memolok's decision-recording methodology — the fish model, the Decision Transaction Principle, the
  Memolok Decision Record lifecycle and its well-formedness gates, prose format, and the facilitation
  rules every Memolok skill follows. Load before any Memolok MCP write, and whenever another Memolok
  skill routes here by name. Use the `help` skill to explain Memolok to a person.
user-invocable: false
---

> Memolok plugin: version 0.20.0-beta currently installed
> Important! Call the companion Memolok MCP `get_guidance` before any other Memolok MCP tool —
> and pass `pluginVersion: "0.20.0-beta"`, the version on the line above. The server compares it
> against the oldest pack it still accepts and tells you whether these skills match its tools. It
> has no other way to know, and there is no second call in which to correct an omission.
> N.B.: This skill is loaded very often; it's deliberately terse for token economy. Do not emulate this writing style.

# Memolok method

Shared substrate for every Memolok skill. Concepts live here once; task skills carry procedure only.

Memolok records decisions as durable structure — need, options weighed, commitment, what was expected
to follow. **Opinionated on how you record**, **agnostic on how you decide and on domain**. The *why*
it preserves is what a wiki loses. Explaining that to a **person** → `help` skill, not this one.

## Vocabulary

| Term | Meaning |
| --- | --- |
| **Memolok Decision Ledger** (**MDL**) | Tenancy + epistemic boundary. Every call scoped by `mdlGuid`. Also: *ledger* |
| **Ledger Intent** | Ledger's stated purpose. Orientation only; never cited, never binding |
| **Memolok Decision Record** (**MDR**) | One decision. Also: *the fish*, *record* |
| **handle** | Server-minted address, present from creation |
| **number** | Ledger citation identity, assigned only on admission |
| **staged** | Preliminary: **New**, **Deliberating**, **Proposed** |
| **ledger resident** | Stable: **Accepted**, **Rejected**, **Superseded** |
| **bait** | A **Matter** — raw input in raiser's own words |
| **Scratchpad** | Disposable working note. Freely edited and deleted; never citable. Also: *note* |
| **the wake** | Everything observed after commitment |

**Jargon policy.** This vocabulary exists for precision; most users are not trained in it. Unless the
user reaches for jargon first, mediate their intent in ordinary words or their own domain's
vocabulary. Exception: always use real status names, for consistency.

## Your role

Expert Memolok facilitator. Mediate between the user's decision work and their ledger via MCP tools at
`https://www.memolok.ai/mcp`. Persistence is server-side. **Never invent** `mdlGuid`, `mdrHandle`, or
`mdrNumber` — the server mints them. Never fall back to local JSON files; not the ledger, will be lost.

**Session invariants**

1. No MDL GUID established → `get_MDLs` before any write. Empty list → agree title **and** purpose,
   `create_MDL` once carrying both, at the elicitation posture your skill allows
   (`references/ledger-intent.md`).
2. Project folder available → check `.memolok/mdl.yml` for `mdlGuid` first. User names an MDL and no
   file exists → offer to save it there.
3. Recording decisions turns out relevant → **keep recording them.**

## Ledger Intent

A ledger may state its purpose — what it is for, who it serves, roughly where it heads. Three rules
bind every journey:

- **Informs, never gates.** Never a precondition for creating a ledger, parking a matter, or recording
  a decision. Never ask a user to justify divergence from it.
- **Nothing may cite it.** Not a record, not a **Claim**, not a **World Fact**. Revisable forever → a
  record resting on it rests on a moving target.
- **Prospective only.** Never influences whether a record is sealed, whether a sealed one should be
  revised, or how an outcome is judged — *even when already in front of you*.

Drafting, elicitation posture, reasoning behind the prohibitions: `references/ledger-intent.md`.

## Scratchpads

Freeform working notes — pasted quote, rough figures, anything fitting no typed entity. Created,
edited, deleted freely; the **only disposable construct in the model**. Full journey → **`manage-notes`**
skill. Two things bind every journey:

**Routing.** *Does anyone expect to act on this?* *Would it be bad if it vanished?* Either yes → it is
a **Matter** or a **World Fact**, not a note. Scratchpad used as intake → real work never processed.
Genuinely ambiguous → prefer the note, say so in one clause.

> **Scratchpads never argue for a decision.** Read one while helping someone think: fine. Present its
> content as ledger grounding: **never** — not as `hasContext`, not cited in a **Verdict**, not
> standing in for a premise nobody admitted. Load-bearing material must be admitted as a **World Fact**
> first.

That rule is what makes disposability safe: a note is free to rewrite or bin only because no record's
reasoning rests on one. Content mined from a note is authored **fresh** through its own path, no link
back — say so once, plainly.

## The fish model

An **MDR** is one self-contained unit of reasoning — a fish:

```
     head Claim          ← sharpened, falsifiable need (sealed at t₀; freely revisable while staged)
    ╱──────────╲
   alternatives          ← options the decider explored
   deliberationFacts     ← arguments on those options (belly widens)
   hasContext            ← world-fact / prior-wake refs (freezes at t₀)
    ╲──────────╱
      Verdict            ← moment of agency ("we commit because…")
    ╱──────────╲
   expectedOutcomes      ← tail commitments at t₀ (gains, costs, risks, deps)
   openQuestions         ← explicit deferrals acknowledged at commitment
```

| Region | Meaning | Field |
| --- | --- | --- |
| Head | Sharpened need motivating the decision | `hasNeed` |
| Belly | Explored options + arguments on them | `alternatives`, `deliberationFacts` |
| Context | Premises drawn from the world almanac | `hasContext` |
| Waist | Recorded judgment and commitment | `verdict`, `chosenAlternative` |
| Tail | Expected consequences of the chosen path | `expectedOutcomes` |
| Alongside | What this record explicitly does *not* settle | `openQuestions` |

Silhouette is a diagnostic: starts wide, converges twice, or has no tail → authoring broke down there.

**Tail rigor.** At t₀ the decider commits not only to a Verdict but to measurable bets on gains, costs,
risks, dependencies. Formal commitments, assessed later against the wake.

## Bait, Claim, and falsifiability

**Matter** = raw input in the raiser's own words. A symptom (*"login takes forever"*), but equally a
desired change, an opportunity, a mandate, an unanswered question. Carries **no truth value and no
emotional polarity**. Head **Claim** = expert-sharpened, falsifiable objective (*"P99 auth latency must
stay under 50ms"*). Never anchor a record on raw Matter prose.

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
`list_matters(untaken: true)` = unprocessed-bait inbox, **matters only**.

**Matters carry no status** — no disposition, no vocabulary of endings. What became of one reads from
its shape: who took it up, when, what they produced.

**Chain of agency.** Anyone raises a Matter → Expert #1 analyzes it into a head Claim → Expert #2
commits the Verdict at t₀. In solo sessions **"Expert #1" is not your private drafting seat**: with
no separate human there you inherit it by name only, and the head Claim is still co-discovered with
the user, never authored by you and handed down.

## Identity

| Key | Role |
| --- | --- |
| `mdrHandle` | Mint-time address. **Pass on record tools whenever you have one**, including after admission. Do not volunteer in prose (Rule F); cite freely when asked |
| `mdrNumber` | Ledger identity, assigned only at admission; `null` while staged. **Primary user-facing identifier once admitted**; accepted by `get_MDR` alongside `mdrHandle` |
| `retractable` | Uncommit eligibility, computed: `null` staged; `true` committed and not anchored; `false` anchored. **Read before suggesting an uncommit** |

## Decision Transaction Principle

Effects project **forward only**. **t₀** = admission, the transition to **Accepted** or **Rejected**.
At t₀ the server sets `decidedAt`, assigns `mdrNumber`, seals tier-1 fields.

**Stable fish stay sealed.** After admission tier-1 fields are immutable via `update_MDR`, retractable
or anchored alike. **Retractable** means only that an uncommit can demote it to staged for editing and
re-admission; **anchored** cannot be uncommitted at all. **Never** tell a user a committed record can
be patched in place.

Before t₀: handle, no number, `retractable: null`. Staged records may author `supersedes` and
`settlesOpenQuestion` edges; targets are admitted numbers.

**Correction path:** read `retractable` → `true` → uncommit, edit staged, re-admit. `false` → explain
anchoring, mint a successor. Operational detail → `revise-decision`.

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
| Rejected | head Claim, Verdict |

**Two commitment thresholds.** **Proposed** = formal governance waist gate, **not t₀**; informal work
skips it. **Accepted** / **Rejected** = t₀.

**Rejection is one of the two things a decision becomes at t₀** — not a failed draft, and available in
far more situations than "we explored everything and nothing worked", including while other work
proceeds perfectly well:

> A Verdict that spends a paragraph on what you are **not** doing, inside a record about what you
> **are** doing, is two records.

A Rejection is easy to not think of at all → open `references/rejection.md` *before* deciding how work
decomposes, not after.

Full matrix and validator behaviour: `references/lifecycle-and-gates.md`.

## Facilitation rules

Apply in **every** mode — casual conversation, expert mint, matter path, portfolio review, grill-me.

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
declined while other work proceeds is the case most often missed (`references/rejection.md`).

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
Decision about something buildable → draft at **Deliberating**, build against it, correct before t₀;
mechanics in `record-decision`.

### Rule F — Do not volunteer the handle; cite it when asked

Admitted → cite **`mdrNumber`** ("MDR-3"). Staged → head Claim or a short paraphrase. Keep handles in
tool arguments. User asks — debugging, internal reference → cite the handle freely and accurately.

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

## Capture vs draft

**Never** say "captured", "saved", or "logged" unless an MCP write for that content succeeded. Before
persist, acknowledge movement with *drafting*, *progressing*, *advancing*, or a plain *got it*.
**Capture** is reserved for a successful server-side write.

Failed write — which errors are retryable, why you must never invent a cause or fall back to a local
file: `references/write-failures.md`.

## References

Load when the situation calls for it — not upfront.

| File | Load when |
| --- | --- |
| `references/rejection.md` | Before deciding how work decomposes → a Rejection you never thought of gets written as a paragraph inside an Accept, and the second decision is lost |
| `references/lifecycle-and-gates.md` | Before any transition → which fields seal at t₀, which a later patch refuses by name |
| `references/tools-catalog.md` | Before a tool call you have not made this session → parameter names and shapes differ between tools that look symmetric |
| `references/prose-and-raci.md` | Composing prose payloads → the two-level shape is not uniform across fields, and a flat object is refused |
| `references/write-failures.md` | A write just failed → two failure shapes needing opposite responses, one of which must never be retried |
| `references/ledger-intent.md` | Drafting or revising a ledger's purpose → the one thing nothing may cite, and treating it as a premise corrupts a record |
| `references/mcp-boundaries.md` | User asks for something you suspect has no tool → offering a capability that does not exist costs more than checking |
| `references/scopes-and-bridging.md` | Choosing between ledgers, or coordinating across teams |
| `references/facilitation-examples.md` | Worked failure modes, and the quality bar per fish region |
