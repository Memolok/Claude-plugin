# Changelog

What changed in the Memolok pack, newest first, written for someone deciding whether to update.

The pack is a set of skills: prose that changes what your agent does. So a wording change here is a
behaviour change, and every shipped edit gets a version — including documentation-only ones.

- **Patch** — corrections that do not change what the agent does.
- **Minor** — new skills, new tools reflected in the catalog, or changed facilitation behaviour.
- **Major** — an installed invocation stops resolving. While the pack is pre-1.0 that rides in a
  minor bump, on the ordinary 0.x convention; you are still told to update, because the server
  publishes the oldest pack it accepts separately from the version number.

The pack version is independent of the Memolok server's. You install the pack on your schedule; the
server is deployed on ours, so the two cannot be held equal. The server publishes the oldest pack it
still accepts, and tells your agent at the start of a session whether the pack it is running still
matches — you are asked to update only when your pack genuinely stops working.

> Entries for `0.9.0-alpha` through `0.16.0-alpha` were written on 2026-08-25 from the release
> history rather than at the time, so they are terser than the ones after them. Nothing in them is
> reconstructed beyond what the tags and their contents show.

## Unreleased

*Landed, not yet in a released pack.*

*Nothing yet.*

## 0.23.0-beta — 2026-09-05

**Update required.** The Memolok server no longer accepts packs older than this one. An earlier pack
describes a call the server now refuses outright, and — the half that matters more — reads several
responses confidently wrong rather than failing, so there is no version of this you would want to
stay on.

**Every entity now has a short identifier of its own.** A matter, world fact, observed outcome or
analysis is named by a nine-character value like `mt_3kf9xq` instead of twenty-four hexadecimal
characters, and the two-letter prefix says what it addresses. Nothing you type changes; the values
your agent quotes back to you do. An identifier written down during an older session may no longer
resolve, and your agent will now tell you that rather than guessing.

- **Withdrawing an analysis reference names the two things it joins** — the analysis and the input —
  instead of a reference id. There is no reference id anywhere on the surface any more.
- **A staged record is now named `MDRh7`, not paraphrased.** An admitted one is still `MDR-7`, exactly
  as before. The two are different addresses and routinely point at different records, so the
  provisional form is marked to stop it being read as a citation.
- **New: your agent can declare that something outside the ledger cites a record.** Once declared,
  that record's number can never be released, so a reference you wrote into a file, a document or an
  email cannot come to mean a different decision later. The declaration records only that a citation
  exists and roughly what kind of place it lives in — never where. It is permanent and nothing
  verifies it.
- **Your agent now knows how to write a citation.** In code, the bare number is enough, because the
  project file that names your ledger binds it. In a document, either the frontmatter names the
  ledger or each reference carries the full address. In mail or chat, where nothing can bind it, the
  full address is the only correct form. Following one of those addresses does not resolve to the
  record yet, and your agent will say so.

## 0.22.0-beta — 2026-09-02

**Not every line on a matter row is the words somebody typed, and your agent now knows the
difference.**

A `list_matters` row can carry a short label Memolok supplies alongside the raiser's own trimmed
words, and `get_matter` carries more of the same beside the full text. That reads exactly like a
short quotation and nothing in the response distinguishes them — so a skill told to present parked
matters *"in the words they were logged in"* would have begun presenting a paraphrase as somebody's
own utterance, confidently and invisibly. That is a wording problem, which is why it is a pack
release.

- **`review-ledger`** now keeps the two apart when it reports the unprocessed inbox: quote the
  excerpt when quoting the raiser, and `get_matter` is where the exact wording lives.
- **The ledger scout** cites the excerpt rather than the label when reporting what a matter says.
- **The tools reference** describes what a matter row and `get_matter` each carry, and states
  plainly that a row is no longer shaped like a trimmed `get_matter`.

**`discover_matters` is a new way to scan a queue.** It answers the same selection as `list_matters`
— same arguments, same order, same ids — as prose rather than rows, so you can tell what is waiting
without opening entries one at a time, and each page says which of its words are the raiser's.
`review-ledger`, `record-decision`, `grill-me` and the ledger scout reach for it when they look for
parked bait.

**One thing you may notice.** Searching matters now reaches more than the raiser's own text, so a
query can return a matter whose wording does not contain your term, and results come back in a
different order than before. Nothing you do changes: no tool gained a parameter and no skill gained
a step.

## 0.21.1-beta — 2026-08-27

**Reading a ledger is now delegated by default.** The scout shipped in 0.21.0-beta and almost nothing
called it, so reviews went on being read inline — into your conversation — exactly as before the
agent existed. Sweeps now go to it whatever the wider task is; short, bounded reads stay inline.

