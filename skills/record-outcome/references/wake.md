# Wake payloads

## Expected — assessing a tail commitment

Requires both `tests` and `testResult`.

```json
{
  "mdlGuid": "<mdlGuid>",
  "mdrHandle": 1,
  "claimDescription": {
    "markdown": "P99 read latency measured 410ms during the December peak, against the 200ms target.",
    "lang": "en"
  },
  "discoveryType": "Expected",
  "tests": { "outcomeId": "eo-3f9a2c11" },
  "testResult": "Violated"
}
```

`tests` also accepts a bare id string. Get the `eo-…` value from `get_MDR` on the source record, or
from the response that created the outcomes.

## Emergent — nobody saw it coming

No `tests`, no `testResult`.

```json
{
  "mdlGuid": "<mdlGuid>",
  "mdrHandle": 1,
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
  "mdrHandle": 1,
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
  "mdrHandle": 1,
  "claimDescription": {
    "markdown": "The earlier 410ms figure came from a dashboard filtered to a single unhealthy node. Fleet-wide P99 was 190ms.",
    "lang": "en"
  },
  "discoveryType": "Expected",
  "tests": { "outcomeId": "eo-3f9a2c11" },
  "testResult": "Satisfied",
  "correctsFact": "<priorObservedOutcomeId>"
}
```

Do **not** use `correctsFact` because the world changed. Latency that was genuinely 410ms in December
and 190ms in March is two honest observations that coexist.

Each prior admission accepts at most one direct corrector:

```
This World Fact already has a direct corrector (at most one correctsFact edge per prior admission).
```

## Reading back

```
get_MDR_learning_delta(mdlGuid, mdrHandle)
```

```json
{
  "mdlGuid": "...",
  "mdrHandle": 1,
  "mdrNumber": 7,
  "expectedOutcomes": [
    {
      "outcomeId": "eo-3f9a2c11",
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
| `Observed Outcomes can only realizeFrom a ledger-resident Memolok Decision Record (Accepted, Rejected, or Superseded).` | Source is staged |
| `discoveryType Expected requires tests referencing an expectedOutcome.` | Missing `tests` |
| `discoveryType Expected requires testResult (Satisfied, Violated, or Inconclusive).` | Missing `testResult` |
| `tests and testResult are only valid when discoveryType is Expected.` | Sent on Emergent or Deducible |
| `Unknown discoveryType {x}. Use one of: Deducible, Emergent, Expected.` | Typo or invented value |
| `Unknown testResult {x}. Use one of: Inconclusive, Satisfied, Violated.` | Typo or invented value |
| `Learning delta is available for ledger-resident Memolok Decision Records only.` | Delta requested on a staged record |
| `A Memolok Decision Record cannot cite its own Observed Outcome in hasContext (decision transaction principle).` | A record trying to cite its own wake as context |

## The self-correcting loop

```
Matter₁ → Analysis₁ → MDR₁ → expected outcome → observed outcome (Violated)
  → Matter₂ → Analysis₂ → MDR₂ at a new t₀
```

The wake becomes bait for the next fish. The original record stays exactly as it was at its own t₀ —
that is what makes the loop legible in hindsight.

MDR₂ closes in one of two shapes, and they are not interchangeable:

| MDR₂ admits as | Link back | When |
| --- | --- | --- |
| **Accepted** | May carry `supersedes: [MDR₁]`, which flips MDR₁ to `Superseded` | A replacement decision is being made |
| **Rejected** | **None.** A Rejection carries no `supersedes` and no `settlesOpenQuestion` | The commitment itself is being declined rather than replaced |

The second is common when MDR₁'s decision shipped and holds, and only what it *promised* was wrong.
Nothing structural connects the pair, so the successor must cite the wake in `hasContext` and say in
its Verdict what it declines — otherwise a reader arriving at MDR₁ finds a violated commitment and no
sign anyone answered it.

## What never happens

- Editing an expected outcome so it matches the result
- Editing a prior observation because a later one disagreed
- Automatic supersession on a `Violated` result
- Backdating an observation — there is no `observedAt` parameter
- Deleting an outcome
