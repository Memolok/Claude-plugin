---
name: send-feedback
description: >-
  Send feedback about Memolok to Memonos, the team behind the project — a bug you hit, an
  inconsistency you noticed, a skill that contradicts a tool, something the user wanted
  and could not find. Use when a Memolok tool fails or misbehaves mid-task, when the user
  says "report this to Memolok", "tell them this is broken", "that's a bug", "they should add X",
  or at the end of a session where Memolok itself got in the way.
  Not for the user's own ledger — this only goes to Memonos.
argument-hint: "<what went wrong, or what should exist>"
---

# /memolok:send-feedback — Report to Memonos

Sends bug reports and suggestions about **Memolok itself** to Memonos, the team behind Memolok, through the same
authenticated MCP surface everything else uses. Batched: a session's findings go in one call.

This is the only journey in the pack that writes somewhere other than the user's ledger. Nothing here
touches an MDL, nothing becomes a Matter, and nothing is citable from any record.

## Usage

```
/memolok:send-feedback <what went wrong, or what should exist>
```

Examples:

- `/memolok:send-feedback update_MDR returned 200 but dropped my chosenAlternative`
- `/memolok:send-feedback the record-decision skill says something the tools don't actually support`
- `/memolok:send-feedback there's no way to rename a ledger after creating it`
- `/memolok:send-feedback tell Memonos their error messages don't say which field failed`

## Step 0 — Load the method

Load the **`memolok-method`** skill for prose format and the capture-versus-draft rule. This journey
writes, so the invariants apply — but note that the fish model does **not**: a feedback report is not
a decision and has no Need, Verdict or tail.

## Non-negotiables

- **Show the user the payload and get an explicit yes before sending.** Every time. This transmits
  prose about their session off their machine
- Never invent a `feedbackId` — the server mints them
- All prose is `{ "markdown": "...", "lang": "en" }`
- Never put the user's dictated words through your own phrasing — they go in `userVerbatim` intact
- Never claim a version you did not read; the server records its own
- Say "sent" only after the write succeeded
- Keep the returned ids in the session — nothing lists or searches reports

## When this skill applies

| Situation | Skill |
| --- | --- |
| Something about **Memolok** is wrong, missing, or confusing | **This one** |
| Something about **the user's own project** needs recording | **`save-matter`** or **`record-decision`** |
| The user hit a Memolok bug and wants it on *their* ledger too | Both — they are different destinations, not alternatives |
| Memolok auth is broken and no tool works | Neither. Point them at <https://www.memolok.ai/contact/> |

That last row matters: this channel rides Memolok's authenticated surface, so it cannot carry a
report that the authentication is broken. The published contact routes exist for exactly that.

## What is worth reporting

The obvious case is a call that failed. Three less obvious ones are the reason this skill exists,
because nothing errors in any of them and no exception handler could catch them:

1. **A call succeeded and did the wrong thing.** A patch returns 200 and silently drops a field. The
   status code says nothing is wrong; the state says otherwise.
2. **Two successful calls disagree.** A record a listing just returned cannot be read back. A note
   that demonstrably exists is not found by search. Neither call failed — only their conjunction is
   wrong.
3. **A skill contradicts a tool, or the design is simply absent.** An instruction promises something
   the surface does not do, prose is ambiguous, or the user wanted a capability nobody built.

**Recognising these is your judgment, and nothing else will do it.** You are the only party holding
the evidence at the moment it exists. A user will not write it up, and a later session cannot: it
starts cold.

Report it even if you might be wrong. A mistaken report costs Memolok one delete; an unreported fault
costs everyone the next occurrence.

## Workflow

### 1. Gather what you are still holding

Before composing anything, collect from **this session**: the exact calls and their responses, any
`requestId` you were given, the skill or tool names in play, and the versions you can actually read —
the plugin version from the `memolok-method` skill, and the server version from `get_guidance`.

Do not go looking for more by re-running calls against the user's ledger. Report what happened, not a
reconstruction.

### 2. Compose the batch

One `submit_feedback` call carrying every finding, each as an item in `reports`:

