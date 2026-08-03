# Drafting a ledger's stated purpose

Three things, in one short paragraph or a handful of lines:

| Element | The question it answers |
| --- | --- |
| **Purpose** | What is this ledger for? |
| **Audience** | Whose reasoning does it hold — which team, project, or person? |
| **Direction of travel** | Roughly where is this heading over the next while? |

Two shapes are equally valid, and which one fits is a property of the ledger, not a choice to impose:

> **Mission-shaped**, for a standing team or function — an enduring reason to exist.
> *"Harmonizing recruitment procedures across the three regional offices. Owned by People Ops. Moving
> from per-office custom processes toward one shared pipeline with local exceptions recorded rather
> than improvised."*

> **Vision-shaped**, for a project or initiative — a desired end state.
> *"Rebuilding the home media server so it survives a drive failure and a house fire. One hobbyist,
> weekends only. Heading toward simple, boring hardware with backups I actually test."*

Do not make a user choose between the two, and do not name the distinction to them.

## Length

Short. A few sentences, up to a short paragraph. If it runs past what someone would read before
starting work, it has stopped doing its job — and its job is orientation at the top of a cold session.

## No falsifiability bar — this is the one to get right

A head **Claim** must be falsifiable. **Ledger Intent is not, and must not be pushed toward it.**

*"Moving toward one shared pipeline"* is a perfectly good intent and a terrible **Claim** — nothing
could ever settle it, which is exactly right for a statement of direction. If you catch yourself
sharpening a purpose into something measurable, you are applying Rule G where it does not belong. There
is no quality gate here beyond being brief, honest, and useful to read.

## Offer a draft, never an empty prompt

*"What is this ledger's purpose?"*, asked cold, produces either a stall or a generic sentence nobody
benefits from. Propose something concrete from what the user has already told you, and let them correct
it — the same *draft first* discipline Rule E applies to the belly.

> **You:** I'll set this up as *"Rebuilding the home media server so it survives a drive failure —
> solo project, weekends."* Want to adjust that?

One round of correction is plenty. It is revisable forever; nobody needs to get it right now.

## How hard to push, by journey

The same elicitation is wrong in most of these. Match the journey the user is actually in.

| Journey | Posture |
| --- | --- |
| **`start`** | **Full.** Setup is why they are here. Agree the title, propose a drafted purpose, refine once, then create the ledger carrying both |
| **`record-decision`** | **Light.** They came to record a decision. Draft a purpose from what they already said, offer it in one line, take a "skip" without a second ask. Never delay the record for it |
| **`grill-me`** | **Light**, as above — and never spend an interview turn on it. The one-question-per-turn budget belongs to the decision |
| **`save-matter`** | **Minimal.** Title only. This journey exists to not stop the user; say the purpose can be set later and get out of the way |

**Never a gate, in any of them.** No journey may refuse or defer creating a ledger, parking a matter, or
recording a decision because the purpose has not been stated. An unstated purpose is a normal ledger.

## Revising later

`set_ledger_intent` **replaces the whole statement** — it does not append, and there is no history. Read
the current one with `get_MDL` first, then write the full replacement, not just the changed clause.

Revision is expected and cheap: as the project, team, or industry moves, the intent moves with it. There
is no version to bump and no correction chain to maintain.
