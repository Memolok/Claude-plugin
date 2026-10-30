# What binds a bare identifier to its ledger

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
