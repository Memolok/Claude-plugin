# Matter branch payloads

## Path A — bait becomes a decision

### 1. `register_matter`

The source actor's verbatim words. No sharpening at this step.

```json
{
  "mdlGuid": "<mdlGuid>",
  "description": {
    "markdown": "Users report the dashboard takes 10+ seconds to load.",
    "lang": "en"
  }
}
```

Returns `{ id, mdlGuid, status: "MatterReceived", description }`. Keep the `id` — the next call
needs it. If it is lost, `list_matters(status="MatterReceived")` finds it again.

### 2. `create_analysis`

```json
{
  "mdlGuid": "<mdlGuid>",
  "analyzes": "<matterId>",
  "analysisRationale": {
    "markdown": "The problem is initial page load latency, not API throughput.",
    "lang": "en"
  },
  "producesDecision": true,
  "claimDescription": {
    "markdown": "Dashboard first meaningful paint must be under 2 seconds on 4G.",
    "lang": "en"
  }
}
```

Returns `{ analysis, mdr }`. The record is at **New**, with `promptedBy` set and `mdrNumber: null`.
Read `mdr.mdrHandle` from the response.

The matter moves `MatterReceived` → `MatterAnalyzing` → `MatterAnalyzed`.

**The `claimDescription` here is the sharpened need, agreed with the user** — not a restatement of the
matter, and not the mechanism you expect to choose.

### 3. Open deliberation

```json
{
  "mdlGuid": "<mdlGuid>",
  "mdrHandle": 1,
  "status": "Deliberating"
}
```

Then continue with the body — see `patch-payloads.md`.

## Path B — honest dismissal

Same `register_matter`, then:

```json
{
  "mdlGuid": "<mdlGuid>",
  "analyzes": "<matterId>",
  "analysisRationale": {
    "markdown": "Reported behaviour matches documented beta limitations; no decision warranted.",
    "lang": "en"
  },
  "producesDecision": false
}
```

Returns `{ analysis }` with no `mdr`. The matter ends at `MatterDismissed`.

Omit `claimDescription` entirely — sending it with `producesDecision: false` is incoherent.

The rationale must actually explain **why no decision is warranted**. A dismissed matter with a
real rationale is useful evidence; inputs that go nowhere are data, not noise.

## Path B is not the same as a Rejected record

| | Path B dismissal | Record at **Rejected** |
| --- | --- | --- |
| Fish minted | No | Yes |
| What it says | The matter never sharpened into a decision worth recording | The need was real, and we commit to **not** proceeding |
| Carries a Verdict | No | Yes, explaining why not |
| Ledger number | None | Assigned at t₀ |

If the user weighed options and concluded "no", that is a **Rejected** record, not a dismissal.

## Errors

| Message | Cause |
| --- | --- |
| `Analysis can only start from a Matter in MatterReceived status.` | Already analyzed — one analysis per matter |
| `claimDescription is required when analysis produces a Memolok Decision Record.` | Path A without a claim |
| `Matter not found.` | Wrong id, or a matter from another ledger |
| `You are not a member of this Memolok Decision Ledger.` | Write without membership |

## Do not

- Sharpen, summarize, or translate the matter text at registration
- Call `create_MDR` with a hand-set `promptedBy` — only Path A can set it
- Re-analyze a matter; a second look needs a new matter
- Register a matter the user did not actually raise, to justify a decision they brought fully formed
