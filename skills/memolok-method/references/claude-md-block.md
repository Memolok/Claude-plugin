# A block that's yours inside a file that's not

Where the only thing a surface loads is the project's own `CLAUDE.md`, the paragraph goes **into**
that file — in a region bounded by markers, which is the plugin's territory inside a file that is the
practitioner's.

```markdown
<!-- memolok:begin -->
# Memolok

…the paragraph, verbatim…
<!-- memolok:end -->
```

**Nothing outside the markers is touched, ever.** Not reformatted, not reordered, not tidied. The
practitioner's prose is theirs; the block is a guest in it.

## Why the markers are HTML comments

Block-level HTML comments in `CLAUDE.md` are **stripped before the file reaches the context window**,
and stay visible when the file is opened with a read tool. So they cost the practitioner nothing on
every session forever, while remaining fully visible to the agent that has to find the block again.
No other delimiter has both properties: anything the reader can see is something the reader pays for.

The fixed literal strings are chosen so they can be found rather than interpreted — the same reason
the pack's unbuilt-capability marker is an exact string rather than a description.

## Presence is judged before markers are looked for

**First ask whether the project's always-loaded prose already says a Memolok ledger is in use.**
Judge it by reading, not by matching. Only if the answer is no does anything get written.

That ordering is load-bearing. Markers are for **rewriting**, never for deciding whether the signal
is already there — so a marker that has been mangled, reformatted or deleted cannot cause a second
copy to be written. The semantic check sees the paragraph and correctly declines.

The cost of that safety, stated plainly because it is real: a block whose markers are lost can no
longer be updated or removed by the plugin. It cannot find the region to rewrite, and cannot delete
what it can no longer bound. Leave it, say so if the practitioner asks, and do not attempt to guess
where the block began.

## Rewriting and removing

As at rung 2 — replace the region whole, never merge or append, and an absent block is an answer to
offer again only on the ordinary trigger — with one difference: **removal deletes the block, not the
file.** Everything else in `CLAUDE.md` stays exactly as it was, including the blank lines around where
the block used to be.

## Asking, which is a heavier ask here

At this rung the plugin is writing into a file the practitioner maintains, and the disclosure has to
say so. Do not reuse the lighter wording:

> The paragraph goes into your project's `CLAUDE.md`, between two marker comments. Nothing outside
> them is touched. It is loaded at the start of every session in this project, but it's very short,
> just one paragraph long, and you can always turn it off by deleting that block.

The words *into your project's `CLAUDE.md`* must not be softened. A practitioner who would
have said no to that, and yes to a file of the plugin's own, has to be given the chance to.

## Where the file is

**The project folder** — not a path relative to whatever copy of `CLAUDE.md` you happen to be reading.
A surface that reaches this rung is one that handed you a copy or reaches the project through a
bridge, so a relative path resolves against a directory that may hold nothing else at all.

Write it where a later session in that project will read it from, which is the practitioner's folder
itself. If you cannot reach that folder to write it, you cannot reach this rung: say nothing and
write nothing, rather than writing somewhere that will be discarded.
