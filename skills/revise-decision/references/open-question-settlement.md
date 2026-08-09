# Settling an open question

The route depends entirely on **the holder's status** — not on which record was created or admitted
first.

## Normal path — the holder is already admitted

1. The holder reached **Accepted** carrying one or more `openQuestions`.
2. A later staged resolver patches `settlesOpenQuestion`.
3. The resolver reaches **Accepted**.
4. Admission mints `settledIn` on the older holder.

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

`hostMdrNumber` is the **holder's** number — the record that asked the question — not the resolver's.
`openQuestionId` is the server-minted `oq-…` value; read it from the holder with `get_MDR`.

This is the right mechanism because the holder crossed t₀. Its open question is frozen ledger history,
so only a later Accepted record can close it retroactively.

## Inverted path — the holder is still staged

1. The resolver is already **Accepted** — say MDR-3.
2. Another record still sits staged with a related open question.
3. That holder has no number yet, so there is nothing frozen to settle.
4. Update the **holder** directly: drop the resolved question, or fold the resolution into the Verdict,
   citing the resolver in prose.

```json
{
  "mdlGuid": "<mdlGuid>",
  "mdrHandle": 9,
  "patch": {
    "verdict": {
      "description": {
        "markdown": "Proceed with the hull mechanism. Manifold and self-intersection safety is not enforced by the tool, per MDR-3.",
        "lang": "en"
      }
    },
    "openQuestions": []
  }
}
```

This is **not** a `settlesOpenQuestion` case, and forcing it produces an error — a staged record has no
`mdrNumber` to target.

## Both records authored in one sitting

Common when one piece of work produces two decisions — one asking a question, one answering it. The
holder's status still decides the route, but now you control that status, so sequence it deliberately:

1. Draft **both** at `Deliberating`. Neither has a number yet.
2. Seal the **holder** first. Admission mints its `mdrNumber` and freezes its `openQuestions`.
3. Patch `settlesOpenQuestion` onto the still-staged resolver, using the number step 2 produced.
4. Seal the resolver.

Getting this backwards is the trap. Seal the resolver first and there is no number to name, the
inverted path above is the only route left, and a genuine settlement edge has been lost to sequencing
rather than to intent.

The edge itself may be patched at any point while the resolver is staged — targets are resolved at
the **resolver's admission**, not when the patch lands — so step 3 is not urgent. Step 2 preceding it
is what matters.

## Consequences

Settling **anchors** the holder: `retractable` becomes `false`, and it can no longer be uncommitted.
Only a successor carrying `supersedes` can change it after that.

`settledIn` is read-only. It is minted at the resolver's admission and cannot be patched:

```
openQuestions[].settledIn cannot be set via MCP; use settlesOpenQuestion on a closing staged record at admission.
```

## Errors

| Message | Cause |
| --- | --- |
| `Only Accepted Memolok Decision Records may carry settlesOpenQuestion.` | The resolver is admitting as Rejected |
| `settlesOpenQuestion target MDR-{n} is not a ledger resident in this MDL.` | Holder is staged, or in another ledger |
| `settlesOpenQuestion target MDR-{n} has no open question with id {id}.` | Wrong `openQuestionId` |
| `...is already settled in MDR-{m}.` | Already closed by an earlier record |
| `Inter-record graph links may only be authored on staged Memolok Decision Records.` | Patch attempted after the resolver admitted |

## Do not

**Do not settle in advance.**

> *"MDR-3 is admitted now, so I can link it back to settle that open question on your staged record."*

A staged holder still has live, editable questions. There is nothing frozen to close, and the schema
says so — `settlesOpenQuestion` targets a number the record does not have.

**Do not treat open questions as a backlog to burn down.** They are scope boundaries that travel with
the record on purpose. A record with three unsettled questions from a year ago is not neglected; it is
a record that was honest about its edges.

**Do not settle a question just because a later record happens to touch the topic.** Settlement is a
claim that this decision closed that question. If it did not, leave it open.
