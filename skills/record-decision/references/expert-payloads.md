# Expert branch payloads

## Minimal mint

The default. Head Claim only, everything else patched in as deliberation proceeds.

```json
{
  "mdlGuid": "<mdlGuid>",
  "claimDescription": {
    "markdown": "P99 API latency must stay under 200ms at 500 concurrent users.",
    "lang": "en"
  },
  "status": "Deliberating"
}
```

Returns the record with a minted `mdrHandle` and `mdrNumber: null`.

`claimDescription` takes the prose object **directly** — no `description` wrapper. It is the one
parameter shaped that way on this tool.

## Pre-populated mint

When the conversation already produced most of the body, send it at create. **Status is still
`Deliberating`** — having a full body is not commitment.

```json
{
  "mdlGuid": "<mdlGuid>",
  "claimDescription": {
    "markdown": "The chosen language must have a mature, well-supported statistical-analysis library ecosystem.",
    "lang": "en"
  },
  "status": "Deliberating",
  "alternatives": [
    {
      "id": "alt-python",
      "label": "Python",
      "description": {
        "markdown": "pandas / numpy / scipy; the team has three people already fluent.",
        "lang": "en"
      }
    },
    {
      "id": "alt-r",
      "label": "R",
      "description": {
        "markdown": "Stronger native statistics, but nobody on the team writes it day to day.",
        "lang": "en"
      }
    }
  ],
  "deliberationFacts": [
    {
      "onAlternative": "alt-python",
      "description": {
        "markdown": "Existing ETL is already Python, so no second runtime enters the pipeline.",
        "lang": "en"
      }
    },
    {
      "onAlternative": "alt-r",
      "description": {
        "markdown": "Better modelling libraries, but would need a hand-off boundary mid-pipeline.",
        "lang": "en"
      }
    }
  ],
  "chosenAlternative": "alt-python",
  "verdict": {
    "description": {
      "markdown": "Python, because the ecosystem clears the bar and it avoids a second runtime in the pipeline.",
      "lang": "en"
    }
  },
  "expectedOutcomes": [
    {
      "id": "eo-methods-available",
      "description": {
        "markdown": "No statistical method needed in the first two quarters turns out to be unavailable or unmaintained.",
        "lang": "en"
      }
    }
  ],
  "openQuestions": [
    {
      "id": "oq-training-host",
      "description": {
        "markdown": "Whether model training later needs to move off the API host entirely.",
        "lang": "en"
      }
    }
  ]
}
```

Note the Need names no language. The choice lives in `chosenAlternative` and `verdict` — that is
Rule G holding.

## One-shot t₀

Only when the user has stated commitment in their own words. Run the ceremony from
**`commit-decision`** first.

```json
{
  "mdlGuid": "<mdlGuid>",
  "claimDescription": {
    "markdown": "Migrate the monolith to microservices by Q3.",
    "lang": "en"
  },
  "status": "Rejected",
  "verdict": {
    "description": {
      "markdown": "We will not pursue this migration: cost exceeds benefit, the team lacks operational maturity, and the current modular monolith meets its SLOs.",
      "lang": "en"
    }
  }
}
```

Creating at `Accepted` additionally needs alternatives, a valid `chosenAlternative`, and at least one
expected outcome — the same gates a transition would enforce.

## Allowed creation statuses

| Status | When |
| --- | --- |
| `Deliberating` | **Default**, including with a full body |
| `New` | Head Claim only, deliberating later — rare on this branch |
| `Proposed` | Formal governance only |
| `Accepted` | One-shot t₀ — explicit commitment only |
| `Rejected` | One-shot t₀ — explicit commitment only |

`Superseded` is not creatable.

**Send canonical status values.** `create_MDR` does not resolve the `Decided` / `Settled` /
`Committed` aliases that `transition_MDR_status` accepts, and fails with a raw error rather than a
helpful one.

## Errors

| Message | Cause |
| --- | --- |
| `Cannot create a Memolok Decision Record with status Superseded.` | Illegal creation status |
| `At least one alternative is required before proposing…` | Created at `Proposed`/`Accepted` without a body |
| `A chosen alternative must reference one of the record's alternatives.` | `chosenAlternative` does not match an `id`, at the admission gate |
| `The chosen alternative '…' does not name one of the record's alternatives.` | The same mismatch, caught at the write instead |
| `…id is required.` | An item on `alternatives`, `expectedOutcomes` or `openQuestions` carries no `id` |
| `…id '…' must start with '…' — did you mean '…'?` | Missing the list's prefix; the message names the correction |
| `…id '…' is used twice in this list.` | Two items claiming one id |
| `At least one expected outcome is required before accepting…` | Created at `Accepted` with no tail |
| `Field required [type=missing]` naming `verdict.description` | Sent `verdict` as bare `{markdown, lang}` |

## What create_MDR cannot carry

The fish body only. `hasContext`, `settlesOpenQuestion` and `supersedes` have **no parameter** on this
tool — they are authored by a follow-up `update_MDR` while the record is still staged.

Worth knowing before you compose the call rather than after: a record that needs to cite a World Fact
or a prior wake as context, or to name what it settles or supersedes, is a **two-call mint**. Discover
it by trying and you will have already committed to a body that reads as if the context were there.

## Do not

- Look for a matter parameter — there is none, and provenance only comes from an analysis producing the record
- Mint at `Accepted` because the Verdict you drafted reads convincingly
- Offer `Proposed` on informal work
