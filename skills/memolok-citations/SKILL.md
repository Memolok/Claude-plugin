---
name: memolok-citations
description: >-
  How to write a citation referencing an entry on a Memolok ledger: IRIs/URIs, notation, what binds
  a compact form in code, in a document, in a commit message, and in a chat message to a user — and
  the anchoring and the timing that come first.
  Load before writing any ledger identifier, especially anywhere that outlives this session (and
  whenever another Memolok skill routes here by name).
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
get one, and is not referred to outside the conversation at all — the rule and its reason are below,
under *Inside the session*.

**The durable prose entries carry their own addresses.** A Matter, World Fact, Observed Outcome or
Analysis is cited by the id the server minted for it — `mt_…`, `wf_…`, `oo_…`, `an_…` — which says
what kind of thing it is as well as which one.

**Scratchpads: freely here, not into artifacts on your own initiative.** Name a note by its `sp_…` id
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
anchor_MDR(mdlGuid, mdrHandle, kind)
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
| Decision record | `https://www.memolok.ai/mdl/<mdlGuid>/mdr/<mdrNumber>` |
| Matter | `https://www.memolok.ai/mdl/<mdlGuid>/matter/<mt_…>` |
| World Fact | `https://www.memolok.ai/mdl/<mdlGuid>/fact/<wf_…>` |
| Observed Outcome | `https://www.memolok.ai/mdl/<mdlGuid>/outcome/<oo_…>` |
| Analysis | `https://www.memolok.ai/mdl/<mdlGuid>/analysis/<an_…>` |
| Scratchpad | `https://www.memolok.ai/mdl/<mdlGuid>/scratchpad/<sp_…>` — say it here, don't ship it |

Identity sits in the path rather than the hostname, so the resolver can move without invalidating a
citation already written.

> **Not built yet.** None of these addresses resolves — not the Markdown link below, not the bare
> URL. Both are the correct thing to write and neither lands a reader on the entry today. Say so when
> you hand one to somebody who will try it.

**A ledger has no address of its own.** There is no landing page for `/mdl/<mdlGuid>` alone, so do not
offer one; name the ledger by its title if a reader needs to know which one you mean.

## What binds a compact form

What binds a bare identifier to a ledger differs by where you are writing, and that is what decides
the form.

| Where | Write |
| --- | --- |
| Code | `MDR-7`, bare |
| A commit message in that repository | `MDR-7`, bare, in the body — never in the subject |
| Markdown declaring `mdlGuid` in frontmatter | `MDR-7`, bare, anywhere in the document |
| Markdown without that frontmatter | A link: `[MDR-7](https://www.memolok.ai/mdl/<mdlGuid>/mdr/7)` |
| Anything else — email, chat, a ticket, another tool's payload | The address, written out |

**In code the binding is the tree.** `.memolok/mdl.yml` names the ledger for everything below it, so
a bare `MDR-7` in a source file is complete and unambiguous. That is the same file you read to find
out which ledger to talk to; it is doing double duty. Where there is more than one, the nearest above
the file wins.

**Nearest for whoever reads the file next, not for you.** A repository that gets cloned on its own
takes only what is inside it, so a declaration in the folder above it binds your working copy and
nothing anybody else will hold. Before writing a bare identifier into a repository, check there is a
`.memolok/mdl.yml` within that repository — and if there is not, say so and offer one rather than
writing a reference that resolves only where you are sitting.

**A commit message is bound by the same tree, and cannot be edited afterwards.** The message is
stored in the repository whose history it belongs to, so what licenses a bare number in a source file
licenses one here. It is a project artifact: anchor as `project` first, and take more care than usual,
because a file can be corrected later and a message that has been pushed cannot.

**Cite in the body, never in the subject.** A subject line tells somebody scanning history what the
change does, before they have asked why, and an identifier there spends characters that sentence
needs on an answer to a question nobody has put yet. The body is where they have put it.

