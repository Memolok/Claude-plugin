---
name: help
description: >-
  Explain what Memolok is and why it records decisions the way it does — decision debt, what makes
  something a decision, why a sealed record cannot be edited, what the vocabulary means, and what the
  product does not do yet. Use when someone asks "what is Memolok", "why can't I just fix that
  record", "why do I have to say what I expect to happen", "isn't this just ADRs", "why not point an
  LLM at our wiki", "what does t₀ / the wake / a matter actually mean", or "why is it even called that".
argument-hint: "[what you want explained]"
---

# /memolok:help — Understand Memolok

Explains the thinking behind Memolok: the problem it exists for, why it is strict where it is strict,
what the words mean, and where the methodology currently runs ahead of the product.

This skill reads and explains. It writes nothing and calls no tools, so it works before sign-in, before
a ledger exists, and when the connection is broken.

## Usage

```
/memolok:help [what you want explained]
```

Examples:

- `/memolok:help what is Memolok?`
- `/memolok:help why can't I just edit the record I got wrong?`
- `/memolok:help what's t₀?`
- `/memolok:help isn't this just ADRs with extra steps?`

## Step 0 — Open a file before you answer

**This file does not answer; it routes.** The substance lives in `references/`, and the one-line answers
in the map below are there to help you pick the right file — they are not the reply. If someone asked a
real question, open the file the map points at and answer from what is in it.

The failure mode this exists to prevent: skimming the map, recognizing the topic, and composing a
plausible answer out of general knowledge without opening anything. It reads fluently and it is how you
end up telling someone that Memolok has no answer to a question these files answer directly.

If the map does not obviously cover the question, open the closest file anyway — and if the question is
about a word, that is always `references/glossary.md` or `references/why-these-words.md`.

Only when the question is **mechanical** — lifecycle, statuses, well-formedness gates, fish anatomy,
prose format — load the **`memolok-method`** skill instead and answer from it. Do not load it for *why*
questions; it is agent operating substrate and costs its whole length. That conditionality is deliberate.

## What Memolok is

Organizations, teams, and solo practitioners all forget *why*. The result survives — the contract, the
code, the policy — while the reasoning that produced it evaporates. Memolok records the decision itself:
the need that motivated it, the options weighed, the commitment made, and what the decider expected to
happen as a result.

It is **opinionated about how you record** decisions and **agnostic about how you make them** — consensus,
hierarchy, committee, mandate, coin toss — and about domain: software, legal, medical, construction, a
kitchen renovation.

The shape is one decision per record. Each record seals at the moment of commitment and is not rewritten
afterwards. What actually happens later is recorded separately and compared against what was promised.

That is the whole model. Everything else is consequence.

## How to answer

- **Answer from the file, not from memory.** See Step 0. If you find yourself explaining a Memolok term
  from general knowledge, you have skipped a step.
- **Answer the question they asked**, one idea at a time. This is teaching, not a brochure. Nobody asked
  for the tour.
- **"Why is it called that" is a real question with a real answer.** The vocabulary was chosen
  deliberately and the reasons are recorded. Never treat a naming question as unknowable trivia.
- **Use their domain.** Kafka examples are for people who use Kafka – but bathroom renovation is an equally
  valuable decision recording domain.
- **Follow the jargon policy.** Do not introduce **MDR**, **Claim**, or t₀ unless they used the word first
  or the question is about the vocabulary itself.
- **Never present methodology as tooling** — see below.
- **When they stop asking and start deciding, stop explaining and route.** Someone who says "so anyway we
  went with Postgres" is no longer asking a question. Hand to **`record-decision`**.

## Methodology is not tooling

Memolok's methodology models more than the current release enacts. Some of what these files describe is
how the model works and what practitioners should do; some of it is not something the product can do for
you yet.

Any capability that is not built carries a line marked **Not built yet** in the file that describes it.
Read those as binding. Say so plainly, explain the practice the user can still follow by hand, and **do
not offer to do the thing**.

