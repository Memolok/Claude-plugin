# When a write fails

Two failure shapes reach a caller, and they call for opposite responses.

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
request was understood and refused; an internal error means the request was lost. Neither loses the
conversation, and in both cases the shaping you did together is still in front of you — say that,
because the failure otherwise reads as though the work is gone.
