---
name: memolok-citations
description: >-
  Load before you write a Memolok ledger identifier, such as `MDR-12`, into anything that outlasts
  this session: a file edit, a code comment, a document, a commit message, a ticket, a message you
  send, or notes kept for later sessions such as agent memory. You MUST NOT write such a reference
  without first loading this skill, once per session, even when the file already carries others.
  Naming entries in chat with the user, or in notes that end with this session, is exempt, unless
  those notes draft text that will be copied out.
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

**Write an id only once a call of yours has returned it**: never invented, completed or relayed
unread from a report. Where a file needs it earlier, leave `wf_…` and grep for it before committing;
a commit message or sent text takes no marker, so create first.

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
repository, a commit message, notes kept for later sessions such as agent memory — and `other` for
anywhere else one can go: an email, a chat message, a ticket, a slide, a payload you hand to a tool
on some entirely different connector. Only the kind is
recorded. There is no location parameter, and that is deliberate: a stored path goes stale the moment
a file moves, and a stale location reads as authoritative.

Do it **before** the citation exists, not after. An unanchored number can be released by an Uncommit
and taken by a later admission — at which point the reference you wrote silently names a different
decision. Anchoring closes that off permanently: the record refuses an Uncommit from then on.

`mdr` takes the handle. Read the record by its number first and take the handle from that response,
which states both; one from a scout's report, memory or a neighbouring record may name another
record, and a wrong anchor is permanent.

**Every citation you write is anchored first, in a project artifact and everywhere else.** Code,
documentation and commit messages ship with the repository and are read for years; a message, a
ticket or a slide is read by people who act on it. Either way the reference is exactly the dependency
the declaration exists to protect. *Anchor, then write.* Text you did not write counts: an archive or
quoted transcript committed verbatim cites each record it names, so anchor each as `project` first.
Unedited, it binds through `mdlGuid` frontmatter, not links. Notes kept for later sessions count
too: the sessions that load them act on what they say, so a number there is anchored before it is
written, exactly as in code.

> **Tell the user first, and let them weigh it.** Before anchoring, say which records the reference
> would anchor and what that means: an anchored record can never be uncommitted, so a mistake found
> in it later — even an hour later, in a message nobody reread — is corrected by a new record, never
> by a withdrawal. The choice is theirs: anchor and cite, or leave the record unnamed in that text
> and say what it decided in words. A citation without its anchor is not one of the options. The
> records one piece of work cites can be put to them together. Where a record already reads
> `retractable: false`, nothing new is spent and there is nothing to explain: declare it and write.

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
| Decision record, staged | `https://app.memolok.ai/m/<mdlGuid>/MDRh<mdrHandle>` — a working link people share while deliberating, never a citation; it moves to the number at admission |
| Matter | `https://app.memolok.ai/m/<mdlGuid>/<mt_…>` |
| World Fact | `https://app.memolok.ai/m/<mdlGuid>/<wf_…>` |
| Observed Outcome | `https://app.memolok.ai/m/<mdlGuid>/<oo_…>` |
| Analysis | `https://app.memolok.ai/m/<mdlGuid>/<an_…>` |
| A record's alternative, expected outcome or open question | `https://app.memolok.ai/m/<mdlGuid>/MDR-<mdrNumber>#<id>` |
| Scratchpad | `https://app.memolok.ai/m/<mdlGuid>/<msp_…>` — say it here, don't ship it |

Every address is the identifier you already hold, under the ledger's own address where the
identifier is the ledger's, and a record is its citation form exactly: `MDR-7` in prose,
`…/MDR-7` in an address. Write identifiers in the case Memolok returned them. A note's `msp_…` is
unique everywhere, but its address still sits under its ledger: `…/m/<msp_…>` alone names nothing.
The `/m/` namespace holds Memolok's addresses and nothing else, and is kept apart from the
application around it so that a reorganisation of the application never moves one: a citation
written now stays good.

