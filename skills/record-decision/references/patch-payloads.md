# Patching the fish body

`update_MDR` requires the record to be **staged**. Patches are incremental — send only what changed.

## Full body at Deliberating

```json
{
  "mdlGuid": "<mdlGuid>",
  "mdrHandle": 1,
  "patch": {
    "alternatives": [
      {
        "id": "alt-cache-layer",
        "label": "Add Redis caching layer",
        "description": {
          "markdown": "Read-through Redis cache on hot read paths; roughly a two-week rollout.",
          "lang": "en"
        }
      },
      { "id": "alt-scale-out", "label": "Horizontally scale the API pods" }
    ],
    "deliberationFacts": [
      {
        "onAlternative": "alt-cache-layer",
        "description": {
          "markdown": "Directly addresses P99 read latency on the hot paths we measured.",
          "lang": "en"
        }
      },
      {
        "onAlternative": "alt-scale-out",
        "description": {
          "markdown": "Raises throughput but leaves per-request latency roughly where it is.",
          "lang": "en"
        }
      }
    ],
    "chosenAlternative": "alt-cache-layer",
    "verdict": {
      "description": {
        "markdown": "We will add a Redis read-through cache for hot read paths.",
        "lang": "en"
      }
    },
    "expectedOutcomes": [
      {
        "id": "eo-read-latency",
        "description": {
          "markdown": "P99 read latency stays under 200ms within 30 days of rollout.",
          "lang": "en"
        }
      }
    ],
    "openQuestions": [
      {
        "id": "oq-write-path-latency",
        "description": {
          "markdown": "Whether write-path latency needs separate treatment next quarter.",
          "lang": "en"
        }
      }
    ]
  }
}
```

An alternative may carry only `id` and `label` when the user never elaborated on it —
`alt-scale-out` above. Do not invent substance they did not articulate.

## You name every id

`alternatives`, `expectedOutcomes` and `openQuestions` replace wholesale, so each item has to say
which one it is. You choose the id, on `create_MDR` and `update_MDR` alike:

| Rule | |
| --- | --- |
| Prefix per list | `alt-` on alternatives, `eo-` on expected outcomes, `oq-` on open questions |
| Unique within its list | two items sharing an id make `chosenAlternative` ambiguous |
| Keeping an item | send the `id` it already has |
| Adding one | send a new `id` you chose |

**Name them for what they are.** `alt-cache-layer` is what a reader meets in `chosenAlternative`
months later, and in every deliberation fact arguing about it. You know what the option is; a
generated `alt-kmzp0u1f` would carry none of that.

Omitting an `id` is refused. On a list that replaces wholesale it reads equally as *keep this* and
*add this*, and the server used to resolve it as the second — minting fresh ids and leaving
`chosenAlternative` pointing at an option that no longer existed, on a write that reported success.

A missing prefix is refused too, naming the corrected form. The server will not add it for you:
rewriting an id you chose would orphan any reference to it that the same patch does not carry, which
is the very failure the rule exists to prevent.

## Revising the Need

Legal and expected while staged. De-sharpening a Need that absorbed the answer is the correct move,
not a retreat.

```json
{
  "mdlGuid": "<mdlGuid>",
  "mdrHandle": 1,
  "patch": {
    "hasNeed": {
      "description": {
        "markdown": "The chosen language must have a mature, well-supported statistical-analysis library ecosystem.",
        "lang": "en"
      }
    }
  }
}
```

`hasNeed` uniquely accepts either the wrapped form above or a bare `{ markdown, lang }`.

## Incremental patch

```json
{
  "mdlGuid": "<mdlGuid>",
  "mdrHandle": 1,
  "patch": {
    "openQuestions": [
      {
        "id": "oq-mobile-transcoding",
        "description": {
          "markdown": "Whether we transcode for mobile clients at all.",
          "lang": "en"
        }
      }
    ]
  }
}
```

Array fields **replace**, they do not append. Send the full intended list for any array you touch.

## Attribution

```json
{
  "mdlGuid": "<mdlGuid>",
  "mdrHandle": 1,
  "patch": {
    "decidedBy": "https://www.memolok.ai/users/<userId>",
    "consulted": ["https://www.memolok.ai/users/<otherUserId>"]
  }
}
```

RACI fields stay patchable after t₀ — they are the only ones that do.

## Ids you will need later

The ids you chose are stored verbatim and come back on every read. Two of them are cited by later
calls, so pick names that will still mean something by then:

- `expectedOutcomes[].id` (`eo-…`) — cited by a later wake's `tests.outcomeId`
- `openQuestions[].id` (`oq-…`) — cited by a later `settlesOpenQuestion`

## Errors

| Message | Cause |
| --- | --- |
| `Use transition_MDR_status to change Memolok Decision Record status.` | `status` in the patch |
| `Field {field} cannot be updated on a Memolok Decision Record.` | Unknown or frozen key |
| `No updatable fields were provided.` | Empty or no-op patch |
| `Cannot change the {label} on a ledger-resident Memolok Decision Record ({status}).` | Record already admitted — see `revise-decision` |
| `Inter-record graph links may only be authored on staged Memolok Decision Records.` | `supersedes` on a resident |
| `openQuestions[].settledIn cannot be set via MCP…` | Settlement is minted at admission, not patched |
| `The chosen alternative '…' does not name one of the record's alternatives.` | `chosenAlternative` names nothing the record has — resend both sides together |
| `deliberationFacts[…].onAlternative '…' does not name one of the record's alternatives.` | Same, for an argument attached to an option that is gone |
| `…id is required.` | An item on `alternatives`, `expectedOutcomes` or `openQuestions` carries no `id` |
| `…id '…' must start with '…' — did you mean '…'?` | Missing the list's prefix; the message names the correction |
| `…id '…' is used twice in this list.` | Two items claiming one id — a reference to it could not say which |

If a tier-1 patch is refused because the record is already admitted, that is the Decision Transaction
Principle working. Do not retry — route to **`revise-decision`**.