**The scout no longer tells you how to fix a record.** It reports whether a decision can still be
uncommitted and leaves the choice to you. It had been recommending a path, and on a sealed record
that advice was wrong.

## 0.21.0-beta — 2026-08-27

**Reading a ledger back no longer floods the conversation, and on large ledgers it works again at
all.** Reviews used to pull every record and every parked matter into the chat to answer one
question. Past a certain size that stopped fitting, and the answer never arrived.

- **Ask about a topic and Memolok searches for it.** Records, matters, almanac facts, recorded
  outcomes and notes are all searchable by content now, and a record is found by the reasoning
  inside it — its verdict, an argument that was weighed, a consequence that was expected — not
  only by the need at its head.
- **Search finds words, not meaning.** A record that discusses your topic in different words will
  not come back, and the skills now say which words were tried rather than reporting "nothing was
  decided". Treat an empty result as *nobody wrote it that way*.
- **Results arrive a page at a time**, with a count of everything that matched. When an answer rests
  on part of a ledger, you are told which part.
- **Rows are excerpts.** Listings show the opening of each entry rather than all of it, and the full
  text is one read away. For your own parked matters this matters most: the excerpt is a trim of
  your words, so anything quoted back to you should come from the whole entry.
- **A new agent does the reading.** `memolok:ledger-scout` sweeps a ledger in its own workspace and
  returns the answer with its sources, so the sweep never reaches your conversation. It only reads,
  and it cannot ask you anything — which is why it never handles the parts where you are present.
  On surfaces without agents the skills read directly instead.
- **A mistyped status is refused instead of answered.** Asking for records in a status that does not
  exist used to come back empty, which read as *nothing was decided*.
- **Finding a note is one tool now**, not two. Browsing and searching your notes were separate calls
  that disagreed with each other about how many results there were.

**Needs the matching Memolok server**, which ships alongside this pack.

**An analysis can now take up a world fact or a recorded outcome directly.** Until now the only thing
an analysis could be about was a *matter* — raw input in someone's own words. So when a new
regulation, or a result that missed what a decision promised, was what actually prompted fresh
thinking, the only way to record that was to retype it as a matter. The ledger then held the same
thing twice and no trace that either led to the other, and matters cannot be edited, so it could not
be put right later.

The skills now route those cases to the entry itself. What changes for you:

- **Ask "what should we do about that?" of something already on your ledger, and the decision that
  follows will point back at it.** A fact in the almanac, an observed outcome from an earlier
  decision — the reasoning names it as what it took up, and a reader of the resulting record can
  walk back to it.
- **The intake fork stops treating premises and outcomes as exits.** Bringing up a world fact or a
  wake used to end the decision conversation and hand you to a different skill. It is now a detour:
  record it there, come back, decide.
- **One analysis can still take up any number of things, and now in any mixture** — a fresh report,
  the constraint that bounds the fix and the outcome that prompted someone to look, together in one
  act of reasoning.
- **Reviews are honest about the direction that is missing.** You can ask what a record's reasoning
  took up; there is no read that answers "what did this fact lead to?", and the skills say so rather
  than implying the ledger computed it.

Needs a Memolok server carrying the widened tool — **not yet deployed**, so none of this works
against the server running today. That ordering is not optional: skills that offer these inputs
against the current server would produce a "not found" on a live, correct id.

- **`wrap-up` now looks for observed outcomes**, not only for content the session originated. It will
  offer to record what you have learned about decisions already on your ledger — including records you
  only read in passing, and outcomes nothing in the session caused.
- **`help` answers instead of describing its own lookup.** It had picked up the habit of saying which
  reference it was about to open before getting to the point, which put the agent's process in front of
  your question. The reading is now invisible by rule, and the skill carries worked examples of what an
  answer sounds like — it had none, alone among the skills in the pack.
- **`help` also says what the agent is there to do.** Someone asking what Memolok is usually meets the
  rigour before they meet the reason for it: records that seal, a commitment ceremony of its own,
  expectations stated before the fact. Explained defensively that sounds like bureaucracy; explained as
  protection for the decider it sounds like the point. The skill now names closing that gap as the job.

## 0.20.0-beta — 2026-08-20

First `-beta` pack; every earlier release was `-alpha`. The track change is a statement about
maturity, not a mechanical difference.

- **New skill: `wrap-up`.** A working session establishes things that live nowhere but in the
  conversation, and when it ends they go with it silently. `wrap-up` finds them and routes each to
  whichever skill or tool already owns that kind of thing — it deliberately owns no procedure of its
  own. Two ideas do the work: record the durable fact behind an observation rather than the
  observation, and record reasoned content at the honesty of the session, never above it. Tidying
  reads as better writing and is the exact failure a ledger exists to prevent.
