# Prose format and RACI

## The prose atom

Every Memolok prose field is Markdown, not plain text. The JSON key is `markdown`:

```json
{ "markdown": "Required non-empty **Markdown** string.", "lang": "en" }
```

| Rule | Detail |
| --- | --- |
| `markdown` | Required; whitespace-only is rejected |
| `lang` | Optional; defaults to `"en"` |
| Shape | No other keys accepted |

Sending `{ "text": ... }` instead of `{ "markdown": ... }` is rejected outright:

```
{field}.text is no longer supported; use {field}.markdown
```

### Markdown and LaTeX

Memolok commits to LaTeX maths inside Markdown across all prose fields:

| Form | Syntax | Example |
| --- | --- | --- |
| Inline | `$...$` | `$P_{99} < 200\,\text{ms}$` |
| Display | `$$...$$` | `$$\sum_i w_i = 1$$` |

Use ordinary Markdown elsewhere — headings, lists, emphasis, links, fenced code. Falsifiable claims
with numeric thresholds should be written as Markdown or LaTeX, not ad-hoc plain-text conventions.

## Wrapped vs bare — the one real trap

Some parameters take the prose atom **directly**; others take it wrapped in a `description` key.

| Bare `{ markdown, lang }` | Wrapped `{ description: { markdown, lang } }` |
| --- | --- |
| `create_MDR.claimDescription` | `verdict` |
| `admit_world_fact.claimDescription` | `openQuestions[]` |
| `record_observed_outcome.claimDescription` | `expectedOutcomes[]` |
| `register_matter.description` | `deliberationFacts[]` |
| `create_analysis.analysisRationale` | `alternatives[]` |
| `create_analysis.claimDescription` | |

Getting it backwards fails with `Field required [type=missing]` naming the wrapped path, or:

```
verdict must be {description: {markdown, lang}}, not a flat {markdown, lang} object.
```

`update_MDR`'s `hasNeed` accepts **either** form.

`expectedOutcomes[]` and `deliberationFacts[]` also accept a `manifests` wrapper instead of
`description` — omit it unless you are attaching an existing Claim by id. The server auto-mints the
Claim and sets `manifests` when you supply `description`, and hydrates it back to `description` on
read, so the read shape differs from what you wrote.

## Ids you choose, and later have to cite

**You** name every id on `alternatives`, `expectedOutcomes` and `openQuestions` — the server mints
none of them. Each must carry its list's prefix and be unique within that list:

| Embed | Prefix | Cited later by |
| --- | --- | --- |
| `alternatives[]` | `alt-…` | `chosenAlternative`, `deliberationFacts[].onAlternative` |
| `expectedOutcomes[]` | `eo-…` | `record_observed_outcome`'s `tests.outcomeId` |
| `openQuestions[]` | `oq-…` | `settlesOpenQuestion.openQuestionId` |

Two of those citations arrive in a *later* session, and one arrives in the same call — so name them
for what they are. `alt-cache-layer` tells a reader what won; a generated id tells them nothing.

Full contract and the refusals: `../record-decision/references/patch-payloads.md`.

## Alternatives

`label` is a short handle; `description` is the option's **substance** — the approach, the sketch,
what the option actually is. Arguments *about* an option belong in `deliberationFacts`, paired with
`onAlternative` matching the alternative's `id`.

Alternatives are passed through to schema validation unchanged, accepting
`{ id?, label?, description, satisfies? }` and nothing else.

## Agent IRIs

Non-empty strings identifying people or systems.

| Field | Cardinality | Default |
| --- | --- | --- |
| `authoredBy` | 0–1 | `https://www.memolok.ai/users/{userId}` |
| `decidedBy` | 0–1 | none |
| `consulted` | 0–n | `[]` |
| `informed` | 0–n | `[]` |
| `performedBy` (Analysis) | 1 | caller |

Get `userId` from `whoami` when attributing a human.

## RACI is narrative, not permission

RACI fields are stored for governance narrative. They do **not** gate writes. Access is decided by
MDL membership alone:

| Role | Reads | Writes | Uncommit |
| --- | --- | --- | --- |
| `visitor` | yes | no | no |
| `member` | yes | yes | no |
| `admin`, `owner` | yes | yes | yes |

If no RACI roles are assigned, Memolok assumes the artifact's author is responsible and accountable.

## Valence fields are not yours to set

`deliberationValence` (`Supports`, `Against`, `Neutral`) and `outcomeValence` (`ExpectedGain`,
`ExpectedCost`, `ExpectedRisk`, `ExpectedDependency`) exist in the model but are tooling-inferred and
rejected if sent. Express the same information in the prose instead.
