# When a write fails

A failed write reaches you in one of a few shapes, and they call for opposite responses. A trailing
`Reference: req_…` does not tell you which: it is there so an administrator can find the call, and it
rides on replies you must repeat as well as on the one you must not.

## Internal error with a reference

```
Memolok hit an internal error and could not complete this request. Reference: req_…
```

The write did **not** happen. Nothing was captured, and the record you were shaping does not exist on
the ledger. Say so plainly rather than describing what you were about to do as though it landed.

The message carries no detail on purpose — the cause is in the server's logs and the reference is the
only handle on it. So there is nothing to reason from, and **any explanation you offer for why it
failed is invented.** Give the user the reference verbatim and let them decide what to do next; an
administrator can resolve it to the exact failure.

**Do not retry the identical call.** These faults are overwhelmingly deterministic, so a second attempt
fails the same way and a third looks like flailing.

**Do not write the content to a local file.** That is not the ledger, and a decision written there is a
decision that was not recorded. Holding it in the conversation and offering to retry once the user has
an answer is the honest fallback.

## A change only partly made

```
This change to the Memolok Decision Record was only partly made: this API lost its store part-way through. This is a fault here. Repeat the same call to complete it. Reference: req_…
```

Only `transition_MDR_status` and `uncommit_MDR` answer this, with *lost its store* or *failed* in the
middle. Part of the change landed, and the record stays marked until the rest does. **Repeat the
identical call** — same record, same status or reason. It completes exactly what was begun; it is not a
second change. The seal or the uncommit has not happened until the repeat succeeds, so do not report
it as done before then.

Rarely, the first call did finish and only its answer was lost. The repeat is then refused as a fresh
call would be — `Cannot transition a Memolok Decision Record from Accepted to Accepted.`, or an
uncommit refused because the record is already staged. That refusal means the change landed: read the
record to confirm, and report it done.

Until it is repeated, any other change to that record is refused with a sentence naming the call:

```
An earlier transition of this Memolok Decision Record to Accepted was only partly made. Repeat that transition to complete it before changing the record in any other way.
```

The same holds for an earlier uncommit, and for sealing a record whose links name one left unfinished.
Repeat the named call first, then the one you were making.

## Data service unavailable

```
Memolok's data service is temporarily unavailable. Try again shortly. Reference: req_…
```

An outage, not a fault in your call, and nothing was written. Repeat the same call after a short pause.
If it keeps failing, tell the user, give them the reference, and hold the work in the conversation.

## Invalid arguments

```
Invalid arguments for this tool: …
```

The opposite case, and **retryable**. It names the fields that were wrong — correct them and call
again. Values are deliberately omitted from the message; only field names and types come back, so do
not expect the payload echoed for comparison.

## Corrective validation errors

Domain messages pass through verbatim and are the ones you can act on directly — they name the field
and what it needed:

```
alternatives[0].satisfies must be a non-empty string.
openQuestions[].settledIn cannot be set via MCP; use settlesOpenQuestion on a closing staged record at admission.
```

Treat these as instructions rather than obstacles. The message is the fix.

## Telling the user

The distinction that matters to them is whether their work is safe. A validation error means the
request was understood and refused; an internal error means the request was lost; a change only
partly made is finished by repeating it, which you do before telling them anything. None loses the
conversation, and in both cases the shaping you did together is still in front of you — say that,
because the failure otherwise reads as though the work is gone.
