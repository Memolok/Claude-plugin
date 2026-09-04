# Citing a record outside the session

A decision worth recording gets referred to somewhere else — in the code it justifies, in a design
document, in the message that told everyone. **Writing that reference is one of the things you
produce**, alongside the record itself, and it has a form and a precondition.

## Only admitted records

A staged record has no number, and it may never get one. Nothing about it is safe to write down
outside the session that is drafting it. Wait for admission.

## Anchor first, then write

**Citing does not anchor. You do.**

```
anchor_MDR(mdlGuid, mdrHandle, kind)
```

`kind` is `project` for a citation living in a project artifact — a source file, a document in the
repository — and `other` for anywhere else one can go: an email, a chat message, a ticket, a slide.
Only the kind is recorded. There is no location parameter, and that is deliberate: a stored path goes
stale the moment a file moves, and a stale location reads as authoritative.

Do it **before** the citation exists, not after. An unanchored number can be released by an
Uncommit and taken by the next admission — at which point the reference you wrote silently names a
different decision. Anchoring closes that off permanently: the record refuses an Uncommit from then
on.

**It cannot be undone, and nothing checks it.** Memolok cannot see your files or your mail, so the
declaration is taken on trust. Tell the user plainly if they ask to reverse one — the correction is a
later record saying so, never a call.

Anchoring is member-level, so anyone who can write the citation can declare it.

## The form the citation takes

Three channels, and the difference between them is what binds a bare number to a ledger.

| Where | Write |
| --- | --- |
| Code | `MDR-7`, bare |
| Markdown declaring `mdlGuid` in frontmatter | `MDR-7`, bare, anywhere in the document |
| Markdown without that frontmatter | A link: `[MDR-7](https://www.memolok.ai/mdl/<mdlGuid>/mdr/7)` |
| Anything else — email, chat, a word processor | The full address, written out |

**In code the binding is the tree.** `.memolok/mdl.yml` names the ledger for everything in the
project, so a bare `MDR-7` in a source file is complete and unambiguous. That is the same file you
read to find out which ledger to talk to; it is doing double duty.

**In Markdown the binding is frontmatter.** A document whose frontmatter carries `mdlGuid` licenses
bare numbers throughout, meaning that ledger and no other. Without it, every reference must carry the
full address — one document, one rule, so a reader never has to work out which convention is in
force. Offer to add the frontmatter when a document will carry several references; it is cheaper than
a link per citation.

**Everywhere else there is no binding at all.** Mail, chat and word processors have nowhere to
declare a ledger, so a bare number there means nothing to whoever reads it next. Write the address
out.

> **Not built yet.** Following one of those addresses does not resolve to the record yet. The form is
> the correct one to write — but a reader who clicks it today does not land on the decision. Say so if
> you are handing a link to someone who will try it.

Never write `MDRh<handle>` outside the session. It names a provisional record, and whether it belongs
in a document at all is unsettled.

## What Memolok knows afterwards

That a citation exists, and roughly what kind of place it lives in. Not where, not how many, not
whether it is still there. The ledger points outward; it does not hold the artifact, and reading a
record will not tell you which files mention it.