- **`help` substantially refreshed.** It had drifted into describing an older version of the product.
- **Corrections to the tool boundaries reference.** Three live tools were missing from the
  *Available* table. The matter-closure fields were also grouped as one case when they are two: some
  exist and are refused at the surface, others do not exist at all — the advice not to invent fields
  was right either way, but a reader who went looking found two different things.
- **`send-feedback`** now names `wrap-up` as the natural occasion to batch a report.

## 0.19.0-alpha — 2026-08-19

**You name every id; the server never mints one.** Required by a breaking change in the service, and
this pack ships *before* that server so there is no window in which your installed skills contradict
the server they reach. Payloads written to this pack's examples work against both.

- Every item in `alternatives`, `expectedOutcomes` and `openQuestions` must carry an `id` that you
  choose — a prefix per list, unique within it, present always. The patch examples now show it, and
  the error table covers the refusals you can meet.
- Why it is worth the keystrokes: these lists replace wholesale, so an item arriving without an `id`
  is equally readable as *keep this one* and *add a new one*. The server used to guess the second,
  minting a fresh id — which silently re-identified every option on the record and left the chosen
  alternative pointing at something that no longer existed, after reporting success. A name like
  `alt-cache-layer` is also what a reader meets months later; a minted `alt-kmzp0u1f` carries
  nothing.
- `get_guidance` now takes a `pluginVersion`, and the method skill instructs passing it. That is what
  lets the server tell your agent its pack is out of date at the start of a session, instead of your
  agent finding out from a refused write halfway through a task.

## 0.18.0-alpha — 2026-08-18

- **New skill: `send-feedback`.** Report a Memolok bug or suggestion to the team behind Memolok,
  directly from the session holding the evidence. It is the first skill whose destination is not your
  own ledger, so the routing table says so up front.
- **Preview-and-confirm is non-negotiable in it.** The skill sends prose about your session off your
  machine, so it always shows you what it is about to send first.
- `get_guidance` now reports **two** versions, the pack's and the server's. They move independently,
  and a bug report naming only one of them is half a report — quote the server version for server
  behaviour, the pack version for skill behaviour.

## 0.17.0-alpha — 2026-08-18

A vocabulary migration. Nothing is aliased, so this pack and the matching server go together.

- An analysis now records each input it took up as its own reference, alongside when it concluded.
- Matter status is **gone**, with all nine of its values. It was a cache over the graph's own shape
  that could disagree with what it shadowed — the disposition of a matter is now read from the
  topology, which cannot drift from itself.
- `save-matter`, `record-decision`, `review-ledger` and `grill-me` all teach a different inbox query
  as a result, and `record-decision` gains a rule against volunteering record-to-matter
  correspondence.
- **Update the server before this pack**, not after. An older pack against the new server gets a
  clear parameter error; this pack against an older server fails at the first tool call in four
  skills.

## 0.16.0-alpha — 2026-08-09

Makes **rejection** a shape the agent reaches for rather than an edge case. A decision not to do
something is a real commitment and deserves the same record as a decision to act.

- New reference on rejection, and one on write failures.
- The method skill was substantially rewritten and shortened.

## 0.15.0-alpha — 2026-08-08

- **New skill: `manage-notes`** — scratchpads, for freeform working material the ledger keeps but
  stands behind no claim about. Fully mutable, unlike everything else in the model.
- `help` gains a scratchpads reference; the method, catalog and boundary docs cover the new tools.

## 0.14.0-alpha — 2026-08-07

Clearer messaging in the method skill, and more on connecting and authenticating.

## 0.13.0-alpha — 2026-08-03

Adds `get_analysis` and its companions to the catalog, and teaches `review-ledger` to use them.

## 0.12.0-alpha — 2026-08-03

- **New skill: `revise-intent`** — state or revise what a ledger is *for*.
- Direct retrieval of a decision record by its ledger number, so an agent no longer has to list an
  entire ledger to resolve a number a practitioner knows by heart.

## 0.11.0-alpha — 2026-08-01

Version-check support in the method skill.

## 0.10.1-alpha — 2026-08-01

Improvements to `help`, mostly on how to introduce Memolok's vocabulary without burying a newcomer in
it. Adds the reference explaining why these particular words were chosen.

## 0.10.0-alpha — 2026-08-01

- **New skill: `help`** — what Memolok is, why records are sealed, what the vocabulary means, and the
  common objections, with a reference per topic.

## 0.9.0-alpha — 2026-08-01

First published pack. Skills for getting started, recording decisions, committing and revising them,
parking matters, managing the almanac, reviewing a ledger, recording outcomes, and the decision
interview.
