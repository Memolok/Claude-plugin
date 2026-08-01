# Transition payloads

## Accepted — t₀

```json
{ "mdlGuid": "<mdlGuid>", "mdrHandle": 1, "status": "Accepted" }
```

Returns the full record with `decidedAt` set, `mdrNumber` assigned, graph reciprocals published, and
`retractable` computed.

## Rejected — t₀

```json
{ "mdlGuid": "<mdlGuid>", "mdrHandle": 1, "status": "Rejected" }
```

Needs a head Claim and a non-empty Verdict. Alternatives, `chosenAlternative`, and expected outcomes
are **not** required — though belly content from real deliberation is welcome evidence.

A record admitting as `Rejected` must not carry `supersedes`.

## Proposed — formal governance only

```json
{ "mdlGuid": "<mdlGuid>", "mdrHandle": 1, "status": "Proposed" }
```

Requires alternatives, a valid `chosenAlternative`, and a Verdict. **This is not t₀** — no `decidedAt`,
no number, still fully editable. Skip it entirely on informal work.

## Aliases

`transition_MDR_status` resolves `Decided`, `Settled`, and `Committed` to `Accepted`.

`create_MDR` does **not**. Sending `"status": "Committed"` there fails with a raw error that bypasses
the friendly error mapping, so always send canonical values at creation.

## Legal transitions

| From | To |
| --- | --- |
| New | Deliberating, Proposed |
| Deliberating | Proposed, Accepted, Rejected |
| Proposed | Accepted, Rejected |
| Accepted, Rejected, Superseded | — |

```
Cannot transition a Memolok Decision Record from {from} to {to}.
```

**Proposed cannot go back to Deliberating.** If a proposal needs rework, either patch it in place —
it is still staged — or reject it and mint a fresh record.

**Superseded is not a transition target.** A record reaches it only when a *different* record admits
carrying `supersedes` naming it.

## Gate errors

| Message | Fix |
| --- | --- |
| `A Memolok Decision Record must have a head Claim before this transition.` | Patch `hasNeed` |
| `At least one alternative is required before proposing a Memolok Decision Record.` | Patch `alternatives` |
| `A chosen alternative must reference one of the record's alternatives.` | `chosenAlternative` must equal an `alternatives[].id` |
| `A Verdict description is required before proposing a Memolok Decision Record.` | Patch `verdict.description.markdown` |
| `At least one expected outcome is required before accepting a Memolok Decision Record.` | Patch `expectedOutcomes` |
| `Each expected outcome must include description prose before accepting…` | An outcome is missing its prose |
| `A Verdict is required before rejecting a Memolok Decision Record.` | Rejection still needs its reasoning |
| `A Rejected Memolok Decision Record must not carry supersedes…` | Clear `supersedes`, or admit as Accepted |

Every one of these is fixable with an `update_MDR` while the record is still staged. Patch, then
return to the ceremony.

## Settling an open question at admission

A staged record can close a question left open on an **already-admitted** record:

```json
{
  "mdlGuid": "<mdlGuid>",
  "mdrHandle": 4,
  "patch": {
    "settlesOpenQuestion": [
      { "hostMdrNumber": 7, "openQuestionId": "oq-manifold-safety" }
    ]
  }
}
```

The edge is authored while staged and takes effect when *this* record admits, minting `settledIn` on
the older host.

`hostMdrNumber` is the **holder's** number, not this record's. If the holder is still staged it has no
number and nothing frozen to settle — update the holder directly instead. Full treatment lives in the
`revise-decision` skill.

## Verifying the seal

After the transition, `get_MDR` and check:

| Field | Expected |
| --- | --- |
| `status` | `Accepted` or `Rejected` |
| `mdrNumber` | An integer — this is now the user-facing name |
| `decidedAt` | A timestamp |
| `retractable` | `true` if nothing anchors it yet; `false` if something already does |
| `expectedOutcomes[].id` | `eo-…` values — a later wake needs these |

A tier-1 `update_MDR` should now fail. That failure is the Decision Transaction Principle working
correctly, not a bug to route around.
