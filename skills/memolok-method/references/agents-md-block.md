# A block that's yours inside AGENTS.md

Codex loads one instructions file per folder, `AGENTS.md`, and nothing like a rules file of the plugin's own. So in Codex the paragraph goes **into** the project's `AGENTS.md`, in a region bounded by markers, which is the plugin's territory inside a file that is the practitioner's.

```markdown
<!-- memolok:begin -->
# Memolok

…the paragraph, verbatim…
<!-- memolok:end -->
```

**Nothing outside the markers is touched, ever.** Not reformatted, not reordered, not tidied. The practitioner's prose is theirs; the block is a guest in it.

The markers are fixed literal strings so they can be found rather than interpreted. Whether Codex hides HTML comments from the context is not established; at worst they are two short lines.

## Presence is judged before markers are looked for

**First ask whether the project's `AGENTS.md` already says a Memolok ledger is in use.** Judge it by reading, not by matching. Only if the answer is no does anything get written.

That ordering is load-bearing. Markers are for **rewriting**, never for deciding whether the signal is already there — so a marker that has been mangled, reformatted or deleted cannot cause a second copy to be written.

The cost of that safety, stated plainly because it is real: a block whose markers are lost can no longer be updated or removed by the plugin. Leave it, say so if the practitioner asks, and do not attempt to guess where the block began.

## Rewriting and removing

Replace the region whole when it differs from the paragraph; never merge or append. An absent block is the practitioner's answer: offer again only on the ordinary trigger. **Removal deletes the block, not the file.** Everything else in `AGENTS.md` stays exactly as it was, including the blank lines around where the block used to be.

## Asking

The plugin is writing into a file the practitioner maintains, and the disclosure has to say so, in these words:

> The paragraph goes into your project's `AGENTS.md`, between two marker comments. Nothing outside them is touched. It is loaded at the start of every Codex session in this project, and by any other agent that reads `AGENTS.md`, but it's very short, just one paragraph long, and you can always turn it off by deleting that block.

The words *into your project's `AGENTS.md`* must not be softened.

## Where the file is

**The project's root**: the nearest folder at or above the project folder that holds `.git`, or the project folder itself when none does. Codex reads `AGENTS.md` from there in both cases. Create the file if it does not exist.

If you cannot write to that folder, say nothing and write nothing.
