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

**Five listing tools are gone, and every collection now has one read for choosing.** `list_MDRs`,
`list_matters`, `list_world_facts`, `list_observed_outcomes` and `list_scratchpads` have been removed
from the server. In their place `discover_MDRs`, `discover_matters`, `discover_world_facts`,
`discover_observed_outcomes` and `discover_scratchpads` each answer as a page to read rather than an
envelope of rows, with `get_*` still the read that returns an entry whole. **You must update:** a
pack older than this one names tools the server no longer has, and those calls fail.

**The pages carry what you need to act on what you find.** A decision record shows its handle beside
its number and the date it was decided; a matter names the analysis that took it up. Nothing has to
be opened merely to get an identifier a write tool will ask for.

**Notes are findable by what they are about, not only by the words in them.** Each one now carries a
title and a summary Memolok wrote, and search reaches those as well as the body — so a note can be
found by a term nobody typed into it. Until a note has a title, its heading is its opening words, as
before.

**Your agent now knows where a citation goes in a line, not just what it has to say.** It had the
address a reference must carry and nothing about the shape it takes in prose. It now puts the
identifier at the end of the clause it explains, groups several into one parenthesis, says out loud
when one decision narrows another, and leaves another project's decision numbers in that project's
own notation. Commit messages are covered for the first time — the identifier belongs in the body,
not the subject line — and all of it now lives in one place your agent can reach from any job,
instead of only while it is recording a decision.

**And it knows that citing comes after sealing.** Where you have it build against a decision that is
still a draft, the code cannot name that decision, because a draft has no number. Writing the
references in is a pass that begins once you have sealed it — so your agent will now tell you the
build is done and the references are what remains, rather than quietly sealing early to get itself a
number it can type. A draft is never referred to outside your conversation, for the same reason:
there is nothing durable there to point at.

**Writing a reference into your project commits your agent to protecting it.** Putting an identifier
into code, documentation or a commit message obliges it to tell Memolok, so the identifier cannot
later be released while your reference still points at it. Mentioning a decision elsewhere — a
ticket, a message, a reply — does not: that declaration cannot be undone, and your agent will ask you
rather than spend it on a passing mention.

**A cut heading no longer leaves an asterisk showing.** A heading trimmed part-way through a bold
run used to render two literal asterisks.

Every skill that used to reach for a listing tool now reaches for its page, `ledger-scout` included.

## 0.26.0-beta — 2026-09-08

**Decision records can be read as prose, which completes the set.** `discover_MDRs` answers the same
selection as `list_MDRs` — `status` filter included — as a page carrying each record's summary, the
terms it names, and its head Claim beside them. All four listings that hold typed entries now have a
prose sibling. `list_MDRs` rows gain a derived title, and `get_MDR` returns the summary and subjects
alongside the record.

**A heading and summary on a record come from its spine**: the head Claim, the alternative it chose,
and the Verdict. They have not read the options it rejected or the arguments about them, though search
reaches all of those — so a record can match a query its summary never mentions, and the match window
is what shows you why.

**And the pack now says plainly what a status means.** `Rejected` is a sealed commitment — the decision
was not to proceed — and `Superseded` means a later record replaced this one while what it decided
still happened. Reporting either as an unfinished record misreads the ledger, so the catalog, the
review skill and the scout all carry it.

**The wake can be read as prose too.** `discover_observed_outcomes` answers the same selection as
`list_observed_outcomes` — including narrowing to one record's wake with `mdrHandle` — as a page
carrying each observation's summary and the terms it names. A record with a long tail of
observations is the case this saves the most on. `list_observed_outcomes` rows now carry a derived
title, and `get_observed_outcome` returns the summary and subjects alongside the claim.

**A heading on an outcome describes what was observed and never judges the decision.** The derivation
is shown the observer's claim and nothing else — not the expectation the entry tests, not the record
it came from. Whether a promise held is `testResult`, which every read still carries. The skills and
the scout say so, because a heading that reads like a verdict is the one way this could mislead.

**World facts can be read the way matters can: as prose to choose from rather than rows to filter.**
`discover_world_facts` answers the same selection as `list_world_facts`, in the same order, with the
same identifiers — but as a page carrying each premise's summary and the terms it names. It is the
read for *which of these bears on what I am doing*, and it saves opening every candidate to find out.
`list_world_facts` rows now carry a derived title beside the admitter's excerpt, and `get_world_fact`
returns the summary and subjects alongside the claim.

Search reaches the derived title, summary and subjects as well as the admitted claim, so a premise can
now be found by a term nobody typed into it.

