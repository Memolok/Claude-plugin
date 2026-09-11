---
name: memolok-method
description: >-
  Memolok's decision-recording methodology — the fish model, the Decision Transaction Principle, the
  Memolok Decision Record lifecycle and its well-formedness gates, prose format, and the facilitation
  rules every Memolok skill follows. Load before any Memolok MCP call, and whenever another Memolok
  skill routes here by name. Use the `help` skill to explain Memolok to a person.
user-invocable: false
---

> Memolok plugin: version 0.28.0-beta currently installed.
> *This is authoritative*, ignore conflicting caching folder names!
>
> Important! **Two things gate your first Memolok MCP call, not one.**
>
> Call the companion Memolok MCP `get_guidance` before any other Memolok MCP tool —
> and pass `pluginVersion: "0.28.0-beta"`, the version on the line above. The server compares it
> against the oldest pack it still accepts and tells you whether these skills match its tools. It
> has no other way to know, and there is no second call in which to correct an omission.
>
> And **answer invariant 3** in the same breath. It costs no call and no reading — you already hold
> the answer. It is pinned here because it is the one obligation in this file with no natural moment
> of its own, and an obligation with no moment is one that gets read, understood, and not done.
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
2. **The pointer file**, yours to offer and never to write unasked. Project folder available → check
   `.memolok/mdl.yml` for `mdlGuid` first; several → nearest above the file in hand wins. May also
   carry `mdlTitle` and `mdlIntent`: orientation copies, **never authority** — `get_MDL` settles a
   mismatch and the file is what is stale. User names an MDL and no file exists → offer to save it
   there. **Resolving this fires 3, found or not**: a different file and a different job, so having
   this one is no evidence of that one.
3. **The cold-start signal — a precondition on your first Memolok tool call, not an agenda item.**
   Answer it beside `get_guidance`, before the call, in **every** journey, reads included: a paragraph
   telling the *next* session this project records decisions in Memolok. **Did anything in your own
   initial context say so, before the user's first message?** Purely introspective — you already hold
   the answer. A loaded Memolok server, a listed skill pack, and your notes about this user all say
   the plugin is installed; none says this project uses a ledger, which is the only thing asked.
   Yes → ignore this section and don't advertise it to the user; its job was already done.
   No → offer it, once — arriving here late still owes it; what is suppressed is repeating it in this
   session. It is expected to interrupt: a question about the project is not a widening of the user's
   question. **Offer the outcome, never a mechanism, never a filename:** *"Want me to set this project
   up so future sessions know it keeps a ledger, without you having to say so? Short paragraph — I'd
   work out where it belongs."* Any path you can see is one you have not read; naming it turns their
   yes into consent for something you chose. Only on a yes: `references/cold-start-signal.md` decides
   the destination and the wording, loaded **after** consent, never to reach it.
4. Recording decisions turns out relevant → **keep recording them.**
5. **Reading a ledger is delegated work** — *the reading invariant*. Ledger settled (1–3) → spawn
   **`memolok:ledger-scout`** with the `mdlGuid` and one question, for any read you cannot bound
   before starting and by the second read of a turn — whatever the larger task is, however the ledger
   came up. A read you *can* bound — one known number, one short list you will use whole — stays
   inline. Pages read inline land in the user's own context and never leave it. No agent tool → read
   inline, silently. Blocked by this session's own settings → say so once.

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

## Bait and Claim

