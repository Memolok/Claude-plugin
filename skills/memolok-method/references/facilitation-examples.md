# Facilitation failures and field quality

## Anti-patterns

**Never infer commitment from your own draft:**

> *"Want me to mint this straight to Accepted (it reads like a firm commitment), or hold it at Proposed?"*

Infers commitment from agent prose, offers **Proposed** on informal work, bundles t₀ into recap.

**Never stage the belly:**

> *"Once you confirm, we move to the belly: alternatives. What are the other ways you could pursue this?"*

Treats recording as form completion and pushes the user to invent options for the ledger.

**Never imply a write that did not happen:**

> *"Captured — and that reasoning ties the two threads together nicely."*

Only wrong when no write just succeeded. After a successful write the same wording is correct.

**Never block on open questions:**

> *"Before we can Accept, we should close these open questions…"*

Log them, seal the scoped decision, settle later in a new record.

**Never settle in advance:**

> *"MDR-3 is admitted now, so I can link it back to settle that open question on your staged record."*

Settlement retroactively closes a **frozen** question on an **admitted** holder. A staged holder still
has live, editable questions and no number to target. Update the holder in place instead.

**Never volunteer internal handles:**

> *"Saved — mdrHandle 2 is now Deliberating."*

Cite **MDR-{n}** when admitted, or the head Claim while staged.

**Never smuggle the Verdict into the Need:**

> Head Claim: *"We needed to choose Python because it has excellent statistical analysis capabilities"*

"Python" was an alternative just proposed, not something true before deliberation. Keep the Need at
*"a language must be chosen for statistical analysis, with a mature, well-supported library
ecosystem"* — still falsifiable, still mechanism-neutral — and move the choice to the belly and
Verdict.

## Field quality

**Head Claim.** Bad: *"we need better performance."* Good: *"P99 API latency must stay under 200ms at
500 concurrent users."*

**Sharpen ≠ resolve.** When the honest Need is mechanism-agnostic, a sharp Need stays that way:

- Over-sharpened: *"…the project must use Python for its mature pandas/numpy/scipy stack."*
- Correctly sharp: *"…the chosen language must have a mature, well-supported statistical-analysis
  library ecosystem."*

The second is falsifiable — a later observation can confirm or refute whether the ecosystem held up —
without pre-committing to which language.

**Alternatives.** `label` is a short handle; `description` is the option substance the user
articulated. Arguments belong in `deliberationFacts`, not here.

**Deliberation facts.** Pair each with `onAlternative`. Prefer arguments the user actually made over
authored pros-and-cons.

**Expected outcomes.** Measurable tail commitments as prose — gains, costs, risks, dependencies.

**Open questions.** Deferral prose, not category labels. *"Should we transcode for mobile?"* is valid
ledger prose on an Accepted fish.

**Prose fields.** Always `{ "markdown": "...", "lang": "en" }`, including LaTeX via `$...$` and
`$$...$$`. Detail in `references/prose-and-raci.md`.

