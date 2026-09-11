# The cold-start signal

**Getting this signal into a project is one of the things you produce** — once per project, not once
per session. It is fixed text you copy, not prose you compose, and the difference matters more than
it looks.

The objective: a session opening in this project learns it records decisions in Memolok **before it
starts work**, without the practitioner saying so. Where that text goes differs by what the session
in front of you can actually do.

## Where it goes — take the first rung that reaches

**Reach the objective through the most isolated surface available.** Isolation means how little of
the practitioner's own material you touch. Work down; take the first rung that genuinely reaches a
later session here, and stop.

| | Ask | If yes |
| --- | --- | --- |
| **1** | Does this session carry an instructions field the practitioner maintains — prose that arrived with no path, that they can edit and you cannot? | **Hand it to them.** You write nothing at all |
| **2** | Is the project a tree you read and write directly, the same one a later session here would open? | **`.claude/rules/memolok.md`**, a file that is entirely yours |
| **3** | Do you reach the project's files only through a bridge, or as a copy staged for you? | A delimited block in the project's `CLAUDE.md` — `claude-md-block.md` |
| **4** | None of those | **Nothing.** Say nothing about it |

**Never insert free-form into prose somebody else wrote**, at any rung. Rung 3 is a bounded region
you own inside a file you do not; that boundary is the whole of what makes it acceptable.

Question 2 is the one that decides most cases, and it is not the same as *is there a folder*. A
surface that hands you a **copy** of the project, or reaches it through a separate tool family, will
not carry a directory you create — you would write a file, the write would succeed, and nothing would
ever load it. Prefer rung 3 there, which is the one thing such a surface does carry.

Rung 4 is not a special case. Where nothing reaches a later session, the most isolated action is to
take none.

## Asking

**The practitioner agrees to this specifically.** A yes to saving a ledger id is **not** a yes to
this; they were asked one question and gave one answer. Ask separately, and say what it costs:

> A file in `.claude/rules/` is loaded at the start of every session in this project, by any Claude
> agent, whether or not it touches Memolok. This one is very short, just one paragraph long.

That disclosure *is* the consent. Without the cost stated, the answer is uninformed.

**Rung 1 needs no disclosure of that kind** — they are pasting it into their own instructions, so the
cost is visible in the act. Show them the paragraph, say where it goes, and leave it with them.
**Rung 3's disclosure is different and heavier**; `claude-md-block.md` carries it.

## Repeating, declining, and staying quiet

A practitioner who never uses Memolok in a session is never asked; one who uses it often is asked in
many sessions, which is the point — the nudge scales with the benefit.

Nothing persists a decline. **A no is a no for this session**, and the next session has no way to know
it was ever asked; after a compaction, neither do you. Say so if the user seems to expect otherwise,
and do not pretend to a memory you lack. If the user complains, offer to log a feedback report to
Memonos.

**Say nothing at all** where no rung reaches — a read-only tree, or rung 4. A session that cannot
carry the signal is not the practitioner's problem and does not need reporting. Rung 1 is the
opposite case and *is* mentioned, once: pasting is theirs to do, so they have to hear about it to do
it.

## The paragraph

Copy this exactly. At rung 2 it is the whole file; at rung 3 it is what goes between the markers; at
rung 1 it is what you show them to paste. Do not rewrite it in this skill's register, do not improve
it, and do not personalise it:

```markdown
# Memolok

<!-- Written and owned by the Memolok plugin. Edits here are replaced wholesale — ask the plugin
     to change it rather than editing it. -->

This project records its decisions in a Memolok Decision Ledger.

If the Memolok skills are available, load `memolok-method` and establish the ledger from
`.memolok/mdl.yml` in the project folder, which names it and says what it is for. Do this now, even
for a question that looks like it needs neither: you cannot tell in advance which turn will want the
ledger, and finding it unreachable part-way through an answer is worse than finding it before one.
Error response
`Memolok Decision Ledger not found.` from the server does not distinguish a ledger the user cannot
see from one that is not there. Report it as ambiguous: they may not be a member, or this address
may be stale.

If the Memolok skills are not available, ignore the paragraph above and don't go looking for them.
```

At rung 1 drop the heading and the comment — they are file furniture, and an instructions field is
not a file.

**It names the project folder, never a path relative to itself.** On a surface that reads it from a
staged copy, a relative reference resolves against a directory holding one whitelisted file and finds
nothing.

**It carries no ledger content, and must never gain any.** No guid, no title, no purpose, no record
numbers. Orientation arrives one step later, from `.memolok/mdl.yml`. A paragraph that names the
ledger reads better and is a trap: it goes stale in the one project file nothing ever refreshes, and
nothing will tell anybody it has.

**Byte-identical everywhere**, which is stronger than short. It means you can replace it without
reading anything first, and a drifted copy shows up as a plain diff instead of a judgement call about
whether the wording still means the same thing.

**The closing sentence is not redundant with the opening conditional.** The conditional stops the
instruction firing; the closing sentence stops an agent treating the absence of the skills as a
problem to solve — hunting for them, proposing an install, reporting them missing. Between them they
are what makes an orphaned paragraph harmless after an uninstall. Trim neither.

**Every referent in it resolves locally**, deliberately. The text lands concatenated among other
instruction files it knows nothing about, so a bare "this" or "the above" can bind to something a
different author wrote. That is why it says *the Memolok skills* and *the paragraph above* where one
word would read more smoothly.

**The not-found line is one the pack states in several places**, and this is the only copy that has
left the building — no grep will find it here if the phrasing changes elsewhere. That is a deliberate
exception, taken because it is the one thing a reader cannot get from the server: the message is
uninformative by design, so it answers identically for a ledger you cannot see, one that never
existed, and an address that has gone stale. Reporting it as ambiguous is the whole of the
instruction. Do not let it drift into diagnosing one.

## Replacing and removing

**At rung 2 the file is yours.** Rewrite it wholesale when it differs from the text above — read it,
judge whether it says the same thing, replace it entirely if not. Never merge, never append.

If it is gone, the practitioner removed it. That is an answer, not an omission: offer again on the
ordinary trigger, and take a second no the same way you took the first.

**At rung 1 nothing is yours.** They pasted it; they can change or delete it, and you neither
maintain it nor go looking for it. **Rung 3 has its own rules** — `claude-md-block.md`.

## What Memolok knows afterwards

Nothing. No tool records that the signal exists, was offered, or was declined, at any rung. The
ledger does not know, and neither will the next session.