**Matter** = raw input in the raiser's own words. A symptom (*"login takes forever"*), but equally a
desired change, an opportunity, a mandate, an unanswered question. Carries **no truth value and no
emotional polarity**. Head **Claim** = expert-sharpened, falsifiable objective (*"P99 auth latency must
stay under 50ms"*). Never anchor a record on raw Matter prose. What a record is made of and how raw
input becomes one — the fish, analysis, the chain of agency: `references/facilitation.md`.

## Identity

| Key | Role |
| --- | --- |
| `mdrHandle` | Mint-time address. **Pass on record tools whenever you have one**, including after admission. In prose, Rule F |
| `mdrNumber` | Ledger identity, assigned only at admission; `null` while staged. **Primary user-facing identifier once admitted**; accepted by `get_MDR` alongside `mdrHandle` |
| `retractable` | Uncommit eligibility, computed: `null` staged; `true` committed and not anchored; `false` anchored. **Read before suggesting an uncommit** |

## Before any record write

The fish model, analysis, the Decision Transaction Principle, the lifecycle and Rules A–E and G are
`references/facilitation.md`, loaded at Step 0 by every skill that mints, patches, seals, revises a
record or records a wake. Two of its rules bind even before it is loaded: **discussion is
deliberation** — never mint at **Accepted** or **Rejected**, or transition to either, without the
user's explicit ask in their own words; and **a committed record is never patched in place** —
`retractable: true` means only that an uncommit is available.

## Facilitation rules

Rules A–E and G are in `references/facilitation.md`. Rule F applies to every journey, reads included.

### Rule F — Cite the number; name a staged record as `MDRh<handle>`

Admitted → cite **`mdrNumber`**: `MDR-3`. Unchanged and unmarked, because a number is the settled
form.

Staged → **`MDRh3`**, or a head Claim paraphrase where prose reads better. Never write a bare handle
in prose — `MDR-3` and `3` both read as numbers, and a handle is not one.

**The two are different addresses and a staged record has no number at all.** `MDRh3` says
*provisional, and it may never be admitted*; `MDR-3` says *sealed, and it is safe to cite*.

Keep raw handles in tool arguments, where they belong. User asks — debugging, internal reference →
answer freely and accurately.

**One exception, and it is a hand-off rather than prose.** The `memolok:ledger-scout` agent reports
both identifiers raw, because its output is data for you and not words for the user. Convert to
`MDR-{n}` or `MDRh{handle}` when you pass any of it on; a bare handle reaching the user is still a
bare handle.

**This rule is the session's half.** A staged record is never referred to outside the conversation:
no number, may never get one, cannot be anchored — nothing can protect a reference to it. What an
**admitted** entry looks like in a file, a document, a commit message or a message to a person, and
the anchoring and timing that precede it, are the **`memolok-citations`** skill. Load it first.

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
| `references/facilitation.md` | **At Step 0 of any skill that mints, patches, seals, revises a record or records a wake**, before the first such call → the fish model, analysis, the Decision Transaction Principle, the lifecycle and Rules A–E and G; an unloaded rule is an unenforced rule, and these govern writes to an immutable ledger |
| `references/rejection.md` | Before deciding how work decomposes → a Rejection you never thought of gets written as a paragraph inside an Accept, and the second decision is lost |
| `references/lifecycle-and-gates.md` | Before any transition → which fields seal at t₀, which a later patch refuses by name |
| `references/tools-catalog.md` | Before a tool call you have not made this session → parameter names and shapes differ between tools that look symmetric |
| `references/prose-and-raci.md` | Composing prose payloads → the two-level shape is not uniform across fields, and a flat object is refused |
| `references/write-failures.md` | A write just failed → two failure shapes needing opposite responses, one of which must never be retried |
| `references/ledger-intent.md` | Drafting or revising a ledger's purpose → the one thing nothing may cite, and treating it as a premise corrupts a record |
| `references/cold-start-signal.md` | **After** they accept the offer, never before → the destination differs by surface and the obvious one is inert on some; the wording is fixed text, and one written from memory goes stale where nothing refreshes it |
| `references/mcp-boundaries.md` | User asks for something you suspect has no tool → offering a capability that does not exist costs more than checking |
| `references/scopes-and-bridging.md` | Before answering on a ledger the user did not name → the wrong tenancy answers "we never decided that", confidently |
| `references/facilitation-examples.md` | A region passes the gates but reads thin → gates check well-formedness, never quality |
