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
        "id": "cache-layer",
        "label": "Add Redis caching layer",
        "description": {
          "markdown": "Read-through Redis cache on hot read paths; roughly a two-week rollout.",
          "lang": "en"
        }
      },
      { "id": "scale-out", "label": "Horizontally scale the API pods" }
    ],
    "deliberationFacts": [
      {
        "onAlternative": "cache-layer",
        "description": {
          "markdown": "Directly addresses P99 read latency on the hot paths we measured.",
          "lang": "en"
        }
      },
      {
        "onAlternative": "scale-out",
        "description": {
          "markdown": "Raises throughput but leaves per-request latency roughly where it is.",
          "lang": "en"
        }
      }
    ],
    "chosenAlternative": "cache-layer",
    "verdict": {
      "description": {
        "markdown": "We will add a Redis read-through cache for hot read paths.",
        "lang": "en"
      }
    },
    "expectedOutcomes": [
      {
        "description": {
          "markdown": "P99 read latency stays under 200ms within 30 days of rollout.",
          "lang": "en"
        }
      }
    ],
    "openQuestions": [
      {
        "description": {
          "markdown": "Whether write-path latency needs separate treatment next quarter.",
          "lang": "en"
        }
      }
    ]
  }
}
```

An alternative may carry only `id` and `label` when the user never elaborated on it — `scale-out`
above. Do not invent substance they did not articulate.

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

## Reading ids back

The response mints ids you will need later:

- `expectedOutcomes[].id` (`eo-…`) — required by a later wake's `tests.outcomeId`
- `openQuestions[].id` (`oq-…`) — required by a later `settlesOpenQuestion`

## Errors

| Message | Cause |
| --- | --- |
| `Use transition_MDR_status to change Memolok Decision Record status.` | `status` in the patch |
| `Field {field} cannot be updated on a Memolok Decision Record.` | Unknown or frozen key |
| `No updatable fields were provided.` | Empty or no-op patch |
| `Cannot change the {label} on a ledger-resident Memolok Decision Record ({status}).` | Record already admitted — see `revise-decision` |
| `Inter-record graph links may only be authored on staged Memolok Decision Records.` | `supersedes` on a resident |
| `openQuestions[].settledIn cannot be set via MCP…` | Settlement is minted at admission, not patched |
| `A chosen alternative must reference one of the record's alternatives.` | Chosen id not in `alternatives` |

If a tier-1 patch is refused because the record is already admitted, that is the Decision Transaction
Principle working. Do not retry — route to **`revise-decision`**.