**A heading or summary on either read is Memolok's wording, not the admitter's — and on this
collection that matters more than anywhere else.** A world fact is a premise decisions are reasoned
from, so quoting a paraphrase of one back as the admitted claim misstates what the ledger rests on.
Every page says which it is showing you, and where nothing has been derived the heading is the
admitter's own opening rather than a placeholder. `get_world_fact` remains the only source for the
words somebody actually wrote.

**Ledger reads now say when things came into being, and the tool catalog says so.** Every discovery
row carries `createdAt`, so *"what came in this week"* is answerable by filtering rows rather than
opening each entry. `get_matter` and matter rows carry `raisedBy`; world-fact reads carry
`createdBy`.

The catalog is explicit that an absent `raisedBy` means *unrecorded*, never *anonymous* — entries
made before the server recorded a raiser have none and never will, because nothing in storage could
recover one and nothing was invented to fill the gap.

**Every routing row that means *choose* now points at a prose read.** The last one still sending the
unprocessed-matter inbox to rows has moved: picking what to work on from a queue is choosing, and an
agent that wants one matter whole has `get_matter` for exactly that.

**Both duplicate checks now read as prose.** Registering a matter and admitting a world fact each begin
by asking whether the same thing is already there, which is a judgement about what an entry is *about* —
and two people parking the same trouble rarely open with the same words, so a positional excerpt is the
wrong evidence. The almanac's check moved first; the matter one had been left behind.

**The cold-start signal says what to do when it is already there.** The invariant asked whether anything
in the session's own context said this project keeps a ledger, and only spelled out the *no* branch.
A yes now says so: ignore the section and do not mention it, because a signal working as intended is
not news, and reporting it spends the user's attention on a question they never asked.

The almanac skill now checks for near-duplicates with the prose read rather than the row read, because
that step is a judgement about whether a premise is already covered, and a row cannot support one.

## 0.25.0-beta — 2026-09-07

**There are two ways to change a decision that is already sealed, and they differ in what happens to
everything you did not mention.** An **amendment** keeps all of it, minus the part you changed. A
**supersession** keeps none of it — the successor governs alone, and may say far less than the record
it retired without that being an oversight. Whatever the new decision *adds* is valid either way, as
in any record.

**Amending is what is new here.** Until now a sealed record could only be taken back whole or
replaced whole, so a decision that still stood with one part gone stale had no honest route: taking
it back revised something nobody disputed, and replacing it retired something still in force. Now the
original stays in force, and a reader who opens it can see that something later modified it.

Because everything you leave alone carries forward, **write only what changes.** An amendment that
restates the original is nonsensical. Your agent knows this and will draft the difference rather than
a rewrite.

That also settles which kind of change counts: all of them. Tightening a commitment, widening one,
adding one that was missed, dropping one that no longer applies, or saying what a clause was always
meant to mean are the same act. You never have to classify the change — only decide whether the rest
of the record still governs.

The new route needs a recent enough Memolok server. Against an older one the write is refused, and
your agent tells you rather than failing quietly.

- **`revise-decision` asks a different first question.** It used to route on whether a record could
  still be withdrawn. It now asks how much of the record is wrong — nothing, part of it, or all of
  it — and treats withdrawability as a separate fact that only rules one of the three answers in or
  out. Asked the old way round, the ledger's state decided something only your situation can.
- **The route that used to have no answer now has one.** *The decision stands and shipped, but
  something it promised was wrong* previously landed on "neither route fits — record a separate
  decision beside it and expect no link back". That's what amendment is for. It stays
  link-less in one variant only: a follow-up that **declines** the promise rather than replacing it,
  since a rejection puts nothing in force and so can change nothing.
- **Amending is not the gentle option, and your agent now warns you before you take it.** Amending a
  decision counts as relying on it, so from that moment it can no longer be withdrawn on the record
  by anyone. It reads lighter than superseding and spends that option just as finally. If you might
  want the original back, take it back first.
- **Two more relationships can be recorded between decisions**: that one operationally relies on
  another, and that two stand in tension. The second carries a sharp edge worth reading before you
  use it — it applies to both records at once, including the one declaring it, and nothing undoes it.
- **Both routes need a decision that is still in force, at each end** — which follows from the
  distinction above, since one keeps that decision's content and the other retires it. So neither
  applies to a rejection, which put nothing in force, or to an already-superseded record, which has
  nothing left. The two new relationships are different: they relate the *decisions*, not
  what those decisions put in force, so a rejection can carry them both. That is less strange than it
  sounds — a decision to decline is still a decision, and later work can rest on it (*had we said yes
  to that, we could not do this*) or collide with it (opening a store in Tokyo after declining the
  Asian market).
