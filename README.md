# Memolok

**Memolok is the Decision Record platform**, and this is its plugin for Claude and Claude Code.
Memolok records decisions as Memolok Decision Records: structured, immutable entries capturing
the need, the alternatives, and the reasoning behind a choice. Each record lands in your
**Memolok Decision Ledger** (**MDL**) and commits to its *expected outcomes* when it is sealed,
after which it cannot be edited. What is *observed* later is recorded separately and linked back
to the record it tests.

Architecture Decision Records inspired the idea but stayed semi-structured and software-only.
Memolok generalizes the same discipline, a Decision Record, to any decision an organization or
its AI agents need to remember why.

Installing the plugin adds the Memolok skills and connects Claude to the Memolok server. Connecting
opens a browser to sign in: there is no token to configure and nothing to paste. You will need a
Memolok account; the plugin will offer to create your first **MDL** if you do not have one yet.

Start with `/memolok:start`, or just describe a decision you are making. If you would rather understand
the thinking first, `/memolok:help` explains it.

## Skills

| Skill | What it does |
| --- | --- |
| `/memolok:help` | Use it any time to understand what Memolok is, why it records decisions the way it does, and what the words mean |
| `/memolok:start` | Connect, sign in, and pick or create a ledger |
| `/memolok:record-decision` | Record a decision, from a raw matter or a sharpened need |
| `/memolok:save-matter` | Park something for later and carry on; pick it up in a later session |
| `/memolok:commit-decision` | Seal a decision as **Accepted** or **Rejected** |
| `/memolok:grill-me` | Be interviewed through a decision, one question at a time |
| `/memolok:review-ledger` | Read back what the ledger already holds |
| `/memolok:revise-decision` | Amend, uncommit, supersede, or settle an earlier open question |
| `/memolok:record-outcome` | Record what actually happened, against what was promised |
| `/memolok:manage-almanac` | Admit the world facts your decisions reason from |
| `/memolok:manage-notes` | Scratchpad management: save, find and destroy working notes |
| `/memolok:revise-intent` | Say what a ledger is for, or update it when the focus shifts |
| `/memolok:send-feedback` | Report a Memolok bug or suggestion to Memonos, **not** to your own ledger! |
| `/memolok:wrap-up` | Before a session ends, save what would otherwise be lost with it |

## Agents

| Agent | What it does |
| --- | --- |
| `memolok:ledger-scout` | Reads your ledger and answers one question about it, without the reading landing in your conversation |

Reviewing a ledger of any size means paging through records and reading a lot of them. On a Claude
surface that supports agents, that reading is handed to the scout: it reads in its own workspace,
which is discarded, and comes back with the answer and its sources. What reaches you is the answer,
not the sweep. This holds wherever a ledger is an input (reviewing one, finding the decision a
result belongs to, or checking what is already settled before an interview) and it holds when the
ledger is only one part of a larger job.

It **only ever reads.** It cannot record a decision, cannot change one, and cannot ask you anything,
so it never stands in for the part of Memolok where you are present. Where agents are not
available the skills simply do the reading themselves, and you should notice no difference beyond a
longer conversation.

Memolok's methodology (the decision lifecycle, what seals at commitment, and how records are
written) and its rules for referring to a decision outside Memolok, in your code and documents and
messages, are loaded automatically by the skills above; you do not invoke them directly.

## Learn More

- **[How it works](https://www.memolok.ai/how-it-works/):** the capture flow, the six-part MDR,
  and what happens after a decision is sealed
- **[What you use today](https://www.memolok.ai/tool-comparison/):** how Memolok compares with ADRs,
  business decision records, decision logs, and wikis
- **[Decision Record template](https://www.memolok.ai/decision-record-template/):** a free template
  for recording any decision, in any tool
- **[Glossary](https://www.memolok.ai/glossary/):** MDR, MDL, world fact, supersession, and the rest

Accounts and support: **[www.memolok.ai](https://www.memolok.ai/)**
