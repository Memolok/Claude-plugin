---
name: start
description: >-
  Get set up with Memolok — check the connection, sign in, and pick or create a decision ledger. Use
  when someone is trying Memolok for the first time, when a Memolok tool returns an authentication
  error, when Memolok "isn't connecting", or when they need to choose between several ledgers or
  create their first one.
---

# /memolok:start — Get set up

Verifies the connection, establishes which ledger the session is working in, and orients the user.

## Usage

```
/memolok:start
```

Examples:

- `/memolok:start`
- `/memolok:start set me up`
- `/memolok:start which ledger am I using?`
- `/memolok:start memolok isn't connecting`

## Step 1 — Say what this is

Briefly, and only if they have not used it before:

> Memolok records **decisions** — not just what you chose, but what you were trying to achieve, what
> else you considered, and what you expected to happen. The point is that in six months the reasoning
> is still there, not just the result.

Do not lecture. One or two sentences, then move on. If what they actually want is to understand the
methodology rather than get connected, hand to **`help`** — that is its whole job.

## Step 2 — Check the connection

```
ping → "pong"
```

If that fails, the connection is not established — go to `references/connect-and-auth.md`. Do not
attempt ledger calls; every one of them will fail the same way.

Optionally `whoami` to confirm identity, which is also what supplies the user IRI for attribution.

## Step 3 — Find the ledger

```
get_MDLs → { mdls: [{ mdlGuid, title, role }] }
```

| Result | Do |
| --- | --- |
| Exactly one | Use it. Mention its name, do not make them choose |
| Several | Ask which, listing them by title |
| Empty | Offer to create one — step 4 |

If project files are available, check `.memolok/mdl.yml` for a `mdlGuid` key first; that is the
project's own answer and beats asking.

Once the ledger is settled, read it:

```
get_MDL(mdlGuid) → { mdlGuid, title, role, ledgerIntent }
```

If it states a purpose, say it back in a line — that is the orientation the user came for. If
`ledgerIntent` is `null`, mention that the ledger has not said what it is for and offer to write it;
most ledgers have not, so this is an invitation, not a problem report.

| Role | Means |
| --- | --- |
| `owner`, `admin` | Full access, including uncommitting records |
| `member` | Can read and write |
| `visitor` | Read-only — writes will be refused |

If they are a `visitor`, say so now rather than letting them build a decision they cannot save.

## Step 4 — Create a ledger, if needed

A ledger is an **epistemic boundary** — a set of decisions sharing a context and a worldview. One
project, one team, one initiative. Not one per person, and not one per company.

Agree a title **and a purpose** before calling anything. This is the moment for it: setup is why the
user is here, and stating it now costs one call instead of two. A purpose is three things in a short
paragraph — what the ledger is for, whose reasoning it holds, roughly where it is heading — and it may
read as a mission (*"Harmonizing recruitment across the three regional offices. Owned by People Ops."*)
or as a vision (*"Rebuilding the home media server so it survives a drive failure. One hobbyist,
weekends only."*); do not make the user choose between the two. It has no falsifiability bar: *"moving
toward one shared pipeline"* is a good purpose and would be a poor head Claim. Propose a draft from
what they have already said and let them correct it once; asked cold, *"what is this ledger for?"*
produces a stall or a generic sentence.

```json
{
  "title": "Media server rebuild",
  "ledgerIntent": {
    "description": {
      "markdown": "Rebuilding the home media server so it survives a drive failure and a house fire. One hobbyist, weekends only.",
      "lang": "en"
    }
  }
}
```

They become `owner`. Adding anyone else is done by a Memolok administrator, not through this plugin.

**The purpose is optional.** If the user does not want to write one now, create the ledger with the
title alone and move on — it can be stated any time with **`revise-intent`**. Never hold up a ledger
over it.

## Step 5 — Offer to remember it

If project files are writable and there is no `.memolok/mdl.yml` where one would be read, offer to
save the ledger so future sessions skip this. Step 3 already fetched everything it needs:

```yaml
mdlGuid: <the guid>
mdlTitle: <the title>
mdlIntent: |-
  <ledgerIntent's markdown, verbatim>
```

**`mdlGuid` is the only key that decides anything.** The other two spare a cold session a call, and
they are copies rather than authority — `get_MDL` is what a session believes when the two differ.
Copy them exactly as it returned them, one line per paragraph and no rewrapping, so a stale file
shows up as a plain diff instead of needing a reader to judge whether the wording drifted. Omit
`mdlIntent` when the ledger has not stated a purpose.

Ask first. Writing files into someone's project uninvited is not yours to decide.

**A yes here is a yes to this file only.** The cold-start signal changes what every future session in
the project does before anybody asks it anything, which is a different question and gets asked as
one: *"Want me to set this project up so future sessions know it keeps a ledger, without you having
to say so? Short paragraph — I'd work out where it belongs."* Name no file, directory or format. Only
on a yes, load **`memolok-method`** and its `cold-start-signal.md`, which decide where it goes. This
is the one journey where both offers can land in the same breath; a no on either leaves the other
open.

**Where to offer it, when the tree holds several repositories.** In the one the user is working in,
not the folder above them. A bare `MDR-7` in a source file resolves through the nearest
`.memolok/mdl.yml` above that file, and only a file inside the repository is still there for someone
who cloned it on its own. A single file above several repositories is a legitimate arrangement and
the user's to choose — say what it costs, once, and take their answer.

## Step 6 — Point at the work

Suggest a starting point based on what they came for:

| They want to | Skill |
| --- | --- |
| Understand what Memolok is and why it works this way | **`help`** |
| Record a decision they have made or are making | **`record-decision`** |
| Be interviewed through a decision they have not worked out | **`grill-me`** |
| Note a problem and get back to work | **`save-matter`** |
| See what is already on the ledger | **`review-ledger`** |
| Seal a decision that is drafted | **`commit-decision`** |
| Change something already sealed | **`revise-decision`** |
| Say what the ledger itself is for, or update it | **`revise-intent`** |
| Record how a past decision turned out | **`record-outcome`** |
| State a premise the ledger should know | **`manage-almanac`** |

Then hand off. Do not walk them through the whole list.

## Tips

- The plugin ships the server URL and nothing else. Connecting opens a browser sign-in — there is no
  token to paste and no client id to configure.
- A ledger with no records is normal, not broken.
- If the user has decisions already scattered in a repository or a wiki, recording them is
  **`record-decision`** like any other — a decision made last year is still worth a record, and the
  reasoning is usually still recoverable from whoever made it.
- Non-members get `Memolok Decision Ledger not found.` rather than a permission error, deliberately —
  it does not reveal whether the ledger exists. If someone expected access, an administrator needs to
  add them.
- The cold-start signal lands wherever a project's sessions actually read from, which differs by
  surface, and you do not need to know the place before offering. **Removing what was written is how
  they opt out**, and it only comes back if they accept the offer again. Uninstalling the plugin
  leaves it in place, harmlessly — it instructs nothing without the skills to act on it.

## References

| File | Load when |
| --- | --- |
| `references/connect-and-auth.md` | `ping` fails, sign-in loops, or the client has no browser flow |
