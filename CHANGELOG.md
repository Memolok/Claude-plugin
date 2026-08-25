# Changelog

What changed in the Memolok pack, newest first, written for someone deciding whether to update.

The pack is a set of skills: prose that changes what your agent does. So a wording change here is a
behaviour change, and every shipped edit gets a version — including documentation-only ones.

- **Patch** — corrections that do not change what the agent does.
- **Minor** — new skills, new tools reflected in the catalog, or changed facilitation behaviour.
- **Major** — an installed invocation stops resolving.

The pack version is independent of the Memolok server's. You install the pack on your schedule; the
server is deployed on ours, so the two cannot be held equal. The server publishes the oldest pack it
still accepts, and tells your agent at the start of a session whether the pack it is running still
matches — you are asked to update only when your pack genuinely stops working.

> Entries for `0.9.0-alpha` through `0.16.0-alpha` were written on 2026-08-25 from the release
> history rather than at the time, so they are terser than the ones after them. Nothing in them is
> reconstructed beyond what the tags and their contents show.

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