**In Markdown the binding is frontmatter.** A document whose frontmatter carries `mdlGuid` licenses
bare identifiers throughout, meaning that ledger and no other, and it outranks any `.memolok/mdl.yml`
above it — which is how a document kept in one project cites a decision recorded in another. Without
it, every reference must carry the full address — one document, one rule, so a reader never has to
work out which convention is in force. Where a document will carry several references, adding the
frontmatter is cheaper than a link per citation — suggest it, and let the user decide; it is their
document.

```yaml
---
mdlTitle: <the title>
mdlGuid: <the guid>
---
```

`mdlTitle` is there so a reader can see which ledger without resolving anything, and it is optional.
`mdlGuid` is what binds. Add them *in that order* alongside whatever keys the document already
carries — the `mdl` prefix keeps them clear of the `title` a renderer has almost certainly claimed
already.

**Everywhere else there is no binding at all.** Mail, chat, tickets and word processors have nowhere
to declare a ledger, so a bare identifier there means nothing to whoever reads it next. Write the
address out.

**A wrong binding fails quietly.** The prefixed ids are scoped to their ledger, so a bare `mt_…` read
under the wrong one cannot be told apart from a reference to nothing. That is the reason the check
above is worth doing at the moment of writing rather than trusting to how the tree looked last time.

## How the citation reads

**Trailing parenthetical, at the end of the clause it justifies.** `(MDR-7)`, closing the sentence or
the clause whose shape the record explains. This is the default and most of what you will write: it
reads as an aside, so the prose still parses with it deleted, and a reader who does not care about the
ledger is never made to step over it mid-sentence.

**Several entries share one parenthesis.** `(MDR-7, MDR-12)`, comma-separated, in the order the
argument uses them. Two parentheticals in a row assert two asides about two different clauses, which
is a claim about the sentence that is usually false.

**A relationship goes inside the parenthesis.** Where one record narrows, settles or replaces another
and a reader needs the pair to follow the line, say which is which in the same breath — `(MDR-12,
narrowing MDR-7)`. The bare pair says only that both bear on this, and sends a reader to open two
records to reconstruct a relationship you already had.

**An identifier may open the line where the entry is the subject.** `MDR-7:` heading a docstring or a
section banner, where what follows exists *because of* that record. That is a topic sentence rather
than a citation beside one, so it is right only where it is true — a banner over code that merely
shares the area claims a provenance the code has not got.

**Backticks mark a token, never a citation.** Write `` `MDR-7` `` when the string itself is the
subject: parsing it, printing it, telling it apart from something else. Write MDR-7 bare when you name
the decision. A backticked citation reads as an identifier to go and find in the code, and the habit
spreads — once one is in code font, the next one matches it.

**Another project's decision numbers keep that project's own notation.** A number from a different
decision-record system is never reshaped into a Memolok-looking one — `ADR-0007` stays `ADR-0007`,
padding and all — and where the surrounding text does not already make the system obvious, its name
goes in front. Reshaped to the local convention it reads as one of yours, and sends a reader to a real
entry that says something else.

Under all of them sits one placement rule: **put the citation where a reader asking "why is this line
like this" would look, and not where the line already answers for itself.** One note in a module
docstring beats the same parenthetical on six lines inside it.

## Inside the session

Rule F in the method governs naming here. The inserted `h` and the missing separator in `MDRh7` are
deliberate: a corrupted handle fails loudly instead of degrading into a citation that looks valid.

> **A staged record is not referred to outside the conversation.** `MDRh7` names it between you and
> the practitioner and nowhere durable — not in a file, not in a document, not in a commit message,
> not in a payload you hand to another tool.

Three things make that a rule rather than a caution. It has no number. It may never acquire one, so
the reference can end up naming something that never became a decision. And it cannot be anchored —
the declaration that makes a citation safe is refused for a staged record, because there is no number
for anything to cite — so a reference to one is a dependency nothing can protect, pointing at prose
still free to change underneath it.

Where staged work has to be named outside the conversation, paraphrase its head Claim. That carries
the meaning and promises nothing the ledger has not promised.

## What Memolok knows afterwards

That a citation exists, and roughly what kind of place it lives in. Not where, not how many, not
whether it is still there. The ledger points outward; it does not hold the artifact, and reading an
entry will not tell you which files mention it.