> **Not built yet.** A feedback report, a user and a service principal have addresses too —
> `https://app.memolok.ai/m/<mfb_…>`, and `…/m/<musr_…>#id` or `…/m/<msvc_…>#id` naming the
> principal itself — but none of them opens yet. Give a reader the identifier rather than a link
> that leads nowhere.

**A record sealed before these addresses existed may quote older ones** —
`https://www.memolok.ai/mdl/…`, a bare ledger guid, an `sp_…` note id. None was ever served and nothing redirects them: quote the
record as it stands, and write the current address in your own reference.

**A record's part is addressed under its record**: the record's address, then `#`, then the part's
id exactly as the record holds it, from your own read, `…/MDR-7#oq-backup`. Citing a part cites its
record, so the record is what you anchor.

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
| Notes only this project's sessions load, such as agent memory | `MDR-7`, bare, for an entry on the ledger the project's `.memolok/mdl.yml` names |
| Markdown declaring `mdlGuid` in frontmatter | `MDR-7`, bare, anywhere in the document |
| Markdown without that frontmatter | A link: `[MDR-7](https://app.memolok.ai/m/<mdlGuid>/MDR-7)` |
| Anything else — email, chat, a ticket, another tool's payload | The address, written out |

**A record's part takes its record's form**: `MDR-7#oq-backup` wherever a bare `MDR-7` may stand,
and its full address everywhere else. **So does every other ledger entry**: a bare `wf_…`, `mt_…`,
`oo_…` or `an_…` where `MDR-7` may stand bare, and its address everywhere else.

**What binds, in short:** in code and in a commit message, a `.memolok/mdl.yml` inside that same
repository; in notes only this project's sessions load, the project's own `.memolok/mdl.yml`; in
Markdown, `mdlGuid` in the document's frontmatter, which outranks it; anywhere else, nothing, so
the address is written out. `references/binding.md` has the rest.

## How the citation reads

**A trailing parenthetical, at the end of the clause it justifies**: `(MDR-7)`, or `(MDR-7, MDR-12)`
for several. Backticks only where the string itself is the subject, never around a citation.
`references/citation-style.md` has the rest. A line quoted from a record comes from your own
`get_MDR` read, not a report: a summary drops what a sentence turns on.

## Inside the session

Rule F in the method governs naming here. The inserted `h` and the missing separator in `MDRh7` are
deliberate: a corrupted handle fails loudly instead of degrading into a citation that looks valid.

> **A staged record's handle stays in your sessions.** `MDRh7` names it in chat and in session notes
> that are neither committed nor sent — a plan nobody commits, a hand-off pasted into the next
> session. It never goes into anything committed to a repository or sent to anyone: not a source
> file, a document, a commit message, a ticket, or a payload you hand to another tool. Feedback to
> Memonos is the one exception: it may name a draft by its handle, because the team behind Memolok
> resolves one reliably.

The test is where the text will live, decided when you write it, not what kind of document it is. A
hand-off that gets committed is a project artifact, and text a note drafts for copying into one is
judged by where it will land. A verbatim archive naming a staged record by `MDRh…` is not
committable as written: paraphrase its head Claim and mark the edit, or omit the passage.

Rule F carries why: a staged record has no number, may never get one and cannot be anchored, so a
reference to one is a dependency nothing can protect. None of that binds a note that is thrown away:
a handle stays a valid address to look the record up by, and nothing depends on the note. Where
staged work has to be named in committed or sent text, paraphrase its head Claim. That carries the
meaning and promises nothing the ledger has not promised.

## What Memolok knows afterwards

That a citation exists, and roughly what kind of place it lives in. Not where, not how many, not
whether it is still there. The ledger points outward; it does not hold the artifact, and reading an
entry will not tell you which files mention it.

## References

| File | Load when |
| --- | --- |
| `references/binding.md` | Before the first bare identifier you write into a repository or a Markdown document — a declaration in the folder above a repository binds only your copy, and a wrong binding fails without a sound |
| `references/citation-style.md` | Before writing a citation into prose — a relationship left out of the parenthesis sends a reader to open two records, and another system's number reshaped to `MDR-` names a real record that says something else |
