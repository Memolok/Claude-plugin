# Memolok

The Memolok plugin for Claude. It turns decision work into recorded, reviewable institutional
memory: Claude facilitates capture, and the result lands in your **Memolok Decision Ledger** (**MDL**)
as durable **Memolok Decision Records** (**MDRs**) rather than as notes in a chat log.

Installing the plugin adds the Memolok skills and connects Claude to the Memolok server. Connecting
opens a browser to sign in — there is no token to configure and nothing to paste. You will need a
Memolok account; the plugin will offer to create your first **MDL** if you do not have one yet.

Start with `/memolok:start`, or just describe a decision you are making.

## Skills

| Skill | What it does |
| --- | --- |
| `/memolok:start` | Connect, pick or create a ledger, and get oriented |
| `/memolok:record-decision` | Record a decision — from a raw matter or a sharpened need |
| `/memolok:save-matter` | Park something for later and carry on; pick it up in a later session |
| `/memolok:commit-decision` | Seal a decision as **Accepted** or **Rejected** |
| `/memolok:grill-me` | Be interviewed through a decision, one question at a time |
| `/memolok:review-ledger` | Read back what the ledger already holds |
| `/memolok:revise-decision` | Uncommit, supersede, or settle an earlier open question |
| `/memolok:record-outcome` | Record what actually happened, against what was promised |
| `/memolok:manage-almanac` | Admit the world facts your decisions reason from |

Memolok's methodology — the decision lifecycle, what seals at commitment, and how records are
written — is loaded automatically by the skills above; you do not invoke it directly.

Documentation, accounts, and support: **[www.memolok.ai](https://www.memolok.ai)**