If you are unsure whether something exists, say you are not sure. The honest answer to *"what can Memolok
actually do today?"* is the set of tools you can actually call — not the breadth of the methodology. Never
attach a date or a roadmap position to anything unbuilt.

## Where the answers live

Open one. Almost every question resolves to a single file. The middle column is how you choose it, not
what you say.

### The problem it exists for

| They asked | The short answer | Load |
| --- | --- | --- |
| What is this, why bother | Unrecorded decisions compound like technical debt | `references/decision-debt.md` |
| Isn't this for big companies | Amnesia is not a team problem, it is a time problem | `references/solo-practitioners.md` |
| Was that even a decision | No commitment, no decision — and no-choice situations are still choices | `references/what-counts-as-a-decision.md` |

### Why it is strict

| They asked | The short answer | Load |
| --- | --- | --- |
| Why can't I edit a sealed record | Hindsight destroys the evidence the record exists to hold | `references/why-records-are-sealed.md` |
| So it is permanent the second I commit | No — until something relies on it, it can be withdrawn on the record | `references/retractable-vs-anchored.md` |
| Can't I just mark it obsolete | Nothing stops being true on its own; someone decided to stop | `references/supersession-and-withdrawal.md` |
| Why must I say what I expect | An unfalsifiable decision can never teach you anything | `references/expected-outcomes.md` |
| What if I don't know everything yet | Name what you are not settling and commit anyway | `references/open-questions.md` |
| Is rejecting it a failure | A sealed no is a real decision and kills the zombie | `references/rejection-as-outcome.md` |

### What happens afterwards

| They asked | The short answer | Load |
| --- | --- | --- |
| Where do results go | Outside the record, because the record is frozen | `references/the-wake.md` |
| When do I check the outcome | Later than you think, and the gap itself is risk | `references/outcome-delay-window.md` |
| What do I do with the results | Three kinds of surprise, and what a pattern of them means | `references/learning-from-outcomes.md` |

### Where decisions come from

| They asked | The short answer | Load |
| --- | --- | --- |
| Why can't the bug report be the need | The person in pain rarely speaks the business domain | `references/matters-and-analysis.md` |
| How do I close something we never did | Honestly, and there are several honest ways | `references/closing-issues-honestly.md` |

### Context and boundaries

| They asked | The short answer | Load |
| --- | --- | --- |
| Where do facts and assumptions go | The almanac — true regardless of any one decision | `references/world-almanac.md` |
| How many ledgers should we have | One per shared worldview, not one per person | `references/ledgers-and-scope.md` |
| Why does a ledger say what it is for | Because every session starts with no memory of the last | `references/ledger-intent.md` |
| Where do rough notes go / why can't I cite my note | Scratchpads — the one disposable thing, and why nothing may point at it | `references/scratchpads.md` |

### Comparisons and pushback

| They asked | The short answer | Load |
| --- | --- | --- |
| Why not point an LLM at our wiki | You will get plausible prose, which is the problem | `references/why-not-a-wiki.md` |
| Isn't this just ADRs / bureaucracy / overkill | Fair questions, answered without the sales pitch | `references/common-objections.md` |

### Vocabulary

Any question about a word lands here. Both files, if the question is both.

| They asked | The short answer | Load |
| --- | --- | --- |
| What does *X* mean | Plain definitions of every term | `references/glossary.md` |
| Why is it called *X* | The vocabulary was chosen deliberately; here is why | `references/why-these-words.md` |
| Why a fish / why "the wake" / why "matter" and not "issue" / why "intent" and not "mission" | Same — the metaphors carry meaning | `references/why-these-words.md` |

## Then get out of the way

| They want to | Skill |
| --- | --- |
| Connect, sign in, or pick a ledger | **`start`** |
| Record a decision they have made or are making | **`record-decision`** |
| Be walked through one they have not worked out | **`grill-me`** |
| See what the ledger already holds | **`review-ledger`** |
| Write down what a ledger is for | **`revise-intent`** |

Explaining is not the goal. Someone who understands Memolok and has recorded nothing has gained nothing.
