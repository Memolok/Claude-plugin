# Scopes and ledger bridging

## Four operational scopes

Fixed order: **record → ledger → project → world.** Scopes say where a relationship applies, not what
one ledger stores.

| Scope | Role | Dynamicity |
| --- | --- | --- |
| **Record** | One fish — deliberation frozen at t₀ | Sealed at commitment |
| **Ledger** | Inter-record graph and almanac within one epistemic boundary | Frozen per t₀; new admissions project forward |
| **Project** | Code, documents, configuration — outside the ledger | Evolves independently |
| **World** | External reality — tickets, regulation, markets | Most dynamic; Memolok witnesses admissions |

A ledger holds records, matters, analyses, almanac admissions, and inter-record links. It does not
own project artifacts or external systems.

## Epistemic boundaries

One organization has **many ledgers** — engineering, legal, an executive committee, a greenfield
project. Forcing a single company-wide ledger erases honest differences in worldview, which is
alignment theatre rather than alignment.

**Privacy protects honesty.** Deliberation published company-wide gets sanitized, and a sanitized
record is worth less than an honest one. Bounding who a record serves is a methodological feature,
not secrecy.

## Bridging

The legitimate cross-team handoff: a decision in ledger A is admitted as a **World Fact** in ledger B,
when B's owners judge it relevant. **A reference, not a merge.**

An executive committee adopts a remote-work policy — their record. Engineering admits *"company policy
permits remote work"* as a world fact, then opens its own fish for VPN architecture.

Bridges run both ways: engineering constraints can flow back into the executive ledger as world facts.

Practically, this means bridging is just `admit_world_fact` in the receiving ledger. There is no
cross-ledger tool, and no way to reference another ledger's record by id — you restate the relevant
claim as a premise in your own almanac.

## Project bridges

- A project artifact cites the record that justifies it.
- A Claim cites project artifacts as evidence.

Memolok witnesses those links; it is not the repository.

**The first direction is declarable.** `anchor_MDR` records that something outside the ledger cites a
record — its kind only, never its location — which is what stops an Uncommit releasing the number the
citation names. It is asserted by whoever writes the citation, not detected. Form and precondition:
the **`record-decision`** skill.

The second direction is not on the tool surface, and neither is freezing a cited artifact at t₀.

## Choosing a ledger

When a user belongs to several, pick by the **worldview the decision belongs to**, not by who is
speaking. A backend engineer deciding a hiring question is working in the people ledger, not the
engineering one.

When it is genuinely ambiguous, ask. Writing to the wrong ledger is not fatal — the record is honest
either way — but it puts the decision outside the boundary where its context lives, so later readers
lose the premises that made it make sense.
