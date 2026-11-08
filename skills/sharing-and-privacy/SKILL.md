---
name: sharing-and-privacy
description: >-
  Invite someone to read a Memolok ledger, give the consent a ledger needs before it can be shared, leave a ledger, or take your name off one for good. Use when the user says "share this ledger with Ana", "invite him to read it", "why can't I share it?", "what is this consent email?", "take me off this ledger", "remove my name from it", "I don't want to be on that ledger any more". Picking or creating a ledger → `start`; who did what on an entry → `review-ledger`.
argument-hint: "[who to invite, or which ledger to leave]"
---

# /memolok:sharing-and-privacy — Who reads a ledger, and what it shows of you

Prepares the requests that change who is on a ledger and what it shows of a person: an invitation, the creator's consent to sharing it, leaving, and vanishing. **Every one of them but leaving is finished by the person themself**, on their own Sharing and privacy page in the Memolok portal, from the mail Memolok sends them. This skill gets them that far and tells them what is left.

## Usage

```
/memolok:sharing-and-privacy [who to invite, or which ledger to leave]
```

Examples:

- `/memolok:sharing-and-privacy invite Ana to read the platform ledger`
- `/memolok:sharing-and-privacy why can't I share this ledger?`
- `/memolok:sharing-and-privacy take me off the old migration ledger`
- `/memolok:sharing-and-privacy remove my name from everything on that ledger`

## Step 0 — Load the method

Load the **`memolok-method`** skill. The tools' contract (`share_MDL`, `request_MDL_disclosure`, `leave_MDL`, `request_MDL_vanish`) and every refusal code below are in its `tools-catalog.md`.

## Non-negotiables

- **No tool confirms, accepts or withdraws anything.** Never say an invitation, a consent or a vanish is done when you have only prepared it; say what the person still has to do
- Decide from a refusal's `Code:`, never from its sentence's words, and quote the sentence to the person
- `leaveWithNoOwner: true` only after telling the person the ledger will be left with no owner and hearing them agree; never on the refusal alone, never by default
- Leaving and vanishing are the caller's own. Nobody leaves or vanishes for someone else
- Confirm the ledger, by title, before any of these calls; never invent an `mdlGuid` or an `iri`

## What this skill is not

| The user wants to | Where |
| --- | --- |
| Pick, find or create a ledger | **`start`** |
| Remove someone else, change a role, or give one wider than `visitor` | No tool: a Memolok administrator's |
| Know who created or changed an entry | **`review-ledger`** |

## Workflow

### 1. Establish the ledger

The `mdlGuid` in play, else `.memolok/mdl.yml`, else `get_MDLs`. `get_MDL` gives the user's `role`, and `members` with each one's roles where the role may list them.

**A ledger the user has left no longer appears in `get_MDLs`**, and every read of it answers not found. They can still vanish from it while Memolok credits them with something there: ask for its address (a `.memolok/mdl.yml`, a link they kept) rather than concluding it is gone.

### 2. Invite someone to read it

Only an `owner` or `admin` can, and only as `visitor`: reads everything but notes, changes nothing.

1. **Get their `iri`.** There is no lookup by email or name: they read theirs with `whoami` in their own session and send it. A member of another ledger you share is in that ledger's `members`.
2. **Confirm the ledger and the person**, then `share_MDL(mdlGuid, iri)`.
3. **Say what happens next**, from `invited`:
   - `true`: *"I've prepared the invitation. Memolok will mail you to confirm it, then mail them to accept; nothing changes until both of you have, and an invitation you don't confirm within seven days lapses."*
   - `false`: a demo account, which reads the ledger now.

| Code | Say and do |
| --- | --- |
| `not-disclosed` | The ledger cannot be shared until its creator consents to it showing their name. The user is the creator → step 3. Anyone else → only the creator can, so the user asks them |
| `already-invited` | An invitation of theirs waits for that person already; to send another, they withdraw it on their Sharing and privacy page |
| `already-member`, `holds-another-role` | That person holds a role there already, which an invitation never changes |
| `unknown-account` | No account has that `iri`: ask for it again, from their `whoami` |
| `mail-unavailable` | The invitation stands but its mail did not go: the same call again sends it |

### 3. The creator's consent

Everyone invited to a ledger sees its creator's name, so Memolok asks the creator first, by mail, and nothing on the ledger can be shared until they confirm. It is asked at creation (`start` says so then); `request_MDL_disclosure(mdlGuid)` asks again, for a ledger created before consent was asked, a request that lapsed after seven days, or one whose mail did not arrive.

Only the creator can ask. To anyone else it answers not found, which says nothing about whether the ledger exists. `already-disclosed`: they confirmed already, and nothing is in the way. Otherwise: *"Memolok has mailed you a link. Once you confirm on your Sharing and privacy page, I can invite people."*

### 4. Leaving or vanishing: say the difference first

*"Take me off this ledger"* can mean either, and they are not alike:

| | Leaving | Vanishing |
| --- | --- | --- |
| Takes effect | At once | Only when they confirm from the mail, on their Sharing and privacy page; lapses after seven days |
| Their role | Gone; only a new invitation brings it back | Gone |
| Their name on what they wrote | Stays | A pseudonym in its place, on everything the ledger held then |
| Undone by | Being invited again | **Nothing, ever** |

Put the table's substance to them in two sentences and ask which, unless their words already settle it (*"remove my name"* is vanishing). Then confirm the ledger by title.

- **Leaving:** `leave_MDL(mdlGuid)`. Destructive and immediate: after it, this session cannot read the ledger either. Say it is done only once it returned `left`.
- **Vanishing:** `request_MDL_vanish(mdlGuid)` prepares it. *"Memolok has mailed you a link. Nothing happens until you confirm it on your Sharing and privacy page, within seven days, and once you do it cannot be undone."* `already-vanished`: they vanished already, and nothing names them there since.

**The last owner.** Either call is refused with `last-owner` when the user is the ledger's only owner. Say it plainly: *"You're this ledger's last owner. If you go, it is left with no owner, and only a Memolok administrator can make someone else one. Go anyway?"* Only on a clear yes, the same call again with `leaveWithNoOwner: true`. Where `members` shows they are the only owner, say so before the first call rather than waiting for the refusal.

## Tips

- Withdrawing an invitation, confirming one and accepting one all happen on the Sharing and privacy page. Point there; there is nothing to call.
