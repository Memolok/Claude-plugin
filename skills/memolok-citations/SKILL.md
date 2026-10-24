---
name: memolok-citations
description: >-
  How to reference a Memolok ledger entry from anything committed to a repository or sent to
  someone: code, a comment, a document, a commit message, a ticket, a message. You MUST NOT write
  such a reference to any ledger entry without first loading this skill, once per session. A draft
  decision record is never referenced there at all, not even as `MDRh…`; and `MDR-` is followed
  only by a record's number, never by its handle. Naming entries in chat with the user, or in your
  own session notes that are neither committed nor sent, is exempt — unless those notes draft text
  that will be copied into one.
user-invocable: false
---

# Memolok citations

A decision worth recording gets referred to somewhere else — in the code it justifies, in a design
document, in the message that told everyone. **Writing that reference is one of the things you
produce**, alongside the record itself. It has a form, a precondition, and a moment before which it
cannot be written at all.

**Every number in this file is illustrative** — avoid repeating these examples casually in
conversations with the user, because they will believe you refer to actual entries in their ledger.
In this skill, `MDR-7` names nothing; it just illustrates a shape.

## What can be cited

**Decision records: only the admitted ones.** A staged record (`MDRh42`) has no number, may never
get one, and is not referred to in anything committed or sent at all — the rule and its reason are
below, under *Inside the session*.

**The durable prose entries carry their own addresses.** A Matter, World Fact, Observed Outcome or
Analysis is cited by the id the server minted for it — `mt_…`, `wf_…`, `oo_…`, `an_…` — which says
what kind of thing it is as well as which one.

**Scratchpads: freely here, not into artifacts on your own initiative.** Name a note by its `msp_…` id
whenever it helps agree which note is meant; the address is deliberately untypeable to discourage the
durable citation, not the spoken one. Do not write one into a project artifact unasked — a note is
free to be rewritten or binned, so an artifact resting on one rests on something the ledger will not
defend. The practitioner may overrule that whenever they like; say once that a note carries no promise
of still saying the same thing later, and leave it with them. "Never citable" is about the ledger's
own reference fields, which is what keeps a note disposable; it is not a rule against naming one.

## Build, seal, then cite

**A staged record has no number, so the citation pass starts after t₀.** A draft carries a handle
(`MDRh42`) and never a number of the `MDR-33` form, and a handle is not a citation — so while you
were building against that draft, the artifact you were writing could not name the record it
implements. The references are their own pass over finished work: build, seal, then go back through
what you built and write them in.

**The seal moves ahead of the references, not ahead of the work.** Building against a draft is worth
the round trip — an implementation contradicts its Verdict often enough to be worth catching, and
before t₀ that costs one `update_MDR` where afterwards it costs a successor. Sealing at the *start*
would buy an earlier number and pay for it with that whole correction window, every time. So the
build keeps its window, and the citations are what waits.

**Say so rather than sealing quietly.** The practitioner is committing a decision at that moment,
which is theirs to do — not a formality to get past on the way to a number you can type. Tell them
that once the build is done, the references are what remains.

That pass is usually its own commit. The change and the references argue different things, and a
reviewer meeting them in one diff cannot tell which of the two it is for.

## Anchor first, then write

**Citing does not anchor. You must.**

```
anchor_MDR(mdlGuid, mdr, kind)
```

`kind` is `project` for a citation living in a project artifact — a source file, a document in the
repository, a commit message — and `other` for anywhere else one can go: an email, a chat message, a
ticket, a slide, a payload you hand to a tool on some entirely different connector. Only the kind is
recorded. There is no location parameter, and that is deliberate: a stored path goes stale the moment
a file moves, and a stale location reads as authoritative.

Do it **before** the citation exists, not after. An unanchored number can be released by an Uncommit
and taken by a later admission — at which point the reference you wrote silently names a different
decision. Anchoring closes that off permanently: the record refuses an Uncommit from then on.

**In a project artifact, this is mandatory.** Code, documentation, and commit messages all ship with
the repository and are read for years, so a reference written into one is exactly the dependency the
declaration exists to protect. *Anchor, then write.*

> **Elsewhere, weigh it and ask.** An anchor cannot be withdrawn, so declaring one because a record
> came up in conversation spends something permanent on a mention that may not outlast the afternoon.
> Ask whether this is a reference somebody will come back to. Where it plainly is — a ticket that
> will outlive the sprint, a message a team will act on — anchor it, and say that you did.

**Declare even when the record already looks anchored.** `retractable: false` does not say *why*. A
record held by ledger topology is held by something that can change; a declared anchor is permanent.
The read surface does not carry the kind, so you cannot tell them apart without attempting the
Uncommit one of them prevents — and a citation resting on the first can outlive what was holding it.
Declaring a kind twice is a no-op, so the redundant call costs nothing and the skipped one costs a
number.

**It cannot be undone, and nothing checks it.** Memolok cannot see your files or your mail, so the
declaration is taken on trust. Tell the user plainly if they ask to reverse one — the correction is a
later record saying so, never a call.