- **Corrected: a relationship that no longer exists was still being named** in two places listing
  what makes a decision un-withdrawable. Your agent could have gone looking for something it would
  never find.

## 0.24.1-beta — 2026-09-07

**Wrapping up a session offers again to tell the next session this project uses Memolok** — but only
where the first offer went unanswered. Declining it still settles it.

## 0.24.0-beta — 2026-09-07

**Corrected: a ledger address and a user identifier are the same shape.** The tools catalog told
your agent that `mdlGuid` was *not* the shape listed for the other identifiers. Since ledger
addresses were re-minted it is exactly the shape of a `userId` — sixteen Crockford base32
characters, no prefix — and nothing about either string says which one it is. The advice is
unchanged and now has a reason behind it: treat it as opaque, and never substitute one for the
other. Your agent is also told that a ledger address quoted from an older session can fail in two
different ways, neither of which means you lack access.

**The project file that names your ledger can now carry the ledger's name and purpose as well, so a
session that opens cold is oriented before it asks you anything.** `.memolok/mdl.yml` takes
`mdlTitle` and `mdlIntent` beside `mdlGuid`. Only `mdlGuid` decides which ledger you are on; the
other two are copies kept for reading, and where one disagrees with the ledger your agent believes
the ledger and tells you the file has gone stale.

- **Your agent reads the nearest one, and offers to write it where it will still be found.** A folder
  holding several repositories used to bind all of them from a single file above — which works until
  somebody clones one repository on its own and every bare `MDR-7` inside it stops naming anything.
  Your agent now offers the file inside the repository instead, and says what the alternative costs
  rather than deciding for you. Keeping one file above several projects is still yours to choose.
- **A document's own frontmatter now outranks the project file**, so a document kept in one project
  can cite a decision recorded in another. Frontmatter takes `mdlTitle` beside `mdlGuid` too.
- **Rewriting what a ledger is for now offers to update the copy in the same breath.** Revising the
  purpose is the one action that makes a project file wrong the moment it succeeds, so your agent
  offers to refresh the file it can see rather than leaving a copy behind that quietly disagrees. It
  will not go hunting through folders it was not working in.

**Your agent can offer to tell the next session that this project uses Memolok, so you do not have
to.** With your say-so it puts a single paragraph wherever this project's sessions actually read
from. A session then reaches for the ledger without being told to, and finds out early if it cannot
reach it at all, rather than at the first decision worth recording.

- **Where it goes depends on where you work, because the same file is not read everywhere.** In
  Claude Code it is a file of the plugin's own, `.claude/rules/memolok.md`. In a Cowork or chat
  project your agent hands you the paragraph to paste into that project's instructions, and writes
  nothing itself. In a Cowork session working from a bare connected folder it goes into that folder's
  `CLAUDE.md`, between two marker comments, touching nothing else in the file. In a plain chat
  nothing would reach a later session, so nothing is offered.
- **You are asked separately, and told the cost.** One paragraph at the start of every session
  in that project, whether or not that session touches Memolok. Agreeing to save a ledger id is not
  agreeing to this; they are different questions and your agent asks them as two.
- **Whatever it writes, it owns — and nothing beyond that.** A file of its own is rewritten whole; a
  block in your `CLAUDE.md` is bounded by markers and nothing outside them is touched, ever.
  **Removing what it wrote is how you turn it off.** It holds no ledger content — no name, no
  purpose, no record numbers — so it never grows.
- **It does nothing to anyone without the plugin.** A colleague who opens the project without Memolok
  installed reads one clause that does not apply to them. Uninstalling leaves it in place,
  harmlessly.

## 0.23.0-beta — 2026-09-05

**Update required, and this pack and the Memolok server go together.** From this release the server
stops accepting older packs: an earlier one describes a call it refuses outright, and — the half that
matters more — reads several responses confidently wrong rather than failing. There is no version of
this worth staying on. If your agent reports that its pack is too old, or a call fails in a way its
examples do not predict, update.

**A matter, world fact, observed outcome or analysis now carries a short identifier of its own** — a
nine-character value like `mt_3kf9xq` instead of twenty-four hexadecimal characters, with the prefix
saying what it addresses. Nothing you type changes; the values
your agent quotes back to you do. An identifier written down during an older session may no longer
resolve, and your agent will now tell you that rather than guessing.

- **Withdrawing an analysis reference names the two things it joins** — the analysis and the input —
  instead of a reference id. There is no reference id anywhere on the surface any more.
- **A staged record can now be named `MDRh7`** instead of only described by its claim. An admitted one
  is still `MDR-7`, exactly as before. The two are different addresses and routinely point at different records, so the
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
