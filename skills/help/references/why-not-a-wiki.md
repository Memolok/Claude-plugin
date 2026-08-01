# Why not point a model at the wiki

It is the first question any technical executive asks, and it deserves a straight answer: you absolutely
can, and you will get plausible prose.

That is the problem.

## What the model is actually working with

Wikis, tickets, and shared drives are unstructured material. They hold contradictory edits nobody
reconciled, pages whose status is ambiguous — is this current policy, a proposal, or someone's draft from
2019? — debates that trail off without resolution, and narrative that was smoothed for an audience rather
than recorded for accuracy.

Ask a language model what the organization decided, and it will do its honest best to reconstruct a story
from those fragments. Reconstruction under ambiguity is exactly the situation in which a model produces
coherence that was never there — not through malice or malfunction, but because assembling a plausible
account is what it is for.

The result is institutional guesswork at scale. And the genuinely dangerous part is that it works often
enough to feel reliable. A demo goes beautifully. Three answers in a row are right. The fourth is
confidently wrong in a way nobody catches, because there is nothing to check it against.

## What structure changes

A decision ledger is not a better-organized wiki. It holds things that were **committed**, with the
properties that matter recorded rather than inferred: which decision replaced which, who was accountable,
whether something was decided or is still being argued, what was explicitly left unsettled and whether
anything later answered it.

Questions like *what constrains this choice*, *what superseded what*, *who approved this bet*, and *which
topics did we explicitly leave open* get answered from recorded structure. An agent reading it is
navigating what was actually decided instead of inventing a narrative that fits.

The honest comparison is not "wiki bad, ledger good". A wiki is a fine place for documentation, and most
of what is in yours belongs there. The claim is narrower: a wiki cannot tell you what was decided, because
nothing in it was ever committed as a decision.

## Where this is going

The broader idea is that decision knowledge becomes infrastructure — a durable, queryable layer beneath
the design threads, strategy chats, and meeting transcripts where choices actually get made, which agents
can read and write as the conversation happens, so that practitioners never face a blank form.

> **Not built yet.** That direction is the design intent, not a description of the product. Today Memolok
> records decisions through this plugin and stores them dependably. There is no meeting capture, no
> automatic distillation from conversation, and no browsing surface — recording still happens because a
> person, or an agent working with them, decides to record it.
