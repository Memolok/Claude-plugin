# Wake payloads

## Expected — assessing a tail commitment

Requires both `tests` and `testResult`.

```json
{
  "mdlGuid": "<mdlGuid>",
  "mdrNumber": 7,
  "claimDescription": {
    "markdown": "P99 read latency measured 410ms during the December peak, against the 200ms target.",
    "lang": "en"
  },
  "discoveryType": "Expected",
  "tests": { "outcomeId": "eo-read-latency" },
  "testResult": "Violated"
}
```

`tests` also accepts a bare id string, and names nothing but the expected outcome: the record is the
one `mdrNumber` names. Get the `eo-…` value from `get_MDR_learning_delta` on the source record.

## Emergent — nobody saw it coming

No `tests`, no `testResult`.

```json
{
  "mdlGuid": "<mdlGuid>",
  "mdrNumber": 7,
  "claimDescription": {
    "markdown": "The cache layer turned out to mask a connection-pool leak that had been present for months; it surfaced only when we bypassed the cache during an incident.",
    "lang": "en"
  },
  "discoveryType": "Emergent"
}
```

## Deducible — foreseeable, but not written down

```json
{
  "mdlGuid": "<mdlGuid>",
  "mdrNumber": 7,
  "claimDescription": {
    "markdown": "Cache invalidation added roughly a day a month of operational work. Predictable at the time; nobody logged it as an expected cost.",
    "lang": "en"
  },
  "discoveryType": "Deducible"
}
```

The honest label matters. A run of Deducible outcomes across a ledger says the team's expected-outcome
discipline is thin — useful, and only visible if the labels are truthful.

## Correcting a prior observation

Only when the earlier reading was **wrong when it was made**:

```json
{
  "mdlGuid": "<mdlGuid>",
  "mdrNumber": 7,
  "claimDescription": {
    "markdown": "The earlier 410ms figure came from a dashboard filtered to a single unhealthy node. Fleet-wide P99 was 190ms.",
    "lang": "en"
  },
  "discoveryType": "Expected",
  "tests": { "outcomeId": "eo-read-latency" },
  "testResult": "Satisfied",
  "correctsFact": "<priorObservedOutcomeId>"
}
```

Do **not** use `correctsFact` because the world changed. Latency that was genuinely 410ms in December
and 190ms in March is two honest observations that coexist.

Each prior admission accepts at most one direct corrector, and a second is refused naming what it
would correct:

```
This Observed Outcome already has a direct corrector (at most one correctsFact edge per prior admission).
```

## Reading back

```
get_MDR_learning_delta(mdlGuid, mdrNumber)
```

```json
{
  "mdlGuid": "...",
  "mdrNumber": 7,
  "mdrNumber": 7,
  "expectedOutcomes": [
    {
      "outcomeId": "eo-read-latency",
      "description": { "markdown": "P99 read latency stays under 200ms within 30 days.", "lang": "en" },
      "wakes": [
        { "observedOutcomeId": "...", "discoveryType": "Expected", "testResult": "Violated", "observedAt": "..." }
      ]
    }
  ],
  "emergentOrDeducible": [ ... ]
}
```

An expected outcome with an empty `wakes` array is an **unmeasured commitment** — worth naming to the
user. Ledger residents only.

## Errors

| Message | Cause |
| --- | --- |
| `Memolok Decision Record not found.` | No record holds that number. A staged record holds none, so it can take no outcome and has no learning delta |
| `discoveryType Expected requires tests referencing an expectedOutcome.` | Missing `tests` |
| `discoveryType Expected requires testResult (Satisfied, Violated, or Inconclusive).` | Missing `testResult` |
| `tests and testResult are only valid when discoveryType is Expected.` | Sent on Emergent or Deducible |
| `tests has unsupported field(s): {fields}.` | `tests` naming anything but `outcomeId`; the record is the one `mdrNumber` names |
| `expectedOutcome '{id}' was not found on the source Memolok Decision Record.` | The `eo-…` id is not one of the record's expected outcomes |
| `Unknown discoveryType {x}. Use one of: Deducible, Emergent, Expected.` | Typo or invented value |
| `Unknown testResult {x}. Use one of: Inconclusive, Satisfied, Violated.` | Typo or invented value |
| `A Memolok Decision Record cannot cite its own Observed Outcome in hasContext (decision transaction principle).` | A record trying to cite its own wake as context |

## The learning loop

```
Matter₁ → Analysis₁ → MDR₁ → expected outcome → observed outcome (Violated)
  → Claim₂ ─┐
            ├→ Analysis₂ → MDR₂ at a new t₀
  Matter₂ ──┘
```

Drawn as a single file for legibility only. The second analysis may take up the wake itself — pass
`observedOutcomeId` in `motivatedBy` — fresh matters raised since, or both together, and may produce
any number of records. Taking up the wake directly is what keeps the arrow into Analysis₂ real: a
Matter retyping the observation would draw the same picture and record none of it.

The wake becomes bait for the next fish. The original record stays exactly as it was at its own t₀ —
that is what makes the loop legible in hindsight.

MDR₂ closes as **Accepted**, carrying `supersedes: [MDR₁]` when it replaces the decision, or as
**Rejected** when it declines the commitment — step 6 of the skill says how the second is linked, since
a Rejection carries no edge.