```json
{
  "reports": [
    {
      "title": "update_MDR orphans chosenAlternative when it mints an alternative id",
      "kind": "bug",
      "report": { "markdown": "Patched `alternatives` with items that had no `id`. The response minted ids and `chosenAlternative` was gone.", "lang": "en" },
      "artifacts": [
        { "kind": "mcp_tool", "name": "update_MDR" },
        { "kind": "mcp_server", "name": "Memolok MCP", "version": "0.2.0" }
      ],
      "evidence": [
        {
          "occurredAt": "2026-08-18T09:12:44Z",
          "toolName": "update_MDR",
          "request": { "markdown": "```json\n{\"alternatives\": [{\"label\": \"A\"}]}\n```", "lang": "en" },
          "response": { "markdown": "200; alternatives gained ids, chosenAlternative absent", "lang": "en" },
          "requestId": "req_abc123"
        },
        {
          "occurredAt": "2026-08-18T09:12:59Z",
          "toolName": "get_MDR",
          "response": { "markdown": "chosenAlternative: null", "lang": "en" }
        }
      ],
      "expectation": { "markdown": "chosenAlternative names an alternative that still exists after a patch.", "lang": "en" }
    }
  ]
}
```

**`title`** is the only thing the user sees in the confirmation and the only handle you will have
afterwards. Make it recognisable standing alone — *"update_MDR orphans chosenAlternative"*, not
*"a bug"*.

**`artifacts` names what was in play, never what is at fault.** A skill contradicting a tool is
**two** artifacts. Do not pick a culprit; the pair is the report.

**`evidence` items come in two shapes**, and both may appear in one report:

| Shape | Fields | For |
| --- | --- | --- |
| Call record | `occurredAt`, `toolName`, `request`, `response`, `requestId` | Something a tool did |
| Citation | `source`, `excerpt` | Something a skill, doc or page *says* |

Every item must carry a `response` or an `excerpt`. An item saying only that a call happened tells a
reader nothing.

**`expectation`** is for the inconsistency case: the invariant you believed held between the calls.
It exists only in your head — nothing on the wire implies it — so if you leave it out, the reader
sees two ordinary responses and no reason they conflict.

**`userVerbatim`** carries the practitioner's words when they dictated something. Copy them exactly.
Your own write-up goes in `report`, beside it, never merged into it.

### 3. Preview, then send

Show the payload — or a faithful summary of every report in it, including anything quoted from their
work — and ask. Send only on a yes.

If the user wants something removed, remove it and show the result again. If they decline outright,
drop it; do not file a reduced version instead.

### 4. Report back

Give them the titles and their ids, and keep the ids in the session:

> Sent two reports to Memonos — `fb_gt71…` *update_MDR orphans chosenAlternative*, and `fb_k39c…`
> *No way to rename an MDL title*.

### 5. Correcting a report

`update_feedback(feedbackId, patch)` — your own reports, no time limit, for as long as you hold the
id. **Arrays replace rather than merge**, exactly as `update_MDR` works: to add one evidence item,
send the whole list back.

`get_feedback(feedbackId)` reads one back. Both may legitimately return not-found for a report that
once existed — triage culls duplicates and noise by deleting — so treat an id as a handle, not a
guarantee.

## Field quality

A report is held to what a cold reader needs, because the schema enforces almost none of it. The
person reading this has none of your context and cannot ask you anything.

**Thin, and typical:**

> `list_scratchpads` seems broken, it didn't find my note.

**What the same finding looks like as a handoff:**

> Created a scratchpad, then searched for a distinctive word from its body and got an empty result.
> Both calls returned 200. Evidence: the create response with the minted `sp_` id, and the search
> response with `total: 0` and zero rows. Expected: a note created in this ledger seconds earlier is
> findable by a word in its body.

The difference is not length. It is that the second one can be acted on without a reply.

## Scope

**Within-session only.** This channel carries what you observed in the session you are in. Noticing
that Memolok has got *worse over time* — a skill that used to read better, a workflow that used to be
smoother — is deliberately out of scope: you start each session cold and cannot honestly compare.

**Duplicates are not your problem.** Do not check whether something was already reported; you have no
way to, and Memonos collapses repeats during triage. File it.

## Tips

- Batch at the end of a working session rather than interrupting mid-task — unless the user asks for
  it now, or the session is about to end. **`wrap-up`** is the skill that runs at that moment, and it
  routes here as one destination among several.
- A suggestion needs no evidence at all. `title`, `kind` and `report` are the whole requirement.
- `mdlGuid` is worth including when the fault happened in a ledger context. It is stored as a
  reproduction hint and resolves nothing.
- If a call is worth reporting *and* worth recording on the user's own ledger, do both. They serve
  different readers.
