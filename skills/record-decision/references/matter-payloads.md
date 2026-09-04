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

Returns `{ id, mdlGuid, description, takenUpBy: [] }`. Keep the `id` — the next call needs it. If it
is lost, `list_matters(untaken: true)` finds it again.

### 2. `create_analysis`

```json
{
  "mdlGuid": "<mdlGuid>",
  "motivatedBy": ["<matterId>", "<anotherMatterId>"],
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

Returns `{ analysis, mdr }`. The record is at **New** with `mdrNumber: null`. Read `mdr.mdrHandle`
from the response.

`motivatedBy` is a **list**, and its entries need not all be matters: an `mt_` matter id, a `wf_`
World Fact and an `oo_` Observed Outcome are equally valid, in any mix — the prefix on each says
which. Pass every input this reasoning took up — three reports of one fault go in one call, not
three, and so do a fresh report and the recorded outcome that prompted someone to look. The analysis
concludes here, and `analysis.references` comes back with each input dated and `late: false`.

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
  "motivatedBy": ["<matterId>"],
  "analysisRationale": {
    "markdown": "Reported behaviour matches documented beta limitations; no decision warranted.",
    "lang": "en"
  },
  "producesDecision": false
}
```

Returns `{ analysis }` with no `mdr`. Nothing is written on the matter itself — the concluded
analysis producing no record *is* the account.

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
| `An analysis must take up at least one input; motivatedBy is empty.` | Empty list |
| `That input is already referenced by this analysis.` | Attaching an input the analysis already references |
| `claimDescription is required when analysis produces a Memolok Decision Record.` | Path A without a claim |
| `motivatedBy[i] was not found in this Memolok Decision Ledger.` | Wrong id, or an id that is not a matter, world fact or observed outcome. The index is the position in your list |
| `The motivatedBy reference belongs to a different Memolok Decision Ledger.` | A live id, from another ledger |
| `You are not a member of this Memolok Decision Ledger.` | Write without membership |

## Do not

- Sharpen, summarize, or translate the matter text at registration
- Pass one `motivatedBy` where several apply — that records reasoning that did not happen
- Split one act of reasoning into an analysis per input to route around the list
- Decide which produced record "belongs to" which matter; nothing asks, and usually nothing is true
- Register a matter the user did not actually raise, to justify a decision they brought fully formed
