# What the words mean

Plain definitions, with what you would probably have called the thing yourself. For *why* a term is the
word it is — the fish, the wake, matter rather than issue — see `why-these-words.md`.

## The two containers

| Term | Meaning |
| --- | --- |
| **Memolok Decision Record** (**MDR**) | One decision, complete: what it was for, what was considered, what was committed, what was expected. Sometimes called *the fish*, from its shape. You might have said "a decision record" or "an ADR" |
| **Memolok Decision Ledger** (**MDL**) | The collection those records live in, together with the facts they reason from. An epistemic boundary, not a folder. You might have said "the project's decision log" |
| **Ledger Intent** | A ledger's short statement of what it is for, who it serves, and roughly where it is heading. Rewritten freely; read for orientation before starting new work. No decision ever points at it |
| **Scratchpad** | A freeform working note on a ledger — no title, no category, no status. The one thing here that can be edited and deleted at will, which is exactly why nothing may ever cite one. You might have said "a sticky note" |

## The parts of a decision

Described by position, because the shape is the point: narrow at the need, wide through the argument,
narrow at the commitment, wide again at the consequences.

| Term | Meaning |
| --- | --- |
| **Claim** | A single statement that could be true or false. The **head Claim** is the sharpened need a decision exists to satisfy — *"authentication must complete in under 50ms"*, not *"login is slow"* |
| **Matter** | Raw input in the raiser's own words, before anyone analyzed it. A bug report, a complaint, an idea, an opportunity, an unanswered question. Carries no truth value and no polarity — it is not yet a claim about anything |
| **Analysis** | The expert step that turns raw input into a sharpened need, recorded with who did it and what they concluded. Its inputs are matters, world facts and observed outcomes, in any mix. It may also conclude that no decision is warranted |
| **World Fact** | Something true regardless of this decision — the budget, the team's skills, a regulation. Lives in the almanac and is referenced, not copied in; a decision can cite it as context, and an analysis can take it up as the input that prompted the work |
| **Alternative** | An option that was considered. Includes the ones that lost, which are usually the informative ones |
| **Deliberation Fact** | One argument about one option, for or against |
| **Verdict** | The moment of agency: what was committed and why. Not the same as which option won — it is the reasoning that closed the deliberation |
| **Expected Outcome** | Something you commit to expecting at the moment of decision: a gain, a cost, a risk, or work it forces. A bet, not a hope |
| **Open Question** | Something this decision explicitly does not settle. A boundary marker, not a loose end |

## Time and commitment

| Term | Meaning |
| --- | --- |
| **t₀** | The moment of commitment. The record seals here and its substance is not rewritten afterwards |
| **tₙ** | Any later moment, when the world has responded and can be observed |
| **the wake** | Everything observed after the decision, recorded beside it rather than inside it |
| **Observed Outcome** | One thing that actually happened, recorded as a dated fact and linked to the decision that caused it |
| **Test Result** | The verdict an observation records against one specific expectation: **satisfied**, **violated**, or **inconclusive**. It belongs to the expectation being assessed, not to the observation event |
| **Expected / Emergent / Deducible** | The three kinds of observation. **Expected**: it assesses something you committed to. **Emergent**: nobody could reasonably have predicted it. **Deducible**: it could have been predicted, and nobody wrote it down at the time |
| **learning delta** | The comparison between what a decision promised and what the wake shows |
| **staged** | Not yet committed — a draft. It has no ledger number, and nothing follows from it |
| **ledger resident** | Committed: accepted, rejected, or superseded. Carries a number and can be cited |
| **retractable** | Committed, but nothing relies on it yet, so it can still be withdrawn on the record |
| **uncommit** | The act of withdrawing a retractable decision: it returns to draft, its number goes back into circulation, and both the commitment and the withdrawal stay visible. Not an edit — the substance is still not rewritten |
| **anchored** | Something relies on it. Sealed; course correction now means a new decision. Three kinds of reliance the ledger sees for itself; a citation outside it is **declared**, permanently, by whoever writes the citation |
| **supersede** | To withdraw an earlier decision by committing a new one that replaces it. The original stays, permanently, at its own moment |

## Statuses

**New** and **Deliberating** are drafts. **Proposed** is a candidate awaiting formal ratification, used
only where governance requires it. **Accepted** and **Rejected** are both commitments, and both are
final. **Superseded** is what an accepted decision becomes when a later one replaces it.

## Review vocabulary

| Term | Meaning |
| --- | --- |
| **decision debt** | The compounding cost of choices that were never properly recorded |
| **decision decay** | Old decisions quietly losing validity because the world moved and nobody revisited them |
| **polluted premise** | A decision resting on a fact that was **wrong when it was admitted**, later corrected. Not the same as the world simply changing |
| **stale constraint** | Reasoning that still treats a retired decision or corrected fact as though it were live |

> **Not built yet.** The last three name things practitioners should recognize and watch for. Memolok does
> not currently detect any of them for you.