Anchoring is member-level, so anyone who can write the citation can declare it.

## The address of each entry

| Entry | Address |
| --- | --- |
| Ledger | `https://app.memolok.ai/m/<mdlGuid>` |
| Decision record | `https://app.memolok.ai/m/<mdlGuid>/MDR-<mdrNumber>` |
| Matter | `https://app.memolok.ai/m/<mdlGuid>/<mt_…>` |
| World Fact | `https://app.memolok.ai/m/<mdlGuid>/<wf_…>` |
| Observed Outcome | `https://app.memolok.ai/m/<mdlGuid>/<oo_…>` |
| Analysis | `https://app.memolok.ai/m/<mdlGuid>/<an_…>` |
| A record's alternative, expected outcome or open question | `https://app.memolok.ai/m/<mdlGuid>/MDR-<mdrNumber>#<id>` |
| Scratchpad | `https://app.memolok.ai/m/<mdlGuid>/<msp_…>` — say it here, don't ship it |

Every address is the identifier you already hold, under the ledger's own address where the
identifier is the ledger's, and a record is its citation form exactly: `MDR-7` in prose,
`…/MDR-7` in an address. Write identifiers in the case Memolok returned them. The `/m/` namespace
holds these addresses and nothing else, and is kept apart from the application around it so that a
reorganisation of the application never moves one: a citation written now stays good.

**A record's part is addressed under its record**: the record's address, then `#`, then the part's
id exactly as the record holds it, `…/MDR-7#oq-backup`. Citing a part cites its record, so the
record is what you anchor.

**Every address in the table opens.** Anyone not signed in gets a sign-in page, the same at every
address, and a member who signs in lands on the entry, a record's part in view and marked. Anyone
else gets a not-found, the same as for an entry that does not exist.

**Name a ledger by its title beside its address.** A reader then knows which one you mean without
resolving anything.

## What binds a compact form

What binds a bare identifier to a ledger differs by where you are writing, and that is what decides
the form.

| Where | Write |
| --- | --- |
| Code | `MDR-7`, bare |
| A commit message in that repository | `MDR-7`, bare, in the body — never in the subject |
| Markdown declaring `mdlGuid` in frontmatter | `MDR-7`, bare, anywhere in the document |
| Markdown without that frontmatter | A link: `[MDR-7](https://app.memolok.ai/m/<mdlGuid>/MDR-7)` |
| Anything else — email, chat, a ticket, another tool's payload | The address, written out |

**A record's part takes its record's form**: `MDR-7#oq-backup` wherever a bare `MDR-7` may stand,
and its full address everywhere else.

**What binds, in short:** in code and in a commit message, a `.memolok/mdl.yml` inside that same
repository; in Markdown, `mdlGuid` in the document's frontmatter, which outranks it; anywhere else,
nothing, so the address is written out. `references/binding.md` has the rest.

## How the citation reads

**A trailing parenthetical, at the end of the clause it justifies**: `(MDR-7)`, or `(MDR-7, MDR-12)`
for several. Backticks only where the string itself is the subject, never around a citation.
`references/citation-style.md` has the rest.

## Inside the session

Rule F in the method governs naming here. The inserted `h` and the missing separator in `MDRh7` are
deliberate: a corrupted handle fails loudly instead of degrading into a citation that looks valid.

> **A staged record's handle stays in your sessions.** `MDRh7` names it in chat and in session notes
> that are neither committed nor sent — a plan nobody commits, a hand-off pasted into the next
> session. It never goes into anything committed to a repository or sent to anyone: not a source
> file, a document, a commit message, a ticket, or a payload you hand to another tool.

The test is where the text will live, decided when you write it, not what kind of document it is. A
hand-off that gets committed is a project artifact, and text a note drafts for copying into one is
judged by where it will land.

Three things make that a rule rather than a caution. It has no number. It may never acquire one, so
the reference can end up naming something that never became a decision. And it cannot be anchored —
the declaration that makes a citation safe is refused for a staged record, because there is no number
for anything to cite — so a reference to one is a dependency nothing can protect, pointing at prose
still free to change underneath it.

None of the three binds a note that is thrown away: a handle stays a valid address to look the
record up by, and nothing depends on the note. Where staged work has to be named in committed or
sent text, paraphrase its head Claim. That carries the meaning and promises nothing the ledger has
not promised.

## What Memolok knows afterwards

That a citation exists, and roughly what kind of place it lives in. Not where, not how many, not
whether it is still there. The ledger points outward; it does not hold the artifact, and reading an
entry will not tell you which files mention it.

## References

| File | Load when |
| --- | --- |
| `references/binding.md` | Before the first bare identifier you write into a repository or a Markdown document — a declaration in the folder above a repository binds only your copy, and a wrong binding fails without a sound |
| `references/citation-style.md` | Before writing a citation into prose — a relationship left out of the parenthesis sends a reader to open two records, and another system's number reshaped to `MDR-` names a real record that says something else |
